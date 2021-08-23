<template>
  <v-app>
    <v-navigation-drawer app>
      <!-- -->
    </v-navigation-drawer>

    <v-app-bar app>
      <!-- -->
    </v-app-bar>

    <v-main>
      <v-container class="d-flex">
      <field class="flex-grow-1" v-bind="field"></field>
      </v-container>
    </v-main>

    <v-footer app class="flex-grow-0">
      <v-card
        flat
        tile
        class="indigo lighten-1 white--text text-center flex-grow-1"
        >
        <v-card-text class="white--text pt-0">
          Contact info here.
        </v-card-text>

        <v-divider></v-divider>

        <v-card-text class="white--text">
          © {{ currentYear }} — <strong>Evgeny Cherkashin</strong>
        </v-card-text>
      </v-card>
    </v-footer>
  </v-app>
</template>

<script>
import Field from "./components/Field.vue";

export default {
  name: "App",

  components: {
    field: Field,
  },

  data: function () {
    return {
      field: {
        cells: new Array(24).fill({}),
      },
      currentYear: new Date().getFullYear(),
    };
  },
};
</script>
