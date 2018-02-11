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

<style >

</style>



// WEBPACK FOOTER //
// src/components/Assistant.vue