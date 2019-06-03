<script>
  import Grid from "./Grid.svelte";
  export let name;
</script>

<style lang="scss">
  @import "variables";
  h1 {
    color: $brand;
    text-align: center;
  }

  div {
    width: 80%;
    margin: 0 auto;
    max-width: 1400px;
  }
</style>

<div>
  <h1>New Malden Korean Restaurant Ranking</h1>
  <Grid />

</div>
