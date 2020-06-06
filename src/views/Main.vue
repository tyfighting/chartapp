<template>
  <div class="main">
    <i-menu></i-menu>
    <div class="content">
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
import IMenu from "@/components/Menu.vue";
export default {
  components: {
    IMenu
  },
};
</script>

<style lang="scss" scoped>
.main {
  display: flex;
  .content {
    flex: 1;
    background: #0c3291;
  }
}
</style>
