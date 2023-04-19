<template>
  <div id="app">
    <router-view />
    <div id="mintFooter" class="container">
      Powered by
      <a href="https://4everland.org/" target="_blank" style="margin-left: 10px"
        >4EVERLAND</a
      >
    </div>
  </div>
</template>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
* {
  padding: 0;
  margin: 0;
}
html {
  background-color: #f9fbfc;
}
#mintFooter {
  display: flex;
  align-items: center;
  height: 100px;
  font-size: 14px;
  color: #6c7789;
  justify-content: center;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  a {
    font-size: 14px;
    color: #2a7eed;
    text-decoration: none;
  }
}
</style>
