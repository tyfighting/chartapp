<template>
  <div class="menu">
    <el-menu
      default-active="1"
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
      background-color="#0255a4"
      text-color="#fff"
      active-text-color="#ffd04b"
    >
      <el-menu-item index="1">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>首页</span>
        </template>
      </el-menu-item>
      <el-menu-item index="2">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>信息检索</span>
        </template>
      </el-menu-item>
      <el-submenu index="3">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>标注管理</span>
        </template>
        <el-menu-item-group>
          <el-menu-item index="3-1">标注管理1</el-menu-item>
          <el-menu-item index="3-2">标注管理2</el-menu-item>
        </el-menu-item-group>
      </el-submenu>
      <el-submenu index="4">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>知识管理</span>
        </template>
        <el-menu-item-group>
          <el-menu-item index="4-1">知识管理1</el-menu-item>
          <el-menu-item index="4-2">知识管理2</el-menu-item>
        </el-menu-item-group>
      </el-submenu>
      <el-submenu index="5">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>模型中心</span>
        </template>
        <el-menu-item-group>
          <el-menu-item index="5-1">模型中心1</el-menu-item>
          <el-menu-item index="5-2">模型中心2</el-menu-item>
        </el-menu-item-group>
      </el-submenu>
    </el-menu>
  </div>
</template>

<script>
export default {
  methods: {
    handleOpen(key, keyPath) {
      console.log(key, keyPath);
    },
    handleClose(key, keyPath) {
      console.log(key, keyPath);
    }
  }
};
</script>

<style lang="scss" scoped>
.menu {
  width: 200px;
  .el-menu {
    width: 100%;
    height: 100%;
    border-right: 0;
  }
}
</style>

