<template>
  <div :class="[
    `word`,
    {
      'word--current': isCurrent
    }
  ]">
    <Letter 
      v-for="letter, i in word"
      :letter="letter"
      :letterIndex="i"
      :wordIndex="wordIndex"
      :wrong="false"
      :isCorrect="isLetterInIndexCorrect(i)"
      :isWrong="!isLetterInIndexCorrect(i) && isLetterInUserInput(i)"
      :key="i"
    />
  </div>
</template>

<script>
import Letter from './Letter'

export default {
  props: {
    word: {
      type: String
    },
    wordIndex: {
      type: Number
    },
    isCurrent: {
      type: Boolean
    },
    userInputForWord: {
      type: String
    }
  },
  methods: {
    isLetterInIndexCorrect(i) {
      return this.word && this.userInputForWord &&
        this.word[i] && 
        this.userInputForWord[i] && 
        this.userInputForWord[i] === this.word[i]
    },
    isLetterInUserInput(i) {
      return this.userInputForWord && i <= this.userInputForWord.length - 1
    }
  },
  components: {
    Letter  
  }
}
</script>

<style lang="scss" scoped>
.word {
  margin: 2px 5px;
  font-size: 24px;
  line-height: 1.35;
}

.word--current {
  background-color: rgba(white, 0.15)
}
</style>