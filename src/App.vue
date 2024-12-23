<template>
  <div id="app">
    <Calculator />
  </div>
</template>

<script>
import Calculator from './components/Calculator.vue';

export default {
  components: {
    Calculator
  }
};
</script>

<style>
#app {
  text-align: center;
  margin-top: 50px;
}
</style>