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
    
    <Letter 
      v-for="letter, i in typedExtras"
      :letter="letter"
      :letterIndex="i + (word.length )"
      :wordIndex="wordIndex"
      :wrong="false"
      :isCorrect="isLetterInIndexCorrect(i + ( word.length ))"
      :isWrong="!isLetterInIndexCorrect(i + ( word.length )) && isLetterInUserInput(i + word.length )"
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
      return this.word && 
        (i <= this.word.length - 1) &&
        this.userInputForWord &&
        this.word[i] && 
        this.userInputForWord[i] && 
        this.userInputForWord[i] === this.word[i]
    },
    isLetterInUserInput(i) {
      return this.userInputForWord && i <= this.userInputForWord.length - 1
    }
  },
  computed: {
    typedExtras() {
      return this.userInputForWord?.slice(this.word.length, this.userInputForWord.length) || ""
    }
  },
  components: {
    Letter  
  }
}
</script>

<style lang="scss" scoped>
.word {
  margin: var(--game-word-gap-v) var(--game-word-gap-h);
  font-size: var(--game-font-size);
  line-height: var(--game-line-height);
}

.word--current {
  background-color: rgba(white, 0.05)
}
</style>