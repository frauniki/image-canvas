<template>
  <div>
    <div>
      <label>
        <input type="file" @change="onFileChange" />
        <button v-on:click="onUpload">Image Add</button>
      </label>
    </div>
    <div>
      <label>
        <button v-on:click="onDownload">Canvas Download</button>
      </label>
    </div>
    <div>
      <v-stage
        style="width:400px; height:400px; border: 2px solid #000000;"
        ref="stage"
        :config="stageSize"
        @mousedown="handleStageMouseDown"
        @touchstart="handleStageMouseDown"
      >
        <v-layer ref="layer">
          <v-image
            v-for="item in images"
            :key="item.id"
            :config="item"
            @transformend="handleTransformEnd"
          />
          <v-transformer ref="transformer" />
        </v-layer>
      </v-stage>
    </div>
  </div>
</template>

<script>
import { v4 as uuidv4 } from "uuid";

export default {
  data() {
    return {
      uploadFile: null,
      images: [],
      stageSize: {
        width: 400,
        height: 400,
      },
      selectedShapeName: "",
    };
  },
  methods: {
    createImage(file, name) {
      const reader = new FileReader();
      reader.onload = (e) => {
        var image = new Image();
        image.src = e.target.result;

        this.images.push({
          id: uuidv4(),
          image,
          rotation: 0,
          x: 10,
          y: 10,
          width: 100,
          height: 100,
          scaleX: 1,
          scaleY: 1,
          name: name.replace(" ", ""),
          draggable: true,
        });
      };
      reader.readAsDataURL(file);
    },
    onFileChange(event) {
      this.uploadFile = event.target.files || event.dataTransfer.files;
    },
    onUpload() {
      this.createImage(this.uploadFile[0], this.uploadFile[0].name);
    },

    onDownload() {
      const transformerNode = this.$refs.transformer.getNode();
      const dataURL = transformerNode.getStage().toDataURL();

      let link = document.createElement("a");
      link.download = "stage.png";
      link.href = dataURL;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    handleTransformEnd(e) {
      const rect = this.images.find((r) => r.name === this.selectedShapeName);

      rect.x = e.target.x();
      rect.y = e.target.y();
      rect.rotation = e.target.rotation();
      rect.scaleX = e.target.scaleX();
      rect.scaleY = e.target.scaleY();
    },
    handleStageMouseDown(e) {
      if (e.target === e.target.getStage()) {
        this.selectedShapeName = "";
        this.updateTransformer();
        return;
      }

      const clickedOnTransformer =
        e.target.getParent().className === "Transformer";
      if (clickedOnTransformer) {
        return;
      }

      const name = e.target.name();
      const rect = this.images.find((r) => r.name === name);
      if (rect) {
        this.selectedShapeName = name;
      } else {
        this.selectedShapeName = "";
      }
      this.updateTransformer();
    },
    updateTransformer() {
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const { selectedShapeName } = this;

      const selectedNode = stage.findOne("." + selectedShapeName);

      if (selectedNode === transformerNode.node()) {
        return;
      }

      if (selectedNode) {
        transformerNode.nodes([selectedNode]);
      } else {
        transformerNode.nodes([]);
      }
      transformerNode.getLayer().batchDraw();
    },
  },
};
</script>
