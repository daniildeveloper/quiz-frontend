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

<style>
  
</style>



// WEBPACK FOOTER //
// src/components/Discount.vue