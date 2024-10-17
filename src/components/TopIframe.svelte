<script lang="ts">
  export let url: string;
</script>

<iframe 
  src={url}
  title="Top content"
  style="
    width: 100vw;
    height: 80vh;
    overflow: hidden;
    outline: none;
    border: none;
    margin: 0;
    padding: 0;
  "
/>

<style>
  iframe {
    border: none;
  }
</style>