<template lang="pug">
  .answer(:class="{ 'answer_type_image': hasImages, 'answer_checked': checked }")
    div(v-if='hasImages')
      .answer__backdrop
        img(:src='imageUrl' ref="img" :class="{ answer__img_loading: !imgLoaded }")
        img.answer__backdrop-img(src="../assets/backdrop.png" v-if="!imgLoaded")
    .answer__title {{ answer.title }}

</template>

<script>
  export default {
    mounted() {
      if (this.hasImages && this.$refs.img) {
        this.$refs.img.onload = () => {
          this.imgLoaded = true;
        };
      }
    },
    props: {
      answer: Object,
      hasImages: Boolean,
      checked: Boolean,
    },
    data() {
      return {
        imgLoaded: false,
      };
    },
    computed: {
      imageUrl() {
        if (!this.answer.image) return null;
        let source = 'estorage';
        if (process.env.NODE_ENV === 'development') {
          source = 'eteststorage';
        }
        source += '-w180-h240-cscale';

        return `http://${this.answer.image.server}/${source}/${this.answer.image.url}`;
      },
    },
  };
</script>

<style>
</style>



// WEBPACK FOOTER //
// src/components/Answer.vue