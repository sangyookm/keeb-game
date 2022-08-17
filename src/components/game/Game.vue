<template>
  <div class="game">
    <div class="caret" :style="caretPos"></div>
    <div class="words" ref="wordsCont">
      <Word 
        v-for="word, i in words"
        :word="word"
        :userInputForWord="getUserInputWordByWordIndex(i)"
        :wordIndex="i"
        :key="i"
        :isCurrent="i === currentWordIndex"
        ref="word"
      />
      <!-- <div class="word" v-for="word, i in words" :key="i" ref="word">
        <span class="letter" v-for="letter, nth in word" :key="nth">
          {{letter}}
        </span>
      </div> -->
    </div>
    <div>
      <div>
        {{words[currentWordIndex]}}
        <strong>
          [{{currentWordIndex}}]
        </strong>
      </div>
      <div class="debug-user-input">
        <div v-for="w, i in inputtedWords" :key="i">
          {{w}}
        </div>
      </div>
    </div>
    <textarea v-model="currentInput" readonly />
  </div>
</template>

<script>
import randomWords from 'random-words'
import Word from './Word'
export default {
  data() {
    return {
      words: [],
      inputtedWords: [],
      currentWordIndex: 0,
      currentInput: "",
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
      document.addEventListener('keydown', this.globalKeyDown, false)
    },
    globalKeyPress(e) {
      const {code, key, which} = e
      const isSpace = code === "Space" || which === 32

      if (isSpace) {
        return this.processSpace()
      }

      if (key) {
        return this.processValidInput(e.key)
      } 
        
    },
    globalKeyDown(e) {
      const {code, key, which} = e
      const isBackspace = key === "Backspace" || code === "Backspace" || which === 8
      if (isBackspace) {
        e.preventDefault()
        console.log("isBackspace")
        return this.processBackspace()
      }
    },
    processValidInput(key) {
      // console.log("key", key)
      this.currentInput += key
      
      const w = this.inputtedWords[this.currentWordIndex] || ''
      this.inputtedWords.splice(
        this.currentWordIndex,
        1,
        `${w}${key}`.trim()
      )
    },
    getUserInputWordByWordIndex(wordIndex) {
      return this.inputtedWords[wordIndex] || null
    },
    processSpace() {
      const currentWord = this.words[this.currentWordIndex]
      const curLen = currentWord.length
      const curInpLen = this.currentInput.length

      if (curLen > curInpLen) return

      this.currentWordIndex += 1
      this.currentInput = ""
    },
    processBackspace() {
      if (this.isStart) return
      if (this.currentInput.length <= 0 && this.currentWordIndex >= 1) {
        this.currentWordIndex -= 1
        this.currentInput = this.inputtedWords[this.currentWordIndex]
      } else {
        this.currentInput = this.currentInput.slice(0, this.currentInput.length - 1)
        // const w = this.inputtedWords[this.currentWordIndex] || ''
        this.inputtedWords.splice(
          this.currentWordIndex,
          1,
          this.currentInput
        )
      }
    }
  },
  computed: {
    isStart() {
      return this.currentInput.length <= 0 && this.currentWordIndex <= 0
    },
    caretPos() {
      try {
        console.log("refs.word", this.$refs.word)
        const wordEl = this.$refs.word?.[this.currentWordIndex]?.$el
        console.log("wordEl", wordEl)
        const letterRect = wordEl?.querySelector('.letter')?.getBoundingClientRect()
        const letterWidth = letterRect?.width || 0
        // console.log("letter", wordEl?.querySelector('.letter'))
        const currentLetterIndex = this.currentInput.length
    
        let top = wordEl ? wordEl.offsetTop : 0
        let left = wordEl ? wordEl.offsetLeft + (currentLetterIndex * letterWidth): 0
        
        // const contEl = this.$refs.wordsCont
        console.log(top, left, letterRect)
        return {
          top: `${top}px`,
          left: `${left}px`,
        }
      } catch {
        return {}
      }
    }
  },
  components: {
    Word
  }
}
</script>

<style lang="scss" scoped>

.game {
  position: relative;
  width: 100%;
  max-width: 1200px;
  margin: auto;
}

.words {
  display: flex;
  flex-wrap: wrap;
  font-family: monospace;
  margin: -2px -5px;
  user-select: none;
  pointer-events: none;
}



.caret {
  position: absolute;
  top: 0;
  left: 0;
  height: calc(24px * 1.35);
  width: 2px;
  background: red;
  transition: 100ms;
  // animation: blink 1s infinite;
}

@keyframes blink {
  0%, to {
    opacity: 0
  }
  50% {
    opacity: 1
  }
}

textarea {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
}

</style>
