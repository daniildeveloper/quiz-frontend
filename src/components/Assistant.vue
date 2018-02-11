<template lang="pug">
  .assistant
    .media(v-if='question.text')
      .media-left(v-if='assistant.name')
        figure.image.assistant__avatar
          img(:src='imageUrl', v-if='imageUrl')
          img(:src='avatar', v-if='!imageUrl')
      .media-content
        div(v-if='assistant.name')
          p.title.assistant__title 
            | {{ assistant.name }}
          p.subtitle.assistant__subtitle
            span.assistant__status
            | {{ assistant.title }}
        // <p class="subtitle is-6">Вопрос {{ activeQuestion + 1 }} из {{ countQuestions }}</p>
    hr(v-if='question.text')
    .content.has-text-left(v-if='question.text' v-html="text")
  
</template>

<script>
  import { mapState, mapGetters } from 'vuex';
  import avatar from '../assets/artem.jpg';
  import showdown from 'showdown';

  const converter = new showdown.Converter({
    simplifiedAutoLink: true,
    openLinksInNewWindow: true,
    simpleLineBreaks: true,
  });

  export default {
    computed: {
      ...mapState('quiz', {
        assistant: state => state.info.assistant,
        activeQuestion: state => state.activeQuestion,
      }),
      ...mapGetters('quiz', {
        question: 'getQuestion',
        countQuestions: 'countQuestions',
      }),
      imageUrl() {
        if (!this.assistant.avatar) {
          return null;
        }

        let source = 'estorage';
        if (process.env.NODE_ENV === 'development') {
          source = 'eteststorage';
        }
        source += '-s80-cscale';
        return `http://${this.assistant.avatar.server}/${source}/${this.assistant.avatar.url}`;
      },
      text() {
        return converter.makeHtml(this.question.text);
      },
    },
    data() {
      return {
        avatar,
      };
    },
  };
</script>

<style lang="less">
  .assistant {
    font-size: 0.9rem;
    width: 100%;
    padding: 23px 26px;
    @media (max-width: 767px) {
      padding: 30px 26px;
      min-height: 300px;
    }
  }

  .assistant__title {
    font-size: 16px;
    font-weight: 700;
    margin-top: 1.1rem;
    margin-bottom: 0;
  }

  .assistant__status {
    display: inline-block;
    width: 5px;
    height: 5px;
    border-radius: 2.5px;
    background-color: #03a646;
    margin-right: 7px;
    margin-bottom: 1px;
  }

  .assistant__subtitle {
    font-size: 12px;
    color: #969696;
    margin-bottom: 0;
    margin-top: -1.4rem;
  }

  .assistant__avatar img {
    width: 70px;
    height: 70px;
    border-radius: 35px;
    box-shadow: 5px 4px 17px 0px rgba(0, 0, 0, 0.16);
  }
</style>



// WEBPACK FOOTER //
// src/components/Assistant.vue