<!-- eslint-disable prettier/prettier -->
<template>
  <div class="game">
    <div class="game__area game__section">
      <div class="game__row">
        <button
          v-for="(color, i) in ['blue', 'red']"
          :key="i"
          :class="`game__option-button game__option-button--${color}`"
          @click="onOptionChoose(i)"
        >
          <div
            class="game__active-option-dark-mask"
            :class="{
              active: activeOptionInd === i
            }"
          ></div>
        </button>
      </div>
      <div class="game__row">
        <button
          v-for="(color, i) in ['yellow', 'green']"
          :key="i"
          :class="`game__option-button game__option-button--${color}`"
          @click="onOptionChoose(2 + i)"
        >
          <div
            class="game__active-option-dark-mask"
            :class="{
              active: activeOptionInd === 2 + i
            }"
          ></div>
        </button>
      </div>
    </div>
    <VButton
      class="game__section"
      :disabled="isGameStarted"
      @click.native="!isGameStarted && startShowingSequence()"
    >
      Play
    </VButton>
    <span class="game__section">Round: {{ roundsNum }}</span>
    <span class="game__section">
      Time between choose option colored button clicks:
      {{ currTimeBetweenClicksInSequenceMs }} milliseconds
    </span>
    <span class="game__section">Difficulty: {{ difficultyName }}</span>
    <p v-if="gameStatus">{{ gameStatusMessages[gameStatus] }}</p>
  </div>
</template>
<!-- eslint-enable -->

<script>
  import { delay } from '@/utils/helpers';
  import { reset } from '@/mixins';
  import { DIFFICULTIES_DATA } from '@/utils/constants';

  export default {
    mixins: [
      reset(() => ({
        gameStatus: 'NOT_STARTED',
        currRoundGeneratedOptionsSequence: [],
        userChooseOptionsSequence: [],
        timerBetweenClicksInSequenceId: null,
        activeOptionInd: null,
        isGameStarted: false,
        isTimerBetweenClicksInSequenceFinished: false,
        isShowingSequence: false,
      })),
    ],
    data() {
      return {
        gameStatusMessages: {
          NOT_STARTED: 'Press "play" button to start game',
          NOT_CHOSE_ANY_OPTION: `
            Press one of 4 colored buttons to start repeating showed sequence
          `,
        },
        difficultyName: DIFFICULTIES_DATA[0].name,
      };
    },
    computed: {
      roundsNum() {
        return this.currRoundGeneratedOptionsSequence.length || 1;
      },
      currTimeBetweenClicksInSequenceMs() {
        return DIFFICULTIES_DATA.find(
          (data) => data.name === this.difficultyName,
        ).timeBetweenClicksInSequenceMs;
      },
    },
    mounted() {
      this.$evBus.$on('new-difficulty-name-setted', ({ newName }) => {
        this.difficultyName = newName;
      });
      this.setOrGetDifficultyNameWithStorage();
    },
    methods: {
      setOrGetDifficultyNameWithStorage() {
        let difficultyName = this.difficultyName;
        const difficultyNameStorage = localStorage.difficultyName;
        if (difficultyNameStorage) {
          difficultyName = difficultyNameStorage;
        } else {
          localStorage.difficultyName = difficultyName;
        }
        this.difficultyName = difficultyName;
      },
      async startShowingSequence() {
        this.isShowingSequence = true;
        if (this.isGameStarted) {
          this.gameStatus = '';
        } else {
          this.isGameStarted = true;
          this.gameStatus = 'NOT_CHOSE_ANY_OPTION';
        }
        this.possiblyStopTimerBetweenClicksInSequence();
        this.userChooseOptionsSequence = [];
        this.generateAndPushSequenceOptionInd();
        await this.activateSequenceOptionButtonsRecursive();
        this.isShowingSequence = false;
      },
      generateAndPushSequenceOptionInd() {
        const { chosenOptionInd } = this;
        const allOptionIndexes = Array.from(new Array(4).keys());
        this.currRoundGeneratedOptionsSequence.push(
          this.getRandomItemFromArr(
            chosenOptionInd
              ? allOptionIndexes.filter(
                  (optionInd) => optionInd !== chosenOptionInd,
                )
              : allOptionIndexes,
          ),
        );
      },
      async activateSequenceOptionButtonsRecursive(optionArrInd) {
        optionArrInd = optionArrInd === undefined ? 0 : ++optionArrInd;
        await this.activateOptionButtonFromSequence(
          this.currRoundGeneratedOptionsSequence[optionArrInd],
        );
        if (
          optionArrInd ===
          this.currRoundGeneratedOptionsSequence.length - 1
        ) {
          return;
        }
        this.activateSequenceOptionButtonsRecursive(optionArrInd);
      },
      async onOptionChoose(optionInd) {
        if (!this.isGameStarted || this.isShowingSequence) return;
        await this.handleOptionChooseActions(optionInd);
        const isGameFailed = this.checkGameFailurePossiblyNotify();
        if (isGameFailed) return;
        if (
          this.userChooseOptionsSequence.length <
          this.currRoundGeneratedOptionsSequence.length
        ) {
          this.notRepeatedAllSequenceTakeCareOfTimers();
        } else {
          this.startShowingSequence();
        }
      },
      async handleOptionChooseActions(optionInd) {
        await this.activateOptionButtonFromSequence(optionInd);
        this.userChooseOptionsSequence.push(optionInd);
      },
      notRepeatedAllSequenceTakeCareOfTimers() {
        this.possiblyStopTimerBetweenClicksInSequence();
        this.timerBetweenClicksInSequenceId = setTimeout(() => {
          this.isTimerBetweenClicksInSequenceFinished = true;
        }, this.currTimeBetweenClicksInSequenceMs);
      },
      checkGameFailurePossiblyNotify() {
        const failureMessage = this.possiblyGetGameFailureMessage();
        if (failureMessage) {
          this.possiblyStopTimerBetweenClicksInSequence();
          const { roundsNum } = this;
          alert(
            `You are failed after ${roundsNum} ${this.adoptWordEndingsToNum(
              roundsNum,
              ['round', 'rounds', 'rounds'],
            )} because ${failureMessage}`,
          );
          this.reset();
          return true;
        }
      },
      possiblyGetGameFailureMessage() {
        let failureMessage;
        const { isTimerBetweenClicksInSequenceFinished } = this;
        if (isTimerBetweenClicksInSequenceFinished) {
          failureMessage = `${this.currTimeBetweenClicksInSequenceMs} milliseconds passed after button click in process of repeating showed sequence`;
        } else if (
          this.userChooseOptionsSequence.some((userOptionInd, i) => {
            return userOptionInd !== this.currRoundGeneratedOptionsSequence[i];
          })
        ) {
          failureMessage = 'chose wrong colored button';
        }
        return failureMessage;
      },
      possiblyStopTimerBetweenClicksInSequence() {
        if (this.timerBetweenClicksInSequenceId) {
          clearTimeout(this.timerBetweenClicksInSequenceId);
          this.timerBetweenClicksInSequenceId = null;
        }
      },
      async activateOptionButtonFromSequence(optionInd) {
        this.playOptionSound(optionInd);
        this.activeOptionInd = optionInd;
        await delay(700);
        this.activeOptionInd = null;
        this.chosenOptionInd = optionInd;
      },
      playOptionSound(optionInd) {
        const audio = new Audio(
          // eslint-disable-next-line global-require
          require(`@/assets/audio/game-options/${optionInd}.mp3`),
        );
        audio.volume = 0.28;
        audio.play();
      },
      getRandomItemFromArr(arr) {
        return Math.floor(Math.random() * arr.length);
      },
      // ru and en
      //
      // https://realadmin.ru/coding/sklonenie-na-javascript.html (ru)
      adoptWordEndingsToNum(number, wordsWithAllEndings) {
        number = Math.abs(number) % 100;
        const n1 = number % 10;
        let index;
        if (number > 10 && number < 20) {
          index = 2;
        } else if (n1 > 1 && n1 < 5) {
          index = 1;
        } else if (n1 === 1) {
          index = 0;
        } else {
          index = 2;
        }
        return wordsWithAllEndings[index];
      },
    },
  };
</script>

<style lang="scss">
  .game {
    display: flex;
    flex-direction: column;
    align-items: center;
    opacity: 0.7;

    &__section {
      margin-bottom: 30px;
    }

    &__area {
      overflow: hidden;
      border-radius: 9999px;
    }

    &__row {
      display: flex;
    }

    &__option-button {
      position: relative;
      display: block;
      width: 100px;
      height: 100px;
      transition: opacity 0.15s;

      &--blue {
        background-color: #0b64dc;
      }

      &--red {
        background-color: #f57f6c;
      }

      &--yellow {
        background-color: #ffc107;
      }

      &--green {
        background-color: #00b300;
      }

      &:hover {
        opacity: 0.7;
        transition: opacity 0.15s;
      }
    }

    &__active-option-dark-mask {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: $bg-black;
      opacity: 0;
      transition: opacity 0.1s;

      &.active {
        opacity: 0.6;
        transition: opacity 0.1s;
      }
    }

    &__row-break {
      width: 100%;
    }
  }
</style>
