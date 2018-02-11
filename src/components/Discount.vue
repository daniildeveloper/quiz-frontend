<template lang="pug">
  .discount(:class="{'is-increasing': isIncreasingDiscount || isIncreasingPercentDiscount}")
    .discount__label(v-if="!isIncreasingDiscount && !isIncreasingPercentDiscount") {{ $t('Ваша скидка') }}:&nbsp;
    .discount__price(v-if="!isIncreasingDiscount && !isIncreasingPercentDiscount") <b>{{ formattedDiscount }}</b> {{ currencySymbol || '₽'}}
    .increasingDiscount(v-if="isIncreasingDiscount")
      .increasingDiscount-step
        .increasingDiscount-stepTitle {{ $t('За каждый ответ') }}
        .increasingDiscount-stepValue + {{ getIncreasingDiscountStep }} {{ currencySymbol || '₽'}}
      .increasingDiscount-current
        .increasingDiscount-currentTitle {{ $t('Скидка') }}:
        .increasingDiscount-currentValue
          b {{ formattedDiscount }}
          | {{ currencySymbol || '₽'}}
        i.mdi.mdi-chevron-double-up
    .increasingDiscount(v-if="isIncreasingPercentDiscount")
      .increasingDiscount-step
        .increasingDiscount-stepTitle {{ $t('За каждый ответ') }}
        .increasingDiscount-stepValue + {{ getIncreasingPercentDiscountStep }} %
      .increasingDiscount-current
        .increasingDiscount-currentTitle {{ $t('Скидка') }}:
        .increasingDiscount-currentValue
          b {{ formattedDiscount }}
          | %
        i.mdi.mdi-chevron-double-up
</template>

<script>
  import { mapGetters } from 'vuex';

  export default {
    props: {
      countQuestions: Number,
    },
    data() {
      return {
        interval: null,
      };
    },
    computed: {
      ...mapGetters('quiz', [
        'formattedDiscount',
        'isIncreasingDiscount',
        'isIncreasingPercentDiscount',
        'getIncreasingDiscountStep',
        'currencySymbol',
        'getIncreasingPercentDiscountStep',
      ]),
    },
  };
</script>

<style lang="stylus">
  .discount
    background #fff
    padding 0.8rem 0.5rem
    display: flex
    text-align: center
    justify-content: space-around
    align-items: center
    flex-wrap wrap

    &__label
      font-size: 14px
      color: #8695b4

    &__price
      font-size 24px
      font-weight 500
      flex-basis: 95px
      white-space: nowrap
      text-align: right

  .increasingDiscount
    display: flex
    width: 100%
    justify-content: space-between

    &-step
      text-align: left
    &-stepTitle
      white-space: nowrap
      font-size: 14px
      color: #8695b4
    &-stepValue
      color: #24b15e
      font-size: 14px
      font-weight: bold
      line-height: 28px
    &-current
      display: grid
      grid-template-columns: auto 1fr
      text-align: left
      padding: 0 25px 0 0
      &Title
        grid-column: span 2
        font-size: 14px
        color: #8695b4
    &-currentValue
      color: #2e2e54
      font-size: 18px
    i
      position: relative
      top: -6px
      color: #24b15e
</style>



// WEBPACK FOOTER //
// src/components/Discount.vue