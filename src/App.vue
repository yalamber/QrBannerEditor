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
    QRCode.toDataURL('This is testing', function (err, url) {
      vm.qrCode = url
    })
  },
  methods: {
   
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
  color: #2c3e50;
  width: 100%;
  height: 100vh;
}
</style>
