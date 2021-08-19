<template>
  <v-app>
    <Field v-bind="field"></Field>
  </v-app>
</template>

<script>
import Field from "./components/Field.vue";

export default {
  name: "App",

  components: {
    Field: Field,
  },

  data: function () {
    return {
      field: {
        cells: new Array(24).fill({}),
      },
    };
  },
};
</script>
