<script lang="ts">
  import { onMount } from 'svelte';

  let visible = true;

  onMount(() => {
    setTimeout(() => {
      visible = false;
    }, 600);
  });
</script>

{#if visible}
  <div class="shimmer">
    <img src="https://framerusercontent.com/images/j3yi5cUPMP5554RxkkvfkduPiAM.svg" alt="Loading" />
  </div>
{/if}

<style>
  .shimmer {
    position: fixed;
    top: 0;
    bottom: 0;
    right: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 100000;
    background-color: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>