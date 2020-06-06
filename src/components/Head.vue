<template>
  <div class="head">
    <div>智能分析系统</div>
    <div><span>Admin</span>|<span>退出</span></div>
  </div>
</template>

<script>
export default {
  name: "Head"
};
</script>
<style scoped lang="scss">
.head {
  display: flex;
  justify-content: space-between;
  background: #0255a4;
  height: 60px;
  align-items: center;
  padding: 0 20px;
  color: #fff;
  span {
    padding: 0 5px;
  }
}
</style>
