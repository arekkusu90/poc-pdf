<template>
  <div class="pdf-edit-pdf-lib">
    <b-button @click="onTextAdd">Click me to add a text in the PDF</b-button>

    <b-tabs>
      <b-tab title="<object>" active>
        <object
            :data="url"
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
            :src="url"
            type="application/pdf"
        >
      </b-tab>

      <b-tab title="PDF.js">
        <pdf-js-viewer :url="url" :filename="filename">
          <template #header-action-download>
            <b-button @click="onDownload">Download</b-button>
          </template>
          <template #page-after="{page, document, index}">
            <pdf-viewer-page-overlay-shapes :ref="`shape-${index}`"/>
          </template>
        </pdf-js-viewer>
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import {PDFDocument, rgb, StandardFonts, degrees} from 'pdf-lib';
import PdfJsViewer from "./PdfJsViewer.vue";
import PdfViewerPageOverlayShapes from "./PdfViewerPageLayerShapes.vue";
import download from "downloadjs";

export default {
  name: "PdfPdfLib",
  components: {PdfViewerPageOverlayShapes, PdfJsViewer},
  data() {
    return {
      url: "",
      pdf: null,
      fonts: {}
    }
  },
  computed: {
    filename() {
      return 'eloquent-js.pdf'
    }
  },
  async mounted() {
    let url = './eloquent-js.pdf'
    let response = await fetch(url)
    let pdfBytes = await response.arrayBuffer();

    this.pdf = await PDFDocument.load(pdfBytes);
    this.fonts[StandardFonts.Helvetica] = await this.pdf.embedFont(StandardFonts.Helvetica);
    this.updateUrl();
  },
  methods: {
    async getPdf(url) {
      let response = await fetch(url)
      let pdfBytes = await response.arrayBuffer();
      return await PDFDocument.load(pdfBytes);
    },
    async updateUrl() {
      let bytes = await this.pdf.save();
      let blob = new Blob([bytes], {type: 'application/pdf'});
      this.url = URL.createObjectURL(blob);
      this.pdf = await PDFDocument.load(bytes);
    },
    onTextAdd() {
      let pages = this.pdf.getPages();
      let firstPage = pages[0];
      let {width, height} = firstPage.getSize()
      firstPage.drawText('This text was added with JavaScript!', {
        x: 5,
        y: height - 30,
        size: 24,
        font: this.fonts[StandardFonts.Helvetica],
        color: rgb(0.95, 0.1, 0.1),
        rotate: degrees(0),
      })

      this.updateUrl();
    },
    async onDownload() {
      // We want to apply the shapes on a fresh pdf copy
      let pdf = await this.getPdf(this.url);
      let pages = pdf.getPages();

      for (let i = 0; i < pages.length; i++) {
        let page = pages[i];
        let ref = this.$refs[`shape-${i}`];

        if (ref) {
          let imageData = ref.toImageData({});
          let image = await pdf.embedPng(imageData);
          let {width, height} = image.scaleToFit(page.getWidth(), page.getHeight());
          page.drawImage(image, {x: 0, y: 0, width, height,})
        }
      }

      let bytes = await pdf.save();
      download(bytes, this.filename, 'application/pdf');
    },
  }
}
</script>

<style lang="scss" scoped>
.pdf-edit-pdf-lib {
  object,
  embed {
    height: calc(100vh - 90px - 2rem - 22px);
    width: 100%;
  }
}
</style>