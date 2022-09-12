<template>
  <pdf-js-viewer-page-layer class="pdf-je-viewer-page-layer-shapes">
    <canvas ref="canvas"/>
    <b-button @click="onAddShape" style="position: absolute; top: 0; left: 0">Add shape</b-button>
  </pdf-js-viewer-page-layer>
</template>

<script>
import PdfJsViewerPageLayer from "./PdfJsViewerPageLayer.vue";
import {fabric} from 'fabric'

export default {
  name: "PdfViewerPageOverlayShapes",
  components: {PdfJsViewerPageLayer},
  data() {
    return {
      canvasFabric: null
    }
  },
  mounted() {
    let canvas = this.$refs.canvas;
    let options = {selection: false, defaultCursor: ''};
    this.canvasFabric = new fabric.Canvas(canvas, options);
    this.addResizeObserver();
  },
  beforeDestroy() {
    this.removeResizeObserver();
  },
  methods: {
    addResizeObserver() {
      this.resizeObserver = new ResizeObserver(entries => {
        let rect = entries[0]?.contentRect;
        if (!rect) return
        this.canvasFabric.setWidth(rect.width);
        this.canvasFabric.setHeight(rect.height);
      })


      this.resizeObserver.observe(this.$el);
    },
    removeResizeObserver() {
      this.resizeObserver.disconnect();
    },
    onAddShape() {
      let rect = new fabric.Rect({
        top: Math.random() * this.canvasFabric.getHeight(),
        left: Math.random() * this.canvasFabric.getWidth(),
        width: 60,
        height: 70,
        fill: 'red'
      });

      this.canvasFabric.add(rect);
    },
    toImageData({format = 'png', quality = 1, multiplier = 1}) {
      // @see http://fabricjs.com/docs/fabric.StaticCanvas.html#toDataURL
      return this.canvasFabric.toDataURL({format, quality, multiplier})
    }
  }
}
</script>

<style lang="scss" scoped>
.pdf-je-viewer-page-layer-shapes {
  canvas {
    width: 100%;
    height: 100%;
  }
}
</style>