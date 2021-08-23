<template lang="html">
  <v-img 
    class="chip"
    :src="src"
  ></v-img>
</template>
<script>
export default {
  props: {
    side: {
      type: String,
      required: true,
      validator: function (value) {
        return ["white", "black"].includes(value);
      },
    },
    type: {
      type: String,
      default: function () {
        return "common";
      },
      validator: function (value) {
        return ["common"].includes(value);
      },
    },
  },
  computed: {
    src: function () {
      return require(`@/assets/chips/${this.type}/${this.side}.svg`);
    },
  },
};
</script>
<style lang="css">
.chip {
  min-width: 20px;
  min-height: 20px;
}
</style>