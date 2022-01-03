<template>
  <div class="input-container">
    <input
      class="input"
      :type="type"
      :value="modelValue"
      @input="updateInput"
      :maxlength="maxlength"
    />
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
  name: "labeled-input",
  props: {
    modelValue: [String, Number, Array],
    maxlength: Number,
    required: Boolean,
    type: String,
  },
  methods: {
    updateInput(e) {
      this.$emit("update:modelValue", e.target.value);
      this.$emit("input");
    },
  },
};
</script>
<style lang="scss">
@import "@/assets/variables";
.required {
  color: $red;
}

.input::-webkit-outer-spin-button,
.input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.input[type="number"] {
  -moz-appearance: textfield;
}
</style>