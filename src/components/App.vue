<!--
  prettier is conflicting with vue/max-attributes-per-line, and there's only 2
  ways to fix this, either configure prettier to wrap all atributes in one line
  up to 80 characters ruler

  https://github.com/prettier/eslint-plugin-prettier/issues/94#issuecomment-438781791

  or write these 2 special comments to disable prettier eslint integration for
  <template> tag

  https://github.com/vuejs/eslint-plugin-vue/issues/260#issuecomment-445791801
-->
<!-- eslint-disable prettier/prettier -->
<template>
  <div class="app">
    <div class="app__inner">
      <Game />
      <ChooseDifficulty />
    </div>
  </div>
</template>
<!-- eslint-enable -->

<script>
  import ChooseDifficulty from './ChooseDifficulty';
  import Game from './Game';

  export default {
    components: {
      ChooseDifficulty,
      Game,
    },
  };
</script>

<style lang="scss">
  .app {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    padding: 0 20px;

    &__inner {
      flex: 1 1 0;
      margin: 0 auto;
      padding: 20px 0;
    }
  }
</style>
