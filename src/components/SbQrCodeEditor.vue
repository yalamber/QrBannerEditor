<template>
  <div class="container">
    <button @click.prevent="addText">Add Text</button>
    <button class="btn btn-info" @click="onPickFile">Add Image</button>
    <input
      type="file"
      style="display: none"
      ref="fileInput"
      accept="image/*"
      @change="onFilePicked"
    />
    <button @click.prevent="saveImg">Save as Image</button>
    <div class="qrEditor">
      <FabricCanvas
        :height="height"
        :width="width"
        :backgroundColor="bgColor"
        @canvas-updated="updateCanvas"
      >
        <FabricText
          :id="text.id"
          v-for="text in texts"
          :key="'text-key-' + text.id"
          :text="text.text"
        ></FabricText>
        <FabricImageFromURL
          :id="img.id"
          v-for="img in images"
          :key="'img-key-' + img.id"
          :url="img.url"
        ></FabricImageFromURL>
        <FabricImageFromURL
          ref="qrCodeRef"
          id="qr-code"
          key="'qr-code'"
          :url="qrCode"
        ></FabricImageFromURL>
      </FabricCanvas>
    </div>
  </div>
</template>

<script>
import vueFabricWrapper from "vue-fabric-wrapper";

export default {
  name: "SbQrCodeEditor",
  props: {
    qrCode: String,
    width: Number,
    height: Number,
  },
  data: function () {
    return {
      canvas: null,
      images: [],
      texts: [],
      bgColor: "#FFFFFF",
    };
  },
  methods: {
    updateCanvas(e) {
      this.canvas = e;
      const qrCodeRef = this.$refs.qrCodeRef
      console.log(qrCodeRef);
    },
    addImage: function () {},
    addText: function () {
      console.log('here')
      const prompt = window.prompt('Please enter text')
      if (prompt) {
        this.texts = [...this.texts, {text: prompt, id: this.texts.length+1}]
      }
    },
    saveImg: function () {
      const canvas = this.canvas;
      const dataURL = canvas.toDataURL({
        width: canvas.width,
        height: canvas.height,
        left: 0,
        top: 0,
        format: "png",
      });
      const link = document.createElement("a");
      link.download = "your-qr.png";
      link.href = dataURL;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    onPickFile: function () {
      this.$refs.fileInput.click();
    },
    onFilePicked(event) {
      const vm = this
      const files = event.target.files;
      const filename = files[0].name;
      const fileReader = new FileReader();
      fileReader.readAsDataURL(files[0]);
      fileReader.addEventListener("load", () => {
        vm.images = [
          ...vm.images,
          { 
            url: fileReader.result, 
            id: `img-${filename}` 
          }
        ]
      });
    },
  },
  components: {
    FabricCanvas: vueFabricWrapper.FabricCanvas,
    FabricText: vueFabricWrapper.FabricText,
    FabricImageFromURL: vueFabricWrapper.FabricImageFromURL,
  },
};
</script>
<style scoped>
.qrEditor {
  margin: 10px;
  border: 1px dashed;
}
</style>
