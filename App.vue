<template>
  <div>
    <b-tabs content-class="mt-3">
      <b-tab title="Viewers">
        <pdf-viewers/>
      </b-tab>

      <b-tab title="Edit (pdf-lib)" active>
        <pdf-edit-pdf-lib/>
      </b-tab>
    </b-tabs>
  </div>
</template>
<script>
import PdfViewers from "./components/PdfViewers.vue";
import PdfEditPdfLib from "./components/PdfEditPdfLib.vue";

export default {
  name: 'App',
  components: {PdfEditPdfLib, PdfViewers},
}
</script>