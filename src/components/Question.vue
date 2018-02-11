<template lang="pug">
  .content.question
    Answers(:answers='question.answers', :select='question.select' :hasOther="!!question.other")
</template>

<script>
  import Answers from './Answers';

  export default {
    components: {
      Answers,
    },
    props: {
      question: Object,
    },
  };
</script>

<style>
</style>



// WEBPACK FOOTER //
// src/components/Question.vue