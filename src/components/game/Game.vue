<template>
  <div class="game">
    <div class="caret"></div>
    <div class="words">
      <div class="word" v-for="word, i in words" :key="i">
        <span class="letter" v-for="letter, nth in word" :key="nth">
          {{letter}}
        </span>
      </div>
    </div>
  </div>
  <input type="text" v-model="inputLog" readonly>
</template>

<script>
import randomWords from 'random-words'
export default {
  data() {
    return {
      words: [],
      inputLog: ""
    }
  },
  created() {
    this.words = this.generateWords()
  },
  mounted() {
    this.attachEvents()
  },  
  methods: {
    generateWords() {
      return randomWords(50)
    },
    attachEvents() {
      document.addEventListener('keypress', this.globalKeyPress, false)
    },
    globalKeyPress(e) {
      if (e.key) {
        return this.processValidInput(e.key)
      }      
    },
    processValidInput(key) {
      this.inputLog += key
    }
  }
}
</script>

<style lang="scss" scoped>

.game {
  position: relative;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}

.words {
  display: flex;
  flex-wrap: wrap;
  font-family: monospace;
  margin: -2px -5px;
}

.word {
  margin: 2px 5px;
  font-size: 24px;
  line-height: 1.35;
}

.letter {
  display: inline-block;
}

.caret {
  position: absolute;
  top: 0;
  left: 0;
  height: calc(24px * 1.35);
  width: 2px;
  background: red;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, to {
    opacity: 0
  }
  50% {
    opacity: 1
  }
}

</style>
