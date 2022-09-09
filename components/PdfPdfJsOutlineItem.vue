<template>
  <div class="pdf-pdf-js-outline-item">
    <b-row align-v="baseline">
      <b-col cols="2" class="pl-0">
        <template v-if="children.length > 0">
          <b-button
              variant="outline"
              class="text-white"
              @click.stop="toggleChildrenGroup"
          >
            <template v-if="!isChildrenGroupVisible">
              <i class="fa fa-caret-right"></i>
            </template>
            <template v-else>
              <i class="fa fa-caret-down"></i>
            </template>
          </b-button>
        </template>
      </b-col>
      <b-col class="pl-0">{{ title }}</b-col>
    </b-row>

    <template v-if="children.length > 0">
      <b-collapse v-model="isChildrenGroupVisible" class="mt-2">
        <pdf-pdf-js-outline-item
            v-for="child in children"
            :key="child.title"
            :title="child.title"
            class="pl-4 py-2"
            @click.native.stop="onChildClick(child)"
        />
      </b-collapse>
    </template>
  </div>
</template>

<script>

export default {
  name: "PdfPdfJsOutlineItem",
  props: {
    title: {type: String, default: ""},
    children: {type: Array, default: () => []}
  },
  data() {
    return {
      isChildrenGroupVisible: false,
    }
  },
  methods: {
    toggleChildrenGroup() {
      this.isChildrenGroupVisible = !this.isChildrenGroupVisible
    },
    onChildClick(child) {
      this.$emit('child-click', child);
    }
  }
}
</script>

<style lang="scss" scoped>
</style>