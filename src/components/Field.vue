<template lang="html">
  <v-container>
    <v-row>
      <v-col>
        <v-row>
          <v-col v-for="cell in leftTopCells" :key="cell.index">
            <cell v-bind="cell">
            </cell>
          </v-col>
        </v-row>
        <v-spacer></v-spacer>
        <v-row>
          <v-col v-for="cell in leftBottomCells" :key="cell.index">
            <cell v-bind="cell">
            </cell>
          </v-col>
        </v-row>
      </v-col>
      <v-divider vertical></v-divider>
      <v-col>
        <v-row>
          <v-col v-for="cell in rightTopCells" :key="cell.index">
            <cell v-bind="cell">
            </cell>
          </v-col>
        </v-row>
        <v-spacer></v-spacer>
        <v-row>
          <v-col v-for="cell in rightBottomCells" :key="cell.index">
            <cell v-bind="cell">
            </cell>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import Cell from "./Cell.vue";

export default {
  components: {
    cell: Cell,
  },
  props: {
    cells: {
      type: Array,
      default: function () {
        return new Array(24).map(() => ({}));
      },
      validator: function (value) {
        return value?.length && !(value?.length % 4);
      },
    },
  },
  data: function () {
    return {
      cellsWithIndexes: this.cells?.map((value, index) => ({
        index,
        value,
      })),
    };
  },
  computed: {
    cellsCount: function () {
      return this.cells?.length;
    },
    quarterCellsCount: function () {
      return this.cellsCount / 4;
    },
    leftTopCells: function () {
      const startIndex = this.quarterCellsCount;
      const stopIndex = startIndex + this.quarterCellsCount;
      return this.cellsWithIndexes.slice(startIndex, stopIndex);
    },
    rightTopCells: function () {
      const startIndex = 0;
      const stopIndex = startIndex + this.quarterCellsCount;
      return this.cellsWithIndexes.slice(startIndex, stopIndex);
    },
    leftBottomCells: function () {
      const startIndex = 2 * this.quarterCellsCount;
      const stopIndex = startIndex + this.quarterCellsCount;
      return this.cellsWithIndexes.slice(startIndex, stopIndex).reverse();
    },
    rightBottomCells: function () {
      const startIndex = 3 * this.quarterCellsCount;
      const stopIndex = this.cellsCount;
      return this.cellsWithIndexes.slice(startIndex, stopIndex).reverse();
    },
  },
};
</script>
<style lang="css">
</style>