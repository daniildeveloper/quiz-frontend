<template>
    <div class="control"
        :class="[iconPosition, {
            'is-expanded': expanded,
            'is-loading': loading,
            'is-clearfix': !hasMessage
        }]">
        <masked-input
            ref="input"
            class="input"
            :mask="mask"
            :class="[statusType, size]"
            :type="newType"
            :autocomplete="newAutocomplete"
            :maxlength="maxlength"
            :value="newValue"
            v-bind="$attrs"
            @input="onInput"
            @blur="$emit('blur', $event) && checkHtml5Validity()"
            @focus="$emit('focus', $event)"/>

        <b-icon v-if="icon"
            class="is-left"
            :icon="icon"
            :pack="iconPack"
            :size="size">
        </b-icon>

        <b-icon v-if="!loading && (passwordReveal || statusType)"
            class="is-right"
            :class="{ 'is-clickable': passwordReveal }"
            :icon="passwordReveal ? passwordVisibleIcon : statusTypeIcon"
            :size="size"
            :type="!passwordReveal ? statusType : 'is-primary'"
            both
            @click.native="togglePasswordVisibility">
        </b-icon>

        <small class="help counter" v-if="maxlength && hasCounter">{{ valueLength }} / {{ maxlength }}</small>
    </div>
</template>

<script>
  /**
   * Компонент объединяет Buefy input и vue-masked-input
   */
  import MaskedInput from 'vue-masked-input';
  import Buefy from 'buefy';

  export default {
    extends: Buefy.Input,
    components: {
      MaskedInput,
    },
    props: {
      mask: String,
    },
    data() {
      return {
        rawVal: null,
      };
    },
    methods: {
      onInput(event, raw) {
        // Возвращаем чистое значение только когда маска полностью заполнена
        if (raw.match(/_/g)) {
          this.rawVal = null;
        } else if (raw.match(/\d/g)) {
          // В качестве чистого значения возвращаем "+" и числа
          const matched = event.match(/\d|\+/g);
          this.rawVal = matched ? matched.join('') : '';
        } else {
          this.rawVal = null;
        }
        this.$nextTick(() => { this.newValue = event; });
      },
    },
    watch: {
      newValue(value) {
        this.$emit('input', value, this.rawVal);
        // eslint-disable-next-line
        !this.isValid && this.checkHtml5Validity();
      },
    },
  };
</script>



// WEBPACK FOOTER //
// src/components/BMaskedInput.vue