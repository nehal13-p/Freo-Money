<script lang="ts">
  export let url: string;
</script>

<iframe 
  src={url}
  title="Bottom content"
  style="
    width: 100vw;
    height: 900vh;
    overflow: hidden;
    outline: none;
    border: none;
    margin: 0;
    padding: 0;
    margin-top: 150px;
  "
  scrolling="no"
  class="iframe-2"
/>

<style>
  iframe {
    border: none;
  }
</style>