<template>
  <div class="index">
    <i-head></i-head>
    <i-main></i-main>
  </div>
</template>

<script>
import IHead from "@/components/Head.vue";
import IMain from "./Main.vue";
export default {
  name: "Index",
  components: {
    IHead,
    IMain
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
