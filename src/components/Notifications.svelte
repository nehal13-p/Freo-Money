<script lang="ts">
  export let notifications: string[];
</script>

<div class="notification-container">
  {#each notifications as notification, index}
    <div class="notification">
      <i class="fa-solid fa-circle-info" style="color: red;"></i>
      <p>{notification}</p>
      <div id="notification_{index}" style="margin-left: 50%;">
        <i class="fa-solid fa-xmark"></i>
      </div>
    </div>
  {/each}
</div>

<style>
  .notification-container {
    position: absolute;
    bottom: 0;
    width: 100%;
  }
  .notification-container .notification {
    display: flex;
    background-color: black;
    color: white;
    font-family: "Poppins", system-ui;
    align-items: center;
    gap: 10px;
    padding: 10px;
    font-size: 14px;
    border-radius: 4px;
    margin-bottom: 10px;
    justify-content: space-between;
  }

  @media screen and (min-width: 768px) {
    .notification-container {
      top: 10vh;
      right: 0;
      width: 30vw;
    }
  }
</style>