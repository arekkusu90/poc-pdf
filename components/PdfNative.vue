<template>
  <div class="pdf-native">
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


    <!-- No fallback content -->
    <embed
        src="https://eloquentjavascript.net/Eloquent_JavaScript_small.pdf"
        type="application/pdf"
        class="pdf-native__embed"
    >
  </div>
</template>

<script>
export default {
  name: "PdfNative"
}
</script>

<style lang="scss" scoped>
.pdf-native {
  &__object {
    height: 500px;
    width: 100%;
  }

  &__embed {
    height: 500px;
    width: 100%;
  }
}
</style>