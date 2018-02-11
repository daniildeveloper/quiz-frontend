<template lang="pug">
  .content.question
    Answers(:answers='question.answers', :select='question.select' :hasOther="!!question.other")
</template>

<script>
  import Answers from '@/components/Answers';

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
  .question {

  }

  .question__title {
    font-size: 30px;
    text-align: left;
    margin-bottom: 0.8rem !important;
    margin-left: -2px;
    font-weight: 300;
  }

  @media (max-width: 767px) {
    .question__title {
      font-size: 24px;
    }
  }
</style>



// WEBPACK FOOTER //
// src/components/Question.vue