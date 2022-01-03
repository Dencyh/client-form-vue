<template>
  <div class="input-container">
    <input class="input" type="text" :value="modelValue" @input="handleInput" />
    <label class="label">
      <slot></slot>
      <slot name="required">
        <span class="required" v-if="required">*</span>
      </slot>
    </label>
  </div>
</template>
<script>
export default {
  name: "phone-input",
  data() {
    return {
      formatRule: "formatRule",
    };
  },
  props: {
    required: Boolean,
    modelValue: [String, Number],
  },
  methods: {
    handleInput(e) {
      let formated = "+7 (";
      let userInput = e.target.value;
      userInput = ("" + userInput)
        .replace(/\D/g, "")
        .replace(/^7/, "")
        .substr(0, this.formatRule.length);

      for (let i = 0; i < userInput.length; i++) {
        if (i == 3) {
          formated = formated + ") ";
        } else if (i == 6) {
          formated = formated + "-";
        } else if (i == 8) {
          formated = formated + "-";
        }
        formated = formated + userInput[i];
      }
      e.target.value = formated;
      this.$emit("update:modelValue", e.target.value);
    },
  },
};
</script>
<style lang='scss'>
</style>