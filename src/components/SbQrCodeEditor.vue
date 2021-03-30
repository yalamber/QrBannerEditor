<template>
  <div class="container">
    <div class="sidebar">
      <div class="modal" v-if="show">
        <div class="text-editor">
          <a class="close-item" @click.prevent="cancelModal"
            ><font-awesome-icon icon="window-close"
          /></a>
          <h2>Add Text</h2>
          <form v-on:submit.prevent="addText">
            <div class="text-edit-options">
              <select name="fontsize" v-model.number="texts.fontsize">
                <option :value="24">Font Size</option>
                <option
                  v-for="(fontSize, i) in fontSizes"
                  :key="i"
                  v-bind:value="fontSize"
                >
                  {{ fontSize }}
                </option>
              </select>
              <select name="fontfamily" v-model="texts.fontfamily">
                <option :value="Arial">Font Family</option>
                <option value="Tahoma">Tahoma</option>
                <option value="Trebuchet MS">Trebuchet MS</option>
                <option value="Arial">Arial</option>
              </select>
            </div>
            <input
              type="text"
              placeholder="Text"
              name="text"
              class="text-field"
              v-model="texts.text"
            />
            <div class="actions">
              <input class="add" type="submit" value="Insert" />
            </div>
          </form>
        </div>
      </div>
      <div class="document-controls">
        <h2>Set Document Size</h2>
        Width:
        <input
          type="number"
          placeholder="Width"
          max="1000"
          min="200"
          v-model="canvasWidth"
        />
        Height:
        <input
          type="number"
          max="800"
          min="200"
          placeholder="Height"
          v-model="canvasHeight"
        />
        <label>Background Color:</label>
        <input type="color" @change="setCanvasColor" :value="bgColor" />
      </div>
      <hr class="separator" />
      <div class="buttons-wrapper">
        Insert
        <button title="Text" @click.prevent="showModal">
          <font-awesome-icon icon="font" />
        </button>
        <button title="Image" class="btn btn-info" @click="onPickFile">
          <font-awesome-icon icon="image" />
        </button>
      </div>
      <hr class="separator" />
      <div class="layers">
        <h2>Layers <font-awesome-icon icon="layer-group" /></h2>
        <div v-for="text in texts" :key="text.id">
          <div class="item" v-if="text.text">
            <div class="text">
              {{ text.text }}
            </div>
            <a @click.prevent="deleteText(text.id)"
              ><font-awesome-icon icon="trash"
            /></a>
          </div>
        </div>
        <div v-for="image in images" :key="image.id">
          <div class="item" v-if="image.url">
            <div class="thumb">
              <img :src="image.url" />
            </div>
            <a @click.prevent="deleteImage(image.id)"
              ><font-awesome-icon icon="trash"
            /></a>
          </div>
        </div>
      </div>
    </div>
    <div class="main">
      <div class="qrEditor">
        <FabricCanvas
          :height="canvasHeight"
          :width="canvasWidth"
          :backgroundColor="bgColor"
          @canvas-updated="updateCanvas"
        >
          <FabricImageFromURL
            ref="qrCodeRef"
            id="qr-code"
            key="'qr-code'"
            :url="qrCode"
          ></FabricImageFromURL>
          <FabricText
            :id="text.id"
            v-for="text in texts"
            :key="'text-key-' + text.id"
            :text="text.text"
            :fontSize="text.fontsize"
            :fontFamily="text.fontfamily"
          ></FabricText>
          <FabricImageFromURL
            :id="img.id"
            v-for="img in images"
            :key="'img-key-' + img.id"
            :url="img.url"
          ></FabricImageFromURL>
        </FabricCanvas>
      </div>
      <div class="buttons">
        <input
          type="file"
          style="display: none"
          ref="fileInput"
          accept="image/*"
          @change="onFilePicked"
        />
        <button @click.prevent="saveImg">
          <font-awesome-icon icon="save" /> Save as Image
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import vueFabricWrapper from "vue-fabric-wrapper";
export default {
  name: "SbQrCodeEditor",
  props: {
    qrCode: String,
  },
  data: function () {
    return {
      show: false,
      canvas: null,
      canvasWidth: 350,
      canvasHeight: 450,
      images: [],
      texts: [
        {
          id: 0,
          text: "",
          fontsize: 24,
          fontfamily: "Arial",
        },
      ],
      bgColor: "#FFFFFF",
      fontSizes: [
        26,
        28,
        30,
        32,
        34,
        36,
        38,
        39,
        40,
      ],
    };
  },
  methods: {
    updateCanvas(e) {
      this.canvas = e;
    },
    showModal: function () {
      this.show = !this.show;
    },
    cancelModal: function () {
      this.show = false;
    },
    addText: function (e) {
      if (this.texts) {
        this.texts = [
          ...this.texts,
          { id: this.texts.length + 1, ...this.texts },
        ];
      }
      this.show = false;
      e.target.reset();
    },
    deleteText: function (id) {
      const remainingArr = this.texts.filter((text) => text.id != id);
      this.texts = remainingArr;
    },
    saveImg: function () {
      const canvas = this.canvas;
      const dataURL = canvas.toDataURL({
        width: this.canvasWidth,
        height: this.canvasHeight,
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
      const vm = this;
      const files = event.target.files;
      const filename = files[0].name;
      const fileReader = new FileReader();
      fileReader.readAsDataURL(files[0]);
      fileReader.addEventListener("load", () => {
        vm.images = [
          ...vm.images,
          {
            url: fileReader.result,
            id: `img-${filename}`,
          },
        ];
      });
    },
    deleteImage: function (id) {
      const remainingArr = this.images.filter((image) => image.id != id);
      this.images = remainingArr;
    },
    setCanvasColor: function (e) {
      this.bgColor = e.target.value;
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
  border: 1px dashed #000;
}
.container {
  display: flex;
  height: 100vh;
  margin: 0px;
  padding: 0px;
}
.sidebar {
  border-right: 1px solid #ccc;
  background: #1e1f22;
  color: #fff;
  padding: 25px;
  height: 100%;
  width: 350px;
}
.main {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  flex-direction: column;
  width: 100%;
  height: 100%;
}
.layers {
  margin-top: 10px;
}
.layers h2,
.document-controls h2 {
  font-size: 14px;
  color: #fff;
  margin-bottom: 5px;
}
.document-controls input[type="number"] {
  display: inline-block;
  padding: 5px;
}
.document-controls label {
  color: #fff;
  font-size: 12px;
  padding: 7px 0;
  display: block;
}

/* Modal */
.modal {
  position: absolute;
  background: rgba(0, 0, 0, 0.8);
  color: #000;
  width: 100%;
  height: 100%;
  z-index: 999;
  top: 0px;
  left: 0px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.text-editor {
  position: relative;
  display: flex;
  justify-content: center;
  flex-direction: column;
  padding: 30px 40px;
  border-radius: 5px;
  background: #fff;
}
.text-editor h2 {
  color: #000;
  margin: 0px 0 10px 0;
}
.buttons-wrapper button {
  font-size: 12px;
  cursor: pointer;
}
.actions {
  display: flex;
  align-items: flex-end;
}
.actions .add,
.buttons button {
  border: none;
  outline: none;
  background: #1e1f22;
  color: #fff;
  padding: 10px 15px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
}
.actions .add:hover,
.buttons button:hover {
  background: #000;
}
.text-edit-options select,
.text-editor input[type="text"] {
  outline: none;
  border: none;
}
.text-editor input[type="text"] {
  width: 300px;
  margin: 10px 0;
  border: 1px solid #EEE;
  padding: 10px;
}
.text-edit-options select + select {
  margin-left: 10px;
}
a.close-item {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 22px;
  cursor: pointer;
}
.item {
  display: flex;
  color: #fff;
  justify-content: space-between;
  align-items: center;
  background: #696565;
  margin: 5px 0;
  padding: 5px 7px;
}
.item .text {
  font-size: 14px;
}
.item a {
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
}
.item a:hover {
  color: rgb(255, 0, 0);
}
.thumb {
  max-width: 40px;
  max-height: 40px;
}
.thumb img {
  max-width: 100%;
}
.separator {
  border: none;
  height: 1px;
  color: rgb(59, 59, 59);
  background-color: rgb(59, 59, 59);
  margin: 15px 0;
}
</style>
