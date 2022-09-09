<template>
  <div class="pdf-pdf-js" :class="`pdf-pdf-js--layout-${layout}`">
    <b-container fluid class="pdf-pdf-js__container">
      <b-row align-v="center" class="pdf-pdf-js__header">
        <b-col cols="auto" class="pdf-pdf-js__menu">
          <b-button variant="outline" class="text-white" @click="toggleSidebar">
            <i class="fa fa-bars"></i>
          </b-button>
        </b-col>
        <b-col class="pdf-pdf-js__header-title">
          {{ documentFilename }}
        </b-col>
        <b-col cols="auto" class="pdf-pdf-js__header-action-list">
          <b-dropdown id="dropdown-1" text="Layout" class="m-md-2">
            <b-dropdown-item @click="onChangeLayout(LAYOUTS.ONE)">
              One page
            </b-dropdown-item>
            <b-dropdown-item @click="onChangeLayout(LAYOUTS.TWO)">
              Two pages
            </b-dropdown-item>
          </b-dropdown>

          <b-button :href="documentUrl" download>Download</b-button>
        </b-col>
      </b-row>

      <b-row>
        <template v-if="isSidebarVisible">
          <b-col cols="3" class="pdf-pdf-js__sidebar">
            <pdf-pdf-js-outline-item
                v-for="item in documentOutline"
                :key="item.title"
                :title="item.title"
                :children="item.items"
                @click.native="onOutlineItemClick(item)"
                @child-click="onOutlineItemClick"
            />
          </b-col>
        </template>

        <b-col class="pdf-pdf-js__page-list">
          <b-row>
            <b-col
                ref="page"
                v-for="page in pageList"
                :key="page.pageNumber"
                cols="auto"
                class="pdf-pdf-js__page"
            >
              <canvas/>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import * as pdfJs from 'pdfjs-dist/legacy/build/pdf.js'
import PdfPdfJsOutlineItem from "./PdfPdfJsOutlineItem.vue";

pdfJs.GlobalWorkerOptions.workerSrc = `https://cdn.jsdelivr.net/npm/pdfjs-dist@${pdfJs.version}/build/pdf.worker.min.js`;

const LAYOUTS = {
  ONE: 'one-page',
  TWO: 'two-pages'
}

export default {
  name: "PdfPdfJs",
  components: {PdfPdfJsOutlineItem},
  data() {
    return {
      g: {},
      LAYOUTS,
      isSidebarVisible: true,
      documentUrl: "./eloquent-js.pdf",
      document: null,
      documentOutline: [],
      pageMap: {},
      layout: LAYOUTS.ONE
    }
  },
  computed: {
    documentFilename() {
      return this.documentUrl.split('/').reverse()[0];
    },
    pageList() {
      let result = Object.values(this.pageMap)
      result.sort((a, b) => a.pageNumber > b.pageNumber);
      return result;
    }
  },
  async mounted() {
    this.loadDocument();
  },
  methods: {
    onChangeLayout(layout) {
      this.layout = layout;
      this.pageList.forEach(page => this.renderPage(page));
    },
    toggleSidebar() {
      this.isSidebarVisible = !this.isSidebarVisible
    },
    getPageRef(pageNumber) {
      return this.$refs.page[pageNumber - 1];
    },
    scrollToPage(pageNumber) {
      let pageRef = this.getPageRef(pageNumber);
      pageRef.scrollIntoView();
    },
    async onOutlineItemClick(item) {
      let [ref] = await this.document.getDestination(item.dest);
      let pageNumber = await this.document.getPageIndex(ref);
      this.scrollToPage(pageNumber);
    },
    async loadDocument() {
      this.document = await pdfJs.getDocument(this.documentUrl).promise;
      window.clx = this.$data;
      this.loadPages();
      this.documentOutline = await this.document.getOutline();
      // this.g.pageMode = await this.document.getPageMode();
      // this.g.viewerPreferences = await this.document.getViewerPreferences();
    },
    async loadPages() {
      let pageCount = this.document.numPages;
      // @TODO: limitiamo a 20 per sviluppo
      pageCount = Math.min(pageCount, 20);

      for (let i = 1; i <= pageCount; i++) {
        this.loadPage(i);
      }
    },
    async loadPage(pageNumber) {
      let page = await this.document.getPage(pageNumber);
      this.pageMap = {...this.pageMap, [page.pageNumber]: page};
      await this.$nextTick();
      this.renderPage(page);
    },
    async renderPage(page) {
      let pageRef = this.getPageRef(page.pageNumber);

      let canvas = pageRef.querySelector('canvas');
      let canvasContext = canvas.getContext('2d');

      let scale = this.layout === LAYOUTS.ONE ? 1.5 : 1;
      let viewport = page.getViewport({scale});
      // Support HiDPI-screens.
      let outputScale = window.devicePixelRatio || 1;

      canvas.width = Math.floor(viewport.width * outputScale);
      canvas.height = Math.floor(viewport.height * outputScale);
      canvas.style.width = Math.floor(viewport.width) + "px";
      canvas.style.height = Math.floor(viewport.height) + "px";

      let transform = null;
      if (outputScale !== 1) transform = [outputScale, 0, 0, outputScale, 0, 0];

      let renderContext = {
        canvasContext,
        transform,
        viewport
      };

      page.render(renderContext);
    },
  }
}
</script>

<style lang="scss" scoped>
.pdf-pdf-js {
  $root: &;

  &__header {
    background-color: var(--gray-800);
    color: #FFFFFF;
    padding: 8px;
  }

  &__header-title {
    font-weight: bold;
  }

  &__sidebar {
    background-color: var(--gray-800);
    color: #FFFFFF;
    padding: 16px;
    max-width: 20%;
    word-break: break-word;
  }

  &__page-list {
    background-color: var(--gray-700);
    padding: 16px;
  }

  &--layout-one-page {
    #{$root}__page {
      width: 100%;
      text-align: center;
      padding: 8px 0;
    }
  }

  &--layout-two-pages {
    #{$root}__page {
      width: 50%;
      text-align: center;
    }
  }

}
</style>