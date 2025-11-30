<template>
  <div id="app">
    <div class="background-overlay"></div>
    <div class="content-wrapper">
      <CryptoList />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import CryptoList from "./components/CryptoList.vue";

export default defineComponent({
  name: "App",
  components: {
    CryptoList,
  },
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  height: 100%;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    "Helvetica Neue", Arial, sans-serif;
}

#app {
  position: relative;
  min-height: 100vh;
  overflow-x: hidden;
}

/* Background Image Layer */
#app::before {
  content: "";
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150%;
  height: 150%;
  background-image: url(./assets/bg-web.webp);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  z-index: 0;
  filter: brightness(0.7);
}

/* Dark Overlay for better readability */
.background-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  z-index: 1;
  pointer-events: none;
}

/* Content Layer */
.content-wrapper {
  position: relative;
  z-index: 2;
  padding: 30px 20px;
  min-height: 100vh;
}

/* Scrollbar styling */
::-webkit-scrollbar {
  width: 12px;
  height: 12px;
}

::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.4);
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.6);
}
</style>