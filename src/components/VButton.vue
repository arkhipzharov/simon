<!-- eslint-disable prettier/prettier -->
<template>
  <button class="btn">
    <slot></slot>
  </button>
</template>
<!-- eslint-enable -->

<script>
  export default {};
</script>

<style lang="scss">
  .btn {
    padding: 15px;
    border: 1px solid $border-black;
    border-radius: 5px;
    transform: scale(1);
    transition: background-color 0.3s, color 0.3s, transform 0.3s;

    &:hover {
      color: $text-white;
      background-color: $bg-black;
      transform: scale(1.1);
      transition: background-color 0.3s, color 0.3s, transform 0.3s;
    }

    &[disabled] {
      opacity: 0.5;

      &:hover {
        color: inherit;
        background-color: inherit;
        transform: scale(1);
      }
    }
  }
</style>
