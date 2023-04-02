<script setup>
</script>

<template>
  <div id="app">
    <router-view />
  </div>
</template>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  /* 好看的滚动条-开始 */
  *::-webkit-scrollbar {
    width: 12px;
    height: 12px;
  }
  *::-webkit-scrollbar-button {
    width: 0;
    height: 0;
    display: none;
  }
  *::-webkit-scrollbar-corner {
    background-color: transparent;
  }
  *::-webkit-scrollbar-thumb {
    border: 4px solid rgba(0, 0, 0, 0);
    height: 6px;
    border-radius: 25px;
    background-clip: padding-box;
    background-color: rgba(0, 0, 0, 0.30);
  }
  /* 好看的滚动条-结束 */
}
#app {
  height: 100vh;
}
</style>
