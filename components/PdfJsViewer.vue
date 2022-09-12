<template>
  <div class="pdf-js-viewer" :class="`pdf-js-viewer--layout-${layout}`">
    <b-container fluid class="pdf-js-viewer__container">
      <b-row align-v="center" class="pdf-js-viewer__header">
        <b-col cols="auto" class="pdf-js-viewer__menu">
          <b-button variant="outline" class="text-white" @click="toggleSidebar">
            <i class="fa fa-bars"></i>
          </b-button>
        </b-col>
        <b-col class="pdf-js-viewer__header-title">
          {{ filename }}
        </b-col>
        <b-col cols="auto" class="pdf-js-viewer__header-action-list">
          <slot name="header-actions"/>
          <b-dropdown id="dropdown-1" text="Layout" class="m-md-2">
            <b-dropdown-item @click="onChangeLayout(LAYOUTS.ONE)">
              One page
            </b-dropdown-item>
            <b-dropdown-item @click="onChangeLayout(LAYOUTS.TWO)">
              Two pages
            </b-dropdown-item>
          </b-dropdown>

          <slot name="header-action-download">
            <b-button :href="url" :download="filename">Download</b-button>
          </slot>
        </b-col>
      </b-row>

      <b-row>
        <template v-if="isSidebarVisible">
          <b-col cols="3" class="pdf-js-viewer__sidebar">
            <pdf-js-viewer-outline-item
                v-for="item in documentOutline"
                :key="item.title"
                :title="item.title"
                :children="item.items"
                @click.native="onOutlineItemClick(item)"
                @child-click="onOutlineItemClick"
            />
          </b-col>
        </template>

        <b-col class="pdf-js-viewer__page-list">
          <b-row>
            <b-col
                v-for="(page, index) in pageList"
                :key="page.pageNumber"
                cols="auto"
                class="pdf-js-viewer__page-col"
            >
              <div ref="page" class="pdf-js-viewer__page">
                <canvas/>
                <slot name="page-after" v-bind="{page, document, index}"></slot>
              </div>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import * as pdfJs from 'pdfjs-dist/legacy/build/pdf.js'
import PdfJsViewerOutlineItem from "./PdfJsViewerOutlineItem.vue";

// {PDFWorker} The worker that will be used for loading and parsing the PDF data.
pdfJs.GlobalWorkerOptions.workerSrc = `https://cdn.jsdelivr.net/npm/pdfjs-dist@${pdfJs.version}/build/pdf.worker.min.js`;

const LAYOUTS = {
  ONE: 'one-page',
  TWO: 'two-pages'
}

export default {
  name: "PdfJsViewer",
  components: {PdfJsViewerOutlineItem},
  props: {
    url: {Type: String, default: ''},
    filename: {Type: String, default: ''}
  },
  data() {
    return {
      LAYOUTS,
      layout: LAYOUTS.ONE,
      isSidebarVisible: true,
      document: null,
      documentOutline: [],
      pageMap: {}
    }
  },
  computed: {
    pageList() {
      let result = Object.values(this.pageMap)
      result.sort((a, b) => a.pageNumber > b.pageNumber);
      return result;
    }
  },
  watch: {
    url: {
      immediate: true,
      handler(newValue) {
        if (!newValue) return;
        this.loadDocument();
      }
    }
  },
  methods: {
    toggleSidebar() {
      this.isSidebarVisible = !this.isSidebarVisible
    },
    onChangeLayout(layout) {
      this.layout = layout;
      this.pageList.forEach(page => this.renderPage(page));
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
      this.document = await pdfJs.getDocument(this.url).promise;
      this.loadPages();
      this.documentOutline = await this.document.getOutline();
    },
    async loadPages() {
      let pageCount = this.document.numPages;
      // @TODO: Limit for performance reason
      pageCount = Math.min(pageCount, 50);

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

      // canvas.width = Math.floor(viewport.width * outputScale);
      // canvas.height = Math.floor(viewport.height * outputScale);
      // canvas.style.width = Math.floor(viewport.width) + "px";
      // canvas.style.height = Math.floor(viewport.height) + "px";

      let width = Math.floor(viewport.width * outputScale);
      let height = Math.floor(viewport.height * outputScale);
      pageRef.style.width = `${width}px`;
      pageRef.style.height = `${height}px`;
      canvas.width = width;
      canvas.height = height;

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
.pdf-js-viewer {
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

  &__page-col {
    padding: 8px 0;
  }

  &__page {
    position: relative;
    text-align: center;
    margin: auto;
  }

  &--layout-one-page {
    #{$root}__page-col {
      width: 100%;
    }
  }

  &--layout-two-pages {
    #{$root}__page-col {
      width: 50%;
    }
  }

}
</style>