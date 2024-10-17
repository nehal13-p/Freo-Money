<script lang="ts">
  import Shimmer from './components/Shimmer.svelte';
  import TopIframe from './components/TopIframe.svelte';
  import MainBox from './components/MainBox.svelte';
  import BottomIframe from './components/BottomIframe.svelte';
  import Notifications from './components/Notifications.svelte';

  let topIframeUrl = 'https://freo.money/webqual/top';
  let bottomIframeUrl = 'https://freo.money/webqual/bottom';
  
  let notifications: string[] = [];

  function addNotification(message: string) {
    notifications = [...notifications, message];
    setTimeout(() => {
      notifications = notifications.slice(1);
    }, 4000);
  }
</script>

<Shimmer />
<TopIframe url={topIframeUrl} />
<MainBox {addNotification} />
<BottomIframe url={bottomIframeUrl} />
<Notifications {notifications} />

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    background-color: rgb(19, 22, 28);
    display: flex;
    flex-direction: column;
    font-family: "Poppins", system-ui;
  }
</style>