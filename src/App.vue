<template>
  <div class="wrapper">
    <registration-form></registration-form>
  </div>
</template>


<script>
import RegistrationForm from "./components/RegistrationForm.vue";

export default {
  components: {
    RegistrationForm,
  },
};
</script>

<style lang='scss'>
@import "@/assets/variables.scss";
@import url("https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap");

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.wrapper {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
