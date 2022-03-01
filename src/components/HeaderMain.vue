<template>
  <header>
    <div class="header-inner">
      <h1 class="text-center">{{ title }}</h1>
    </div>
  </header>
</template>

<script>
export default {
  name: "HeaderMain",
  props: {
    title: String,
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
header {
  padding: 20px 0 0;
}
h1 {
  color: #111111;
  font-weight: 600;
}
</style>
