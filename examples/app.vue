<template>
  <div>
    <div class="container">
      <nav>
        <router-link to="/AutoTable">AutoTable</router-link>
        <router-link to="/EditTable">EditTable</router-link>
        <router-link to="/CurdLine">CurdLine</router-link>
        <router-link to="/FilterSearch">FilterSearch</router-link>
        <router-link to="/DatePickers">DatePickers</router-link>
        <router-link to="/TFSwitch">TFSwitch</router-link>
      </nav>
      <router-view></router-view>
    </div>

  </div>
</template>

<script>
export default {};
</script>

<style scoped>
.container {
  padding: 20px;
}
.container a {
  padding: 0 10px;
  border-right: 1px solid #ccc;
}
</style>
