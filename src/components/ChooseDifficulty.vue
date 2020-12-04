<!-- eslint-disable prettier/prettier -->
<template>
  <div class="choose-difficulty">
    <button
      class="choose-difficulty__button"
      @click="showPrompt"
    >
      <VIcon
        :href="'settings'"
      />
    </button>
  </div>
</template>
<!-- eslint-enable -->

<script>
  import { DIFFICULTIES_DATA } from '@/utils/constants';

  export default {
    methods: {
      showPrompt() {
        const { difficultyName } = localStorage;
        let isInputValidOrClosingPopup;
        while (!isInputValidOrClosingPopup) {
          const newName = prompt(
            `
              Current game difficulty is: ${difficultyName}

              You can choose new one by typing
              any name from this list:

              ${DIFFICULTIES_DATA.map((data) => data.name)
                .filter((name) => name !== difficultyName)
                .join(', ')}
            `,
            '',
          );
          let newNameLower;
          if (newName !== null) {
            newNameLower = newName.toLowerCase();
            let errorMessage;
            if (newNameLower === difficultyName) {
              errorMessage = `
                new difficulty name should not be same
                as current one, if you want to
                close popup, press "cancel"
              `;
            } else if (
              newNameLower !== null &&
              DIFFICULTIES_DATA.every((data) => data.name !== newNameLower)
            ) {
              errorMessage = 'difficulty name is incorrect, try again';
            }
            if (errorMessage) {
              alert(errorMessage);
              continue;
            }
          }
          if (newNameLower) {
            this.$evBus.$emit('new-difficulty-name-setted', {
              newName: newNameLower,
            });
            localStorage.difficultyName = newNameLower;
          }
          isInputValidOrClosingPopup = true;
        }
      },
    },
  };
</script>

<style lang="scss">
  .choose-difficulty {
    &__button {
      position: absolute;
      top: 15px;
      right: 15px;

      > svg {
        width: 35px;
        height: 35px;
      }
    }
  }
</style>
