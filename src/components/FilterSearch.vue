<script>
import {
  CheckboxGroup,
  Form,
  Checkbox,
  Button,
  Input,
  Icon,
  Dropdown,
  DropdownMenu,
  DropdownItem
} from "iview";
import "animate.css";
import _ from "lodash";
export default {
  name: "FilterSearch",
  data() {
    return {
      hideShowClicked: false,
      rowNum: this.rowNumProp,
      show: false,
      fixed: false
    };
  },
  props: {
    //一行显示的个数
    rowNumProp: {
      type: Number,
      default() {
        return 3;
      }
    },
    columns: {
      type: Array,
      required: true
    },
    value: {
      type: Object,
      required: true
    },
    hideBtnHide: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    //分割数组
    splitArray(datas) {
      let datasArray = [];
      let count = 0;
      let rowIndex = 1;
      let arr = [];
      _.map(datas, (val, index) => {
        let num = val.num ? val.num : 1;
        count += num;
        if (count >= this.rowNum * rowIndex || index + 1 == datas.length) {
          arr.push(val);
          datasArray.push(arr);
          arr = [];
          rowIndex += 1;
        } else {
          arr.push(val);
        }
      });
      return datasArray;
    },
    hideShow() {
      this.hideShowClicked = true;
      this.rowNum = !this.show ? 5 : 3;
      this.show = !this.show;
    },
    getValue(valuekey, valKey) {
      let inputVal;
      if (valuekey) {
        const valueHasKey = _.has(this.value, valuekey);
        if (!valueHasKey) {
          this.$set(this.value, valuekey, {});
        } else {
          inputVal = this.value[valuekey][valKey];
        }
      } else {
        inputVal = this.value[valKey];
      }
      return inputVal;
    },
    setValue(valuekey, valKey, valValue) {
      if (valuekey) {
        this.$set(this.value[valuekey], valKey, valValue);
      } else {
        this.$set(this.value, valKey, valValue);
      }
    },
    setEmtpy(value) {
      _.map(value, (val, key) => {
        if (_.isString(val)) {
          value[key] = "";
        }
        if (_.isObject(val)) {
          this.setEmtpy(val);
        }
      });
    },
    doAction(action) {
      if (_.isString(action)) {
        switch (action) {
          case "reset":
            this.setEmtpy(this.value);
            this.$emit("input", this.value);
            break;
          //Todo other type button
          default:
            break;
        }
      }
      if (_.isFunction(action)) {
        action();
      }
    }
  },
  computed: {
    inputs() {
      const inputs = _.find(this.columns, val => val.type === "inputs");
      if (_.isEmpty(inputs)) {
        console.warn("inputs为空");
        return {};
      } else {
        if (!_.has(inputs, "datas")) {
          console.error(`inputs需要datas,结构如下所示: 
          {
            datas: [
              {
                label: "用户名",
                render: () => {
                  return (
                    <i-select
                      value={this.searchData.address}
                      onInput={value => {
                        this.searchData.address = value;
                      }}
                      clearable
                    >
                      {_.map(this.cityList, val => {
                        return (
                          <i-option value={val.value} key={val.value}>
                            {val.name}
                          </i-option>
                        );
                      })}
                    </i-select>
                  );
                }
              },
              {
                label: "密码",
                key: "name"
              }
            ],
            key: "test", //传入节点值 不传扩展到根对象
            type: "inputs"
          },`);
        }
        return inputs;
      }
    },
    checkboxs() {
      const checkboxs = _.find(this.columns, val => val.type === "checkboxs");
      if (_.isEmpty(checkboxs)) {
        console.warn("checkboxs为空");
        return {};
      } else {
        if (!_.has(checkboxs, "datas")) {
          console.error(`checkboxs需要datas,结构如下: 
          datas: [
              { label: "aa", key: "gg", value: "aa" },
              { label: "ss", key: "aa", value: "cc" },
              { label: "dd", key: "bb", value: "cc" }
          ]`);
        }
        if (!_.has(checkboxs, "key")) {
          console.error("checkboxs的key必传");
        }
        return checkboxs;
      }
    },
    buttons() {
      const buttons = _.find(this.columns, val => val.type === "buttons");
      if (_.isEmpty(buttons)) {
        console.warn("buttons为空");
        return {};
      } else {
        if (!_.has(buttons, "datas")) {
          console.log(`buttons需要datas,结构如下: 
          datas: [
              {
                label: "查询",
                action: () => {
                  this.search();
                }
              },
              {
                label: "取消"
              }
          ]`);
        }
        return buttons;
      }
    }
  },

  render(h) {
    let allRow = [];
    let rowNum = this.rowNum;
    let inputDatas = _.get(this.inputs, "datas");
    let rowArray = this.splitArray(inputDatas);
    let datas = _.get(this.buttons, "datas");
    let height =
      52 +
      (this.show ? Math.floor(inputDatas.length / rowNum) * 42 : 0) +
      (this.checkboxs ? 35 : 0) +
      11;
    let otherButtons = _.filter(
      datas,
      item => item.label != "查询" && item.label != "重置"
    );
    _.map(rowArray, (row, index) => {
      let currentNum = 0;
      let arr = [];
      _.map(row, val => {
        let num = val.num ? val.num : 1;
        currentNum = currentNum + num;
        arr.push(
          <div
            class="itemOption"
            style={[
              {
                width: `${(100 / rowNum) * num}%`
              },
              val.style
            ]}
          >
            {val.render ? (
              val.render(h, val)
            ) : (
              <Input
                clearable
                placeholder={val.label}
                value={this.getValue(this.inputs.key, val.key)}
                onInput={value => {
                  this.setValue(this.inputs.key, val.key, value);
                }}
              />
            )}
          </div>
        );
      });
      allRow.push(
        this.show || index == 0 ? <div class="oneRow">{arr}</div> : ""
      );
    });
    return (
      <div class="contain">
        <div class={["filterSearch", this.fixed ? "fixed" : null]}>
          <Form>
            <div class="top">
              <div class={["InputList", this.show ? "w70" : "w50"]}>
                {allRow}
              </div>
              <div class="rightList animated">
                <div style={{ width: this.show ? "100%" : "50%" }}>
                  {(rowArray.length <= 1 && !this.hideShowClicked) ||
                  this.hideBtnHide ? null : (
                    <Button
                      type="primary"
                      class="hideShow"
                      size="large"
                      onClick={this.hideShow}
                    >
                      {this.show ? "隐藏" : "过滤"}
                      <Icon
                        style={{ marginLeft: "3px" }}
                        type={!this.show ? "ios-arrow-down" : "ios-arrow-up"}
                      />
                    </Button>
                  )}
                  {_.map(
                    _.filter(
                      datas,
                      item => item.label == "查询" || item.label == "重置"
                    ),
                    val => {
                      return (
                        <Button
                          type="primary"
                          onClick={() => this.doAction(val.action)}
                        >
                          {val.label}
                        </Button>
                      );
                    }
                  )}
                </div>

                <div
                  class={[
                    "right animated",
                    this.hideShowClicked
                      ? this.show
                        ? "fadeOutRight"
                        : "fadeInRight"
                      : null
                  ]}
                >
                  {_.map(otherButtons, (val, index) => {
                    if (index < 2) {
                      return (
                        <Button
                          type="primary"
                          onClick={() => this.doAction(val.action)}
                        >
                          {val.label}
                        </Button>
                      );
                    }
                  })}
                  {otherButtons.length > 2 ? (
                    <Dropdown trigger="click">
                      <Button type="primary" size="small" style="width:78px">
                        更多操作
                        <Icon
                          style={{ marginLeft: "3px" }}
                          type={!this.show ? "ios-arrow-down" : "ios-arrow-up"}
                        />
                      </Button>
                      <DropdownMenu slot="list">
                        {_.map(otherButtons, (val, index) => {
                          if (index >= 2) {
                            return (
                              <DropdownItem>
                                <div onClick={() => this.doAction(val.action)}>
                                  {val.label}
                                </div>
                              </DropdownItem>
                            );
                          }
                        })}
                      </DropdownMenu>
                    </Dropdown>
                  ) : null}
                </div>
              </div>
            </div>
            {!_.isEmpty(this.checkboxs) ? (
              <div class="bottom">
                <div class="first">筛选条件:</div>
                <CheckboxGroup
                  value={
                    this.value[this.checkboxs.key]
                      ? _.compact(this.value[this.checkboxs.key].split(","))
                      : []
                  }
                  onInput={val => {
                    this.$set(this.value, this.checkboxs.key, val.toString());
                  }}
                >
                  {_.map(_.get(this.checkboxs, "datas"), val => {
                    return (
                      <Checkbox label={val.value}>
                        <span>{val.label}</span>
                      </Checkbox>
                    );
                  })}
                </CheckboxGroup>
              </div>
            ) : null}
            {this.show ? (
              <div class="fixedTextContain">
                <span class="fixedText">
                  <Checkbox
                    value={this.fixed}
                    onInput={value => {
                      this.fixed = value;
                    }}
                  >
                    固定筛选框
                  </Checkbox>
                </span>
              </div>
            ) : null}
          </Form>
        </div>
        <div style={{ marginTop: this.fixed ? `${height}px` : 0 }} />
      </div>
    );
  }
};
</script>
<style lang="less" scoped>
.contain {
  width: 100%;
}
.filterSearch {
  flex: 1;
  background: #f5f5f5;
  padding: 10px;
  position: relative;
  top: -10px;
  left: -10px;
  width: calc(100% + 20px);
  .w70 {
    width: 70% !important;
  }
  .w50 {
    width: 50% !important;
  }
  .w30 {
    width: 30% !important;
  }
  &.fixed {
    position: fixed;
    top: 0px;
    left: 0px;
    width: 100%;
    z-index: 999;
  }
  .fixedTextContain {
    width: 100%;
    display: block;
    text-align: right;
    .fixedText {
      background: #f2f2f2;
      color: #666;
      padding: 5px 10px;
    }
  }
  .top {
    display: flex;
    .InputList {
      width: 50%;
      display: flex;
      flex-wrap: wrap;
      .itemOption {
        height: 42px;
        overflow: hidden;
        text-align: left;
        padding-right: 10px;
        input {
          width: 100%;
        }
      }
    }
    .rightList {
      width: 50%;
      height: 42px;
      display: flex;
      justify-content: space-between;
      .hideShow {
        transform: scale(0.8);
        margin-left: 0;
        background: #f5f5f5;
        padding: 0;
        border: 1px solid #d9d9d9;
        color: #666;
      }
      .right {
        display: flex;
        flex-wrap: wrap;
        button {
          width: auto;
          min-width: 70px;
        }
      }
      button {
        margin: 0px 10px 10px;
        width: 70px;
        height: 33px;
      }
    }
  }
  .oneRow {
    width: 100%;
    display: flex;
  }
  .bottom {
    display: flex;
    .first {
      margin-right: 10px;
    }
  }
}
</style>
