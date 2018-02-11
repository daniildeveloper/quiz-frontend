<template lang="pug">
  .input-question
    b-field
      b-input(
        v-model="val"
        @keyup.native.enter="nextQuestion"
        :placeholder="question.options ? question.options.placeholder : null"
        size="is-large"
        ref="input"
      )

</template>

<script>
  import { mapActions } from 'vuex';

  export default {
    props: {
      question: Object,
    },
    mounted() {
      this.$refs.input.focus();
    },
    data() {
      return {
        val: this.$store.getters['quiz/getAnswerValue'],
      };
    },
    methods: {
      ...mapActions('quiz', ['nextQuestion']),
    },
    watch: {
      val(val) {
        this.$store.dispatch('quiz/setAnswer', val);
      },
    },
  };
</script>



// WEBPACK FOOTER //
// src/components/InputQuestion.vue