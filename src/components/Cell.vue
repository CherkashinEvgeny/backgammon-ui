<template lang="html">
  <drag-and-drop :values="test">
    <chip side="white"></chip>
  </drag-and-drop>
</template>
<script>
import DragAndDrop from "./DragAndDrop.vue";
import Chip from "./Chip.vue";

export default {
  components: {
    "drag-and-drop": DragAndDrop,
    "chip": Chip,
  },
  props: {
    values: {
      type: Array,
      default: function () {
        return [];
      },
    },
  },
  data: function () {
    return {
      test: [1, 2],
    };
  },
};
</script>
<style lang="css">
</style>