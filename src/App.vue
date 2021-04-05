<template>
  <div id="app">
    <div class="appContainer">
      <SbQrCodeEditor :qrCode="qrCode" />
    </div>
  </div>
</template>

<script>
import QRCode from 'qrcode'
import SbQrCodeEditor from './components/SbQrCodeEditor.vue'

export default {
  name: 'App',
  data: function (){
    return {
      qrCode: ''
    }
  },
  components: {
    SbQrCodeEditor
  },
  created: function () {
    const vm = this
    QRCode.toDataURL('This is testing', { margin: 2, width: 150, height: 150 }, function (err, url) {
      vm.qrCode = url
    })
  }
}
</script>

<style>
body {
  margin: 0px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #FFF;
  width: 100%;
  height: 100vh;
}
</style>
