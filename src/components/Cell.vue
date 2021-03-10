<template>
  <div class="cell light-border margin-0" v-bind:class="{shaded: shaded, head: head, fruit: fruit}"></div>
</template>

<script>
export default {
  name: "Cell",
  props: {
    cellnum: null,
    head:null,
    shaded: null,
    fruit: null,
  },
};
</script>