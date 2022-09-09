<template>
  <div>
    <b-tabs content-class="mt-3">
      <b-tab title="Nativo">
        <pdf-native/>
      </b-tab>

      <b-tab title="PDF.js" active>
        <pdf-pdf-js/>
      </b-tab>

      <b-tab title="jsPDF">
        <p>J</p>
      </b-tab>
    </b-tabs>
  </div>
</template>
<script>
import PdfNative from "./components/PdfNative.vue";
import PdfPdfJs from "./components/PdfPdfJs.vue";

export default {
  name: 'App',
  components: {PdfPdfJs, PdfNative},
}
</script>