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

<style lang="less">
  @import '../variables.less';

  .quiz {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    width: 100vw;
    min-height: 100vh;
    @media (max-width: @screen-xs-max) {
      padding-bottom: 90px;
      flex-direction: column;
    }
  }

  .quiz__questions {
    position: relative;
    padding: 20px;
    flex: 4 1 69%;
    width: 100%;
    -webkit-overflow-scrolling:touch;
    overflow: hidden;
    @media (max-width: @screen-xs-max) {
      flex-basis: auto;
    }
    @media (min-width: 768px) {
      padding: 30px;
    }
  }

  .quiz__sidebar {
    position: relative;
    flex: 1 1 246px;
    background: #f7f7f7;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    @media (max-width: @screen-xs-max) {
      flex-basis: auto;
    }
  }

  .quiz__made {
    font-size: 14px;
  }

  .quiz__title {
    font-size: 16px;
    text-align: left;
    margin-bottom: 1.5rem;
    display: flex;
    align-items: center;
    .icon {
      color: var(--color);
      margin-right: 10px;
    }
  }

  .quiz__buttons {
    padding: 0;
    padding-top: 1rem;
    @media (max-width: @screen-xs-max) {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      padding: 20px;
      background: #fff;
      z-index: 100;
    }
    @media (min-width: @screen-md-min) {
      padding: 20px;
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
    }
  }

  .quiz__button {
    font-size: 18px !important;
    height: 48px;
    z-index: 100;
    padding-top: 0;
    padding-bottom: 0;

    &_prev {
      text-decoration: none !important;
      color: #888888 !important;
      font-size: 18px !important;
      font-weight: 300;
      background: transparent !important;
      box-shadow: 1.7px 5.8px 21px 0 rgba(0, 0, 0, 0.08) !important;
    }
    &_next {
      color: var(--color-text);
      background-color: var(--color) !important;
    }
  }

  // Фикс к скроллу на мобильной версии без консультанта
  .quiz__scrollfix {
    @media (max-width: 767px) {
      height: 90px;
      flex-basis: 100%;
    }
  }

  .quiz__question {
    position: relative;
    @media (min-width: @screen-xs-max) {
      height: 248px;
    }
    @media (min-width: @screen-sm-min) and (max-width: @screen-sm-max) {
      height: 436px;
    }
    @media (min-width: @screen-lg-min) {
      height: 300px;
      padding: 5px 0 0;
    }
    @media (max-width: @screen-sm-max) {
      padding-top: 20px;
    }
  }

  @media (max-width: @screen-xs-max) {
    .quiz {
      padding-top: 20px;
    }
  }

  .progress {
    display: block;
    border-radius: 3px;
    height: 6px;
    line-height: 21px;
    font-size: 15px;
    font-family: sans-serif;
    text-align: center;
    -webkit-appearance: none;
    appearance: none;
    box-sizing: border-box;
  }

  .progress::-webkit-progress-bar {
    //
  }
  .progress[value]::-webkit-progress-value {
    background: var(--color);
    background-image:
        -webkit-linear-gradient(-45deg, transparent 33%, var(--color) 33%, rgba(0,0, 0, .1) 66%, transparent 66%),
        -webkit-linear-gradient(top,rgba(255, 255, 255, .25), var(--color-lighten));

    border-radius: 2px;
    background-size: 35px 20px, 100% 100%, 100% 100%;
    box-shadow: 0.6px 1.9px 7px 0 rgba(61, 59, 238, 0.46);
  }
  .progress::-moz-progress-bar {
    // background: #f1c6e1;
    background: var(--color-lighten);
  }

  .progress__label {
    // color: #ca2f94;
    font-size: 12px;
    text-align: left;
    font-weight: 500;
    color: #7e8ca8;
    margin-bottom: 8px;
    span {
      font-size: 14px;
      color: var(--color);
      font-weight: 700;
      margin-left: 10px;
    }
  }

  .quiz__discount {
    margin: 1rem 1.2rem 0;
    &.is-increasing {
      margin: 0.1rem 0.1rem 0;
    }
    @media (max-width: @screen-xs-max) {
      display: none !important;
    }
  }

  .quiz__discountTop {
    border: 2px solid #f7f7f7;
    margin-top: -1rem;
    margin-bottom: 1rem;
    @media (min-width: @screen-sm-min) {
      display: none !important;
    }
  }

  .quiz__made-in {
    font-size: 12px;
    color: #7e8ca8;
    padding: 10px 0;
    a {
      font-weight: 900;
      color: #7e8ca8;
    }

    &_placement_bottom {
      flex: 0 0 100%;
      @media (min-width: @screen-sm-min) {
        position: absolute;
        bottom: 5px;
        left: 50%;
        margin-left: -80px;
      }
      @media (max-width: @screen-xs-max) {
        margin-top: 60px;
      }
    }
  }
</style>



// WEBPACK FOOTER //
// src/components/Quiz.vue