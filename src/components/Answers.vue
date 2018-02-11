<template lang="pug">
  div
    template(v-if="!hasImages")
      .answers.answers_type_text(ref="answers")
        .answers__group(v-model="val")
          .answers__textVariant(
            ref="answer"
            v-for="answer in answers",
            :key="answer.id"
            :class="{ 'answers__textVariant_selected': isChecked(answer.title) }"
          )
            b-radio(:native-value="answer.title" v-if="select === 'one'" v-model="val")
              Answer(:answer="answer", :hasImages="hasImages", :checked="isChecked(answer.title)")
            b-checkbox(:native-value="answer.title" v-if="select === 'many'" v-model="val")
              Answer(:answer="answer", :hasImages="hasImages", :checked="isChecked(answer.title)")

          // Поле "свой вариант"
          .answers__textVariant.answers__other(v-if="hasOther")
            b-radio(v-if="select === 'one'" :value="!!other" @input="checkOtherValue" :native-value="true")
            b-checkbox(v-if="select === 'many'" :value="!!other" @input="checkOtherValue")
            b-field
              b-input(v-model="other" placeholder="Другое..." @keyup.native.enter="nextQuestion")

    template(v-if="hasImages")
      .answers.answers_has-images(ref="answers")
        .answers__group
          swiper(:options="swiperOption")
            swiper-slide(ref="answer" v-for="answer in answers", :key="answer.id")
              .answers__answerContainer.field
                b-checkbox(:native-value="answer.title" v-if="select === 'many'" v-model="val")
                  Answer(:answer="answer", :hasImages="hasImages", :checked="isChecked(answer.title)")
                b-radio(:nativeValue='answer.title' v-if="select === 'one'" v-model="val")
                  Answer(:answer="answer", :hasImages="hasImages", :checked="isChecked(answer.title)")
            .swiper-button-prev(slot="button-prev")
              b-icon(icon="chevron-left")
            .swiper-button-next(slot="button-next")
              b-icon(icon="chevron-right")

</template>

<script>
  import Answer from './Answer';
  import CheckboxRadioGroup from './CheckboxRadioGroup';
  import _ from 'lodash';
  import { mapActions } from 'vuex';

  export default {
    components: {
      Answer, CheckboxRadioGroup,
    },
    data() {
      const val = this.$store.getters['quiz/getAnswerValue'];
      return {
        val,
        other: this.getOtherValue(val),
        swiperOption: {
          navigation: {
            prevEl: '.swiper-button-prev',
            nextEl: '.swiper-button-next',
          },
          keyboard: true,
          mousewheel: true,
          scrollbar: '.swiper-scrollbar',
          scrollbarHide: false,
          slidesPerView: 'auto',
          grabCursor: true,
          centredSlides: true,
          breakpoints: {
            1440: {
              slidesPerView: 4,
              slidesPerGroup: 4,
            },
            1024: {
              slidesPerView: 4,
              slidesPerGroup: 4,
              spaceBetween: 40,
            },
            768: {
              slidesPerView: 3,
              slidesPerGroup: 3,
              spaceBetween: 30,
            },
            640: {
              slidesPerView: 2,
              slidesPerGroup: 2,
              spaceBetween: 20,
            },
            320: {
              slidesPerView: 2,
              slidesPerGroup: 2,
              spaceBetween: 10,
            },
          },
        },
      };
    },
    props: {
      answers: Array,
      select: String,
      hasOther: Boolean,
    },
    computed: {
      // Возвращает индекс варианта "другое"
      otherIndex() {
        return this.getOtherIndex(this.val);
      },
      // Возвращает значение варианта "другое"
      otherValue() {
        return this.getOtherValue(this.val);
      },
      // Возвращает, есть ли изображения в ответах
      hasImages() {
        let hasImages = false;
        this.answers.forEach((a) => {
          if (a.image) {
            hasImages = true;
          }
        });
        return hasImages;
      },
    },
    methods: {
      ...mapActions('quiz', ['nextQuestion']),
      // Возвращает индекс варианта "другое"
      getOtherIndex(val) {
        let otherIndex;
        // Ищем значение поля "другое" - оно такое, которое не содержится в массиве ответов
        const diff = _.difference(val, _.map(this.answers, 'title'));
        if (diff.length) {
          otherIndex = val.indexOf(diff[0]);
        } else {
          otherIndex = null;
        }
        return otherIndex;
      },
      // Возвращает значение варианта "другое"
      getOtherValue(val) {
        if (this.select === 'one') {
          return _.includes(_.map(this.answers, 'title'), val) ? null : val;
        }
        if (!val) return null;
        const otherIndex = this.getOtherIndex(val);
        return val && otherIndex >= 0 ? val[otherIndex] : null;
      },
      // Устанавливает значением ответа значение из поля "другое"
      setOtherVal(val) {
        if (this.select === 'one') {
          this.val = val;
        } else if (this.otherIndex !== null) {
          // Если значение поля "другое" уже есть, то заменяем его
          if (!val) {
            // Удаляем если пусто
            this.val.splice(this.otherIndex, 1);
          } else this.val[this.otherIndex] = val;
        } else {
          // Если нет, то добавляем в массив значений новое
          this.val.push(val);
        }
      },
      // Проверяет, выбрано ли значение ответа
      isChecked(answerValue) {
        let isChecked = false;
        if (this.val) {
          if (this.select === 'one') {
            isChecked = this.val === answerValue;
          } else {
            isChecked = this.val.indexOf(answerValue) >= 0;
          }
        }
        return isChecked;
      },
      // Возвращает, вылазят ли изображения за пределы блока
      getIsWider() {
        let answersWidth = 0;
        this.$refs.answer.forEach((a) => {
          answersWidth += a.clientWidth;
        });
        return this.$refs.answers.clientWidth <= answersWidth;
      },
      // Является ли значение другим, не из вариантов ответов
      isOther(val) {
        let isOther = true;
        if (this.select === 'one') {
          isOther = _.indexOf(_.map(this.answers, 'title'), val) >= 0;
        } else {
          isOther = _.difference(this.val, _.map(this.answers, 'title'));
        }
        return isOther;
      },
      // Событие при изменении чекбокса
      checkOtherValue(v) {
        if (!v) {
          this.other = '';
        }
      },
    },
    watch: {
      other(v) {
        // Устанавливаем значением ответа значение из поля "другое"
        if (v !== null) this.setOtherVal(v);
      },
      val(val) {
        // Убираем значение поля "другое" если нажат вариант ответа
        if (val && this.other && this.select === 'one' && val !== this.other) {
          this.other = null;
        }
        this.$store.dispatch('quiz/setAnswer', val);
      },
    },
  };
</script>

<style >

</style>



// WEBPACK FOOTER //
// src/components/Answers.vue