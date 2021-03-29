<template>
  <div id="app">
    <SbQrCodeEditor :width="350" :height="450" :qrCode="qrCode" />
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
    QRCode.toDataURL('This is testing', function (err, url) {
      vm.qrCode = url
    })
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
}
</style>
