<template>
  <div class="pdf-js-viewer-page-layer">
    <slot />
  </div>
</template>

<script>
export default {
  name: "PdfJsViewerPageLayer"
}
</script>

<style lang="scss" scoped>
.pdf-js-viewer-page-layer {
  background-color: rgba(0, 0, 0, .4);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>