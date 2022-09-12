<template>
  <div class="pdf-viewers">
    <b-tabs content-class="mt-3">
      <b-tab title="<object>">
        <object
            data="https://eloquentjavascript.net/Eloquent_JavaScript_small.pdf"
            type="application/pdf"
        >
          <p>
            PDF viewer non supportato,
            <a href="https://eloquentjavascript.net/Eloquent_JavaScript_small.pdf">scaricalo qui</a>
          </p>
        </object>
      </b-tab>

      <b-tab title="<embed>">
        <!-- No fallback content -->
        <embed
            src="https://eloquentjavascript.net/Eloquent_JavaScript_small.pdf"
            type="application/pdf"
        >
      </b-tab>

      <b-tab title="PDF.js">
        <!-- PDF.js cannot load files that do not satisfy the CORS restrictions.  -->
        <!-- So we need to use our local pdf -->
        <pdf-js-viewer url="./eloquent-js.pdf" filename="eloquent-js.pdf"/>
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import PdfJsViewer from "./PdfJsViewer.vue";

export default {
  name: "PdfNative",
  components: {PdfJsViewer}
}
</script>

<style lang="scss" scoped>
.pdf-viewers {
  object,
  embed {
    height: calc(100vh - 90px - 2rem);
    width: 100%;
  }
}
</style>