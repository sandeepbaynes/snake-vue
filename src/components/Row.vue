<template>
  <div class="">
    <Cell
      v-for="cell in cells"
      :cellnum="cell.colnum"
      v-bind:key="cell.colnum"
      :shaded="cell.shaded"
      :fruit="cell.fruit"
      :head="cell.head"
    ></Cell>
  </div>
</template>

<script>
import Cell from "@/components/Cell.vue";
export default {
  name: "Row",
  props: {
    rownum: null,
    colcount: null,
  },
  components: {
    Cell,
  },
  data: () => {
    return {
      cells: null,
    };
  },
  methods: {
  },
  mounted() {
    if (this.$parent.blocks) {
      this.cells = this.$parent.blocks[this.rownum].cells;
    }
  },
};
</script>