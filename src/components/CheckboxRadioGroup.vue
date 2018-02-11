<template lang="pug">
  .checkbox__radio
    b-checkbox-group(v-if="select === 'many'" v-model="value")
      slot
    b-radio-group(v-if="select === 'one'" v-model="value")
      slot
</template>

<script>
  export default {
    props: ['select', 'value'],
  };
</script>