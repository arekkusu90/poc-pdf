<template>
  <div class="pdf-native">
    <b-tabs content-class="mt-3">
      <b-tab title="<object>">
        <object
            data="https://eloquentjavascript.net/Eloquent_JavaScript_small.pdf"
            type="application/pdf"
            class="pdf-native__object"
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
            class="pdf-native__embed"
        >
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
export default {
  name: "PdfNative"
}
</script>

<style lang="scss" scoped>
.pdf-native {
  &__object,
  &__embed {
    height: calc(100vh - 90px - 2rem);
    width: 100%;
  }
}
</style>