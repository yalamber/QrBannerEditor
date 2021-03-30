<template>
<div class="container">
    <div class="sidebar">
      <div class="modal" v-if="show">
        <a class="close-item" @click.prevent="cancelModal" ><font-awesome-icon icon="window-close" /></a>
        <div class="text-editor">
          <h2>Add Text</h2>
          <form v-on:submit.prevent="addText">
            <div class="text-edit-options">
              <select name="fontsize" v-model="texts.fontsize">
                <option :value="24">Font Size</option>
                <option 
                v-for="(fontSize, i) in fontSizes" 
                :key="i" 
                v-bind:value="fontSize">
                  {{fontSize}}
                </option>
              </select>
              <select name="fontfamily" v-model="texts.fontfamily">
                <option :value="Arial">Font Family</option>
                <option value="Tahoma">Tahoma</option>
                <option value="Trebuchet MS ">Trebuchet MS </option>
                <option value="Arial">Arial</option>
              </select>
            </div>
            <input type="text" placeholder="Text" name="text" v-model="texts.text"/> 
            <div class="actions">
              <input class="add" type="submit" value="Submit" />
              <input @click.prevent="cancelModal" class="cancel" type="submit" value="Cancel" />
            </div>
          </form>
        </div> 
      </div>
      <div class="buttons-wrapper">
        <button @click.prevent="showModal">
          <font-awesome-icon icon="font" />
        </button>
        <button class="btn btn-info" @click="onPickFile">
          <font-awesome-icon icon="image" />
        </button>
      </div>
      <!-- <button @click.prevent="addText"><font-awesome-icon icon="font" /> Add Text</button> -->
      <div class="canvas">
        <h2>Set Canvas</h2>
        <input
          type="number"
          placeholder="Width"
          max="1000"
          min=200
          @change="setCanvasWidth"
        />
        <input
          type="number"
          max="800"
          min=200
          placeholder="Height"
          @change="setCanvasHeight"
        />
        <label>Background Color:</label>
        <input 
          type="color" 
          @change="setCanvasColor"
          :value="bgColor"
        />
      </div>
      <div class="layers">
        <h2>Layers <font-awesome-icon icon="layer-group" /> </h2>
        <div v-for="text in texts"  :key="text.id">
          <div class="item" v-if="text.text">
            <div class="text">
              {{text.text}}
            </div>
            <a @click.prevent="deleteText(text.id)"><font-awesome-icon icon="trash" /></a>
          </div>
        </div>
        <div v-for="image in images" :key="image.id">
          <div class="item" v-if="image.url">
            <div class="thumb">
              <img :src="image.url" />
            </div>
            <a @click.prevent="deleteImage(image.id)"><font-awesome-icon icon="trash" /></a>
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
          <FabricImageFromURL
            ref="qrCodeRef"
            id="qr-code"
            key="'qr-code'"
            :url="qrCode"
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
        <button @click.prevent="saveImg"> <font-awesome-icon icon="save" /> Save as Image</button>
      </div>
    </div>
</div>
</template>

<script>
import vueFabricWrapper from "vue-fabric-wrapper"
export default {
  name: "SbQrCodeEditor",
  props: {
    qrCode: String
  },
  data: function () {
    return {
      show: false,
      canvas: null,
      canvasWidth: 350,
      canvasHeight: 450,
      images: [],
      texts: [{
        id: 0,
        text: '',
        fontsize: 24,
        fontfamily: 'Arial'
      }],
      bgColor: "#FFFFFF",
      fontSizes: [26, 28, 30, 32, 34, 36, 38, 39, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64]
    };
  },
  methods: {
    updateCanvas(e) {
      this.canvas = e;
      // const qrCodeRef = this.$refs.qrCodeRef
      // console.log(qrCodeRef);
    },
    showModal: function () {
      this.show = !this.show
    },
    cancelModal: function () {
      this.show = false
    },
    addText: function (e) {
      console.log('here')
      console.log(this.texts)
      // const prompt = window.prompt('Please enter text')
      // if (prompt) {
      //   this.texts = [...this.texts, {text: prompt, id: this.texts.length+1}]
      // }
      // if(this.texts.text){
      //   this.texts = [...this.texts, {text: this.texts.text, id: this.texts.length+1}]
      // }
      if(this.texts){
         this.texts = [...this.texts, {id: this.texts.length+1, ...this.texts}]
      }  
      this.show = false
      e.target.reset()
    },
    deleteText: function(id) {
      const remainingArr = this.texts.filter(text => text.id != id);
      this.texts = remainingArr
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
    deleteImage: function(id) {
      const remainingArr = this.images.filter(image => image.id != id);
      this.images = remainingArr
    },
    setCanvasWidth: function (e) {
      const width = parseInt(e.target.value)
      if(width >= 200 && width <= 1000 ){
        this.canvasWidth = width
      }
    },
    setCanvasHeight: function (e) {
      const height = parseInt(e.target.value)
      if(height >= 200 && height <= 800 ){
        this.canvasHeight = height
      }
    },
    setCanvasColor: function (e) {
      this.bgColor = e.target.value
    }
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
.container {
  display: flex;
  height: 100vh;
  margin: 0px;
  padding: 0px;
}
.sidebar {
  background-color: #535353;
  padding: 15px 10px;
  height: 100%;
  width: 150px;
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
.layers h2, .canvas h2 {
  font-size: 16px;
  color: #fff;
  margin-bottom: 5px;
}
.canvas input[type="number"] {
  display: inline-block;
  width: 44%;
}
.canvas label{
  color: #fff;
  font-size: 12px;
  padding: 7px 0;
  display: block;
}
.canvas input[type="color"] {
  width: 96%;
}
/* Modal */
.modal {
  position: absolute;
  background: rgba(0, 0, 0, 0.8);
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
  display: flex;
  justify-content: center;
  align-items: center;  
  flex-direction: column;
  padding: 30px 40px;
  background: rgba(109, 107, 107, 0.8);
}
.text-editor h2 {
  color: #fff;
  margin: 0px 0 10px 0;
}
.buttons-wrapper button {
  width: 45%;
  font-size: 32px;
  cursor: pointer;
}
.buttons-wrapper button + button {
  margin-left: 10%;
}
.actions{
  display: flex;
  margin-top: 20px;
  justify-content: space-between;
}
.actions .add, .actions .cancel, .buttons button {
  background: rgb(4, 124, 4);
  border: none;
  outline: none;
  color: #fff;
  padding: 10px 15px;
  cursor: pointer;
  transition: all .3s cubic-bezier(.645,.045,.355,1);
} 
.actions .cancel {
  background: rgb(255, 0, 0);
}
.actions .add:hover, .buttons button:hover {
  background: rgb(55, 172, 55);
}
.actions .cancel:hover {
  background: rgb(204, 83, 83);
}
.text-edit-options select, .text-editor input[type="text"] {
  height: 30px;
  outline: none;
  border: none;
}
.text-editor input[type="text"] {
  margin-top: 10px;
  width: 96%;
  padding: 0 2%;
}
.text-edit-options select + select {
  margin-left: 10px;
}
a.close-item {
  position: absolute;
  top: 10px;
  right: 15px;
  color: #fff;
  font-size: 32px;
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
  transition: all .3s cubic-bezier(.645,.045,.355,1);
} 
.item a:hover{
  color: rgb(255, 0, 0);
}
.thumb {
  max-width: 40px;
  max-height: 40px;
}
.thumb img {
  max-width: 100%;
}
</style>
