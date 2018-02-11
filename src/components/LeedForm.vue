<template lang="pug">
  .leed-form
    .leed-form__contacts(v-if="!thanks")
      .leed-form__header
        b-icon(icon="check-circle-outline").leed-form__icon
        p.title.leed-form__title {{ $t('Отлично. Последний шаг!') }}

      div
        .columns
          .column(:class="{ 'is-7': twoColumns }" v-if="twoColumns")

            .leed-form__text.content(v-if="info.form.text" v-html="text")

            .leed-form__discount(v-if="fixedDiscount")
              span {{ $t('Итоговая скидка') }}:
              .leed-form__discount-sep |
              .leed-form__discount-value(v-if="isIncreasingPercentDiscount") {{ fixedDiscount }} %
              .leed-form__discount-value(v-else) {{ fixedDiscount }} {{ currencySymbol }}

            .leed-form__extra.mb-2(v-if="info.form.extra")
              .leed-form__iconContainer
                b-icon(icon="gift")
              div
                b {{ $t('Дополнительно') }}:
                | {{ info.form.extra }}

          .column.is-5(:class="{ 'is-offset-3': !twoColumns, 'is-6': !twoColumns }")
            form.leed-form__fields(@submit.prevent="send" name="contacts")
              b-field(v-show="!showOtherFields" v-for="field in fields1" :key="field.key")
                b-input(:placeholder="$t('Email')" type="email" size="is-large" v-if="field.key === 'email'" v-model="email" required icon="email" :id="field.key" ref="fields")
                b-input(:placeholder="$t('Имя')" type="text" size="is-large" v-if="field.key === 'name'" v-model="name" required icon="account" :id="field.key" ref="fields")
                b-input(:placeholder="$t('Телефон')" type="tel" size="is-large" v-if="!phoneMask && field.key === 'phone'" v-model="phone" required icon="phone" :id="field.key" ref="fields")
                BMaskedInput(
                  :placeholder="$t('Телефон')"
                  type="tel" size="is-large"
                  v-if="phoneMask && field.key === 'phone'"
                  required
                  icon="phone"
                  :id="field.key"
                  ref="fields"
                  :mask="phoneMask"
                  :value="maskedPhone"
                  @input="maskedPhone = arguments[0]; phone = arguments[1]"
                )

              b-field(v-if="showOtherFields" v-for="field in fields2" :key="field.key")
                b-input(:placeholder="$t('Email')" type="email" size="is-large" v-if="field.key === 'email'" v-model="email" required icon="email" :id="field.key" ref="fields")
                b-input(:placeholder="$t('Имя')" type="text" size="is-large" v-if="field.key === 'name'" v-model="name" required icon="account" :id="field.key" ref="fields")
                b-input(:placeholder="$t('Телефон')" type="tel" size="is-large" v-if="!phoneMask && field.key === 'phone'" v-model="phone" required icon="phone" :id="field.key" ref="fields")
                BMaskedInput(
                  :placeholder="$t('Телефон')"
                  type="tel" size="is-large"
                  v-if="phoneMask && field.key === 'phone'"
                  required
                  icon="phone"
                  :id="field.key"
                  ref="fields"
                  :mask="phoneMask"
                  :value="maskedPhone"
                  @input="maskedPhone = arguments[0]; phone = arguments[1]"
                )

              button.leed-form__button.button.button_color_theme.is-medium.button_rounded(
                type="submit"
                :class="{ 'is-loading': loading }"
              )
                b-icon(icon="checkbox-marked-circle-outline")
                span {{ info.form.button || $t('Получить результаты') }}
    article.leed-form__thank(v-if="thanks")
      b-icon.leed-form__icon(icon="checkbox-multiple-marked-circle")
      p.title.leed-form__thank-title {{ $t('Спасибо') }}!
      p.subtitle.leed-form__thank-subtitle(v-html="thanksText")
</template>

<script>
  import { mapState, mapGetters } from 'vuex';
  import showdown from 'showdown';
  import BMaskedInput from './BMaskedInput';
  import _ from 'lodash';

  const converter = new showdown.Converter({
    simplifiedAutoLink: true,
    openLinksInNewWindow: true,
    simpleLineBreaks: true,
  });

  export default {
    components: {
      BMaskedInput,
    },
    mounted() {
      this.$store.commit('quiz/fixDiscount');
      this.focusInput();
    },
    data() {
      return {
        loading: false,
        thanks: false,
        name: '',
        phone: '',
        maskedPhone: '',
        email: '',
        showOtherFields: false,
      };
    },
    computed: {
      ...mapState('quiz', ['info', 'fixedDiscount']),
      ...mapGetters('quiz', [
        'getEnabledField1',
        'getEnabledField2',
        'currencySymbol',
        'isIncreasingPercentDiscount',
      ]),
      text() {
        return converter.makeHtml(this.info.form.text);
      },
      thanksText() {
        return converter.makeHtml(this.info.form.thanks);
      },
      // Поля в шаге 1
      fields1() {
        const fields1 = this.getEnabledField1;
        // Если полей в шаге 1 нет, тогда переносятся поля со второго шага
        if (!fields1.length) {
          this.showOtherFields = true;
        }
        return fields1;
      },
      // Поля в шаге 2
      fields2() {
        const fields2 = this.getEnabledField2;
        return fields2;
      },
      // Имеются ли поля для шага 2
      isStep2FieldsExist() {
        return this.fields2.length;
      },
      // Нужно ли показывать в две колонки
      twoColumns() {
        return this.info.form.text || this.fixedDiscount || this.info.form.extra;
      },
      // Маска для телефона
      phoneMask() {
        let mask = _.get(this.info, 'form.phoneMask');
        if (!mask) return null;
        mask = mask.replace('+', '\\+');
        return mask;
      },
      redirectUrl() {
        return _.get(this.info, 'form.redirect');
      },
    },
    methods: {
      // Отправляет ответ на сервер
      send() {
        if (!this.showOtherFields && !this.validateFields(1)) {
          return false;
        }
        if (this.showOtherFields && !this.validateFields(2)) {
          return false;
        }

        this.loading = true;

        this.$store.dispatch('quiz/sendAnswers', {
          name: this.name,
          email: this.email,
          phone: this.phone,
        }).then(() => {
          if (this.isStep2FieldsExist && !this.showOtherFields) {
            this.loading = false;
            this.showOtherFields = true;
            this.focusInput();
          } else if (this.redirectUrl) {
            this.$store.dispatch('quiz/finish');
            this.$store.dispatch('quiz/redirect');
          } else {
            this.$store.dispatch('quiz/finish');
            this.thanks = true;
          }
        });

        return true;
      },
      // Валидирует поля для шага step
      validateFields(step) {
        let result = true;
        if (step === 1) {
          this.fields1.forEach((field) => {
            if (field.enabled && !this[field.key]) {
              result = false;
            }
          });
        } else if (step === 2) {
          this.fields2.forEach((field) => {
            if (field.enabled && !this[field.key]) {
              result = false;
            }
          });
        }
        return result;
      },
      // Фокус на первом инпуте
      focusInput() {
        this.$nextTick(() => {
          let input;
          if (this.$refs.fields && this.$refs.fields.length) {
            if (!this.showOtherFields) {
              input = this.$refs.fields[0].$refs.input;
              // Поправка для фокуса на инпут с маской
              if (input.$el) input.$el.focus(); else input.focus();
            } else if (this.$refs.fields[this.fields1.length]) {
              input = this.$refs.fields[this.fields1.length].$refs.input;
              // Поправка для фокуса на инпут с маской
              if (input.$el) input.$el.focus(); else input.focus();
            }
          }
        });
      },
    },
  };
</script>

<style lang="stylus">
  .leed-form
    display flex
    align-items center
    justify-content center
    min-height 100vh
    padding: 40px 20px
    @media (max-height: 500px)
      padding: 20px

    &__header
      margin-bottom 3rem

    &__fields
      .field
        display block
      .icon
        i:before
          font-size: 25px

    &__icon
      color: var(--color)
      margin-bottom: 20px
      height: 45px
      width: 45px
      i
        &:before
          font-size: 60px !important

    &__iconContainer
      width: 46px
      color: var(--color)
      font-size: 29px

    &__title
      margin-bottom: 2.1rem !important
      font-weight: 500

    &__extra
      background: #f5f7f9
      padding: 20px
      text-align: left
      display: flex
      align-items: center
      color: #7e8ca8
      font-weight 500
      font-size 18px

      b
        color #2e2e54
        margin-right 0.6rem

      .leed-form__iconContainer
        display: flex
        align-items: center
        margin-right 1rem

      .icon
        height: 30px

      @media (max-width: 767px)
        flex-direction column
        .leed-form__iconContainer
          margin-right 0
          justify-content center

    &__button
      margin-top: 10px
      white-space: normal
      min-height: 50px

    &__thank-title
      margin-bottom: 2.5rem !important

    &__discount
      font-size 18px
      margin-bottom: 1rem
      margin-top -0.5rem
      font-weight 600
      display flex
      align-items center
      justify-content space-around
      max-width 350px
      margin 0 auto 1.5rem
      color #2e2e54
      &-sep
        font-weight 400
        color #eae9f1
      &-value
        color: #03a646
        font-size: 24px

    &__text
      color: #7e8ca8
      text-align justify
      padding 0 2rem
      p:not(:last-child)
        margin-bottom: 0.5rem
      ul
        text-align left
        margin-left: 2.5rem
        margin-top 0
        &:not(:last-child)
          margin-bottom: 0.5rem

    &__contacts
      width 100%
</style>



// WEBPACK FOOTER //
// src/components/LeedForm.vue