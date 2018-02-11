<template lang="pug">
  div(:style="styles")
    .quiz(
      v-if="questionNumber !== getLastStep"
      :class="{ 'quiz_sidebar_on': showRightSidebar }"
    )
      .quiz__questions
        div
          h3.quiz__title
            b-icon(icon="checkbox-multiple-marked-outline")
            |  {{ info.title }}

          Discount.quiz__discountTop(
            v-if="info.marketing && info.marketing.discount && (info.marketing.discount.value || info.marketing.discount.percent)"
            :countQuestions="questions.length"
          )

          p.title.question__title {{ getQuestion.title }}

          .progress__label {{ $t('Готово') }}:
            span {{ Math.round(getPassedPercent * 100) }}%
          progress.progress.is-danger(:value='getPassedPercent * 100', max='100')

          .quiz__question
            div(v-for='(question, i) in questions', v-if='i == activeQuestion')
              Question(:question='question', :key='i' v-if="!question.type || question.type === 'answers'")
              InputQuestion(:question='question', :key='i' v-if="question.type === 'input'")

          .column.is-12.is-clearfix.quiz__buttons
            button.button.is-large.is-pulled-left.is-link.quiz__button.quiz__button_prev(@click='prevQuestion', :disabled='questionNumber === 1')
              b-icon(icon='arrow-left' size="is-small")
              span {{ $t('Назад') }}

            button.button.button_color_theme.is-pulled-right.quiz__button.quiz__button_next(
              @click='nextQuestion',
              :disabled='!getIsFilled',
              v-if='questionNumber !== countQuestions'
            )
              b-icon(icon="checkbox-marked-circle-outline")
              span {{ $t('Далее') }}
            button.button.button_color_theme.is-pulled-right.quiz__button.quiz__button_next(
              @click='showLeedForm',
              v-if='questionNumber === countQuestions'
            ) {{ $t('Последний шаг') }}

      .quiz__sidebar(v-if="showRightSidebar")
        div
          Discount.quiz__discount(
            v-if="discount"
            :countQuestions="questions.length"
          )
          assistant(v-if="getQuestion.text")
        .quiz__made-in
          | {{ $t('Сделано с помощью') }} 
          a(href="https://marquiz.ru?utm_source=widget" target="_blank") Marquiz

      .quiz__made-in.quiz__made-in_placement_bottom(v-if="!showRightSidebar") {{ $t('Сделано с помощью') }} 
        a(href="https://marquiz.ru?utm_source=widget" target="_blank") Marquiz

    div(v-if='questionNumber === getLastStep')
      .container
        LeedForm
</template>

<script>
  import Assistant from './Assistant';
  import { mapState, mapActions, mapGetters } from 'vuex';
  import Question from './Question';
  import LeedForm from './LeedForm';
  import Discount from './Discount';
  import InputQuestion from './InputQuestion';
  import _ from 'lodash';

  export default {
    components: {
      Assistant, Question, LeedForm, Discount, InputQuestion,
    },
    computed: {
      ...mapState('quiz', [
        'questions', 'activeQuestion', 'info',
      ]),
      ...mapGetters('quiz', ['questionNumber', 'countQuestions', 'getPassedPercent', 'getAnswerValue', 'getIsFilled',
        'getLastStep', 'getQuestion', 'colors']),
      styles() {
        return {
          '--color': this.colors.main,
          '--color-lighten': this.colors.lighten,
          '--color-lighten2': this.colors.lighten2,
          '--color-alpha': this.colors.alpha,
          '--color-text': this.colors.text,
          '--color-text2': this.colors.text2,
        };
      },
      discount() {
        return _.get(this.info, 'marketing.discount.value') || _.get(this.info, 'marketing.discount.percent');
      },
      // Показывать ли правый сайдбар
      showRightSidebar() {
        return this.discount || (this.getQuestion && this.getQuestion.text);
      },
    },
    methods: {
      ...mapActions('quiz', ['prevQuestion', 'nextQuestion']),
    },
  };
</script>

<style>
  
</style>



// WEBPACK FOOTER //
// src/components/Quiz.vue