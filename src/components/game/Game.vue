<template>
<div class="game-wrap">
  <div class="before blur"></div>
  <div class="before"></div>
  <div class="after blur"></div>
  <div class="after"></div>
  <div class="game" ref="game">
    <div class="scroll-wrap">
      <div class="scroll" :style="
        `top: calc(${scrollAmountByLine} * -36px)`
      ">
        <div :class="[
          `caret`,
          {
            'caret--blink': !didStart
          }
        ]" :style="caretPosStyle"></div>
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
        </div>
      </div>
    </div>
    <textarea v-model="currentInput" readonly />
  </div>

  <div class="debug" v-if="debugMode">
    <div>
      currentLine: {{currentLine}}
    </div>
    <div>
      currentWordIndex: {{currentWordIndex}}
    </div>
    <div>
      currentWordEl: {{currentWordEl}}
    </div>
    <div>
      WPM: {{wpm}}
    </div>
  </div>
</div>
</template>

<script>
import randomWords from 'random-words'
import Word from './Word'
const getRandomInt = (min, max)=> {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min) + min); //The maximum is exclusive and the minimum is inclusive
}

export default {
  data() {
    return {
      words: [],
      inputtedWords: [],
      currentWordEl: null,
      currentWordIndex: 0,
      currentInput: "",
      didStart: false,
      lastInputTime: new Date().getTime(),
      startInputTime: new Date().getTime(),
      playSound: false,
      debugMode: false,
      caretPos: {
        top: 0,
        left: 0,
      },
      window: {
        width: window.innerWidth,
        height: window.innerHeight
      }
    }
  },
  created() {
    this.words = this.generateWords()
  },
  mounted() {
    this.attachEvents()
  },  
  beforeUnmount() {
    this.attachEvents(true)
  },
  watch: {
    currentWordIndex(n) {
      this.currentWordEl = this.$refs.word?.[n]?.$el || null
    },
    currentInput() {
      this.setCaretPos()
    }
  },
  methods: {
    generateWords() {
      return randomWords(100)
    },
    attachEvents(detach = false) {
      const listener = detach ? 'removeEventListener' : 'addEventListener'
      window[listener]('resize', this.onWindowResize, false)
      document[listener]('keypress', this.globalKeyPress, false)
      document[listener]('keydown', this.globalKeyDown, false)
    },
    globalKeyPress(e) {
      const {code, key, which} = e
      const isSpace = code === "Space" || which === 32
      const isEnter = code === "Enter"

      if (isEnter) return

      if (isSpace) {
        this.lastInputTime = new Date().getTime()
        return this.processSpace()
      }

      if (key) {
        this.lastInputTime = new Date().getTime()
        return this.processValidInput(e.key)
      } 
        
    },
    globalKeyDown(e) {
      const {code, key, which} = e
      const isBackspace = key === "Backspace" || code === "Backspace" || which === 8
      if (isBackspace) {
        e.preventDefault()
        this.lastInputTime = new Date().getTime()
        console.log("isBackspace")
        return this.processBackspace()
      }
    },
    processValidInput(key) {
      // console.log("key", key)
      this.currentInput += key

      if (!this.didStart) {
        this.didStart = true
        this.startInputTime = new Date().getTime()
        this.lastInputTime = new Date().getTime()
      }
      if (this.playSound) new Audio(require(`@/assets/key_${getRandomInt(1, 2)}.mp3`)).play()
      
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

      if (this.playSound) new Audio(require(`@/assets/key_space_1.mp3`)).play()


      this.currentWordIndex += 1
      this.currentInput = ""
    },
    processBackspace() {
      if (this.isBeginning) return
      if (this.playSound) new Audio(require(`@/assets/key_space_1.mp3`)).play()
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
    },
    onWindowResize() {
      this.window.innerWidth = window.innerWidth
      this.window.innerHeight = window.innerHeight
      this.setCaretPos()
    },
    setCaretPos() {
      const wordEl = this.$refs.word?.[this.currentWordIndex]?.$el

      if (!wordEl) return
      const letterRect = wordEl?.querySelector('.letter')?.getBoundingClientRect()
      const letterWidth = letterRect?.width || 0
      const currentLetterIndex = this.currentInput.length
  
      let top = wordEl ? wordEl.offsetTop : 0
      let left = wordEl ? wordEl.offsetLeft + (currentLetterIndex * letterWidth): 0

      this.caretPos = { top, left }
     
    }
  },
  computed: {
    isBeginning() {
      return this.currentInput.length <= 0 && this.currentWordIndex <= 0
    },
    currentLine() {
      const wordEl = this.currentWordEl
      if (wordEl) {
        // const stylesheet = getComputedStyle(document.documentElement)
        // const wordFontSize = Number(stylesheet.getPropertyValue('--game-font-size').trim().replace('px', '')) || 0
        // const wordLineHeight = Number(stylesheet.getPropertyValue('--game-line-height')) || 0 
        // const wordGapY = Number(stylesheet.getPropertyValue('--game-word-gap-v').trim().replace('px', '')) || 0
        // const wordHeight = wordFontSize * wordLineHeight
        
        const wordComputedStyle = getComputedStyle(wordEl)
        const wordGapY = Number(wordComputedStyle.getPropertyValue('margin-top').replace('px', '')) 

        const wordElRect = wordEl.getBoundingClientRect()
        const wordElPosYFromGameBox = wordEl.offsetTop
        const wordHeight = wordElRect.height
        const wordOuterHeight = Math.round(wordHeight + (wordGapY * 2))
        console.log({wordElPosYFromGameBox, wordHeight, wordOuterHeight, wordGapY})
        return Math.round(wordElPosYFromGameBox / wordOuterHeight) + 1
      } else {
        return 1
      }
    },
    scrollAmountByLine() {
      return (this.currentLine > 2) ? Math.abs(2 - this.currentLine) : 0
    },
    caretPosStyle() {
      return {
        top: `${this.caretPos.top}px`,
        left: `${this.caretPos.left}px`
      }
      // try {
      //   // console.log("refs.word", this.$refs.word)
      //   const wordEl = this.$refs.word?.[this.currentWordIndex]?.$el
      //   const letterRect = wordEl?.querySelector('.letter')?.getBoundingClientRect()
      //   const letterWidth = letterRect?.width || 0
      //   // console.log("letter", wordEl?.querySelector('.letter'))
      //   const currentLetterIndex = this.currentInput.length
    
      //   let top = wordEl ? wordEl.offsetTop : 0
      //   let left = wordEl ? wordEl.offsetLeft + (currentLetterIndex * letterWidth): 0
        
      //   // const contEl = this.$refs.wordsCont
      //   // console.log(top, left, letterRect)
      //   return {
      //     top: `${top}px`,
      //     left: `${left}px`,
      //   }
      // } catch {
      //   return {}
      // }
    },
    wpm() {
      
      const progressedTimeInSeconds = (this.lastInputTime - this.startInputTime)/1000
      const wpm = Math.round(this.inputtedWords.length / progressedTimeInSeconds * 60)
      console.log({
        inputtedWordsLen: this.inputtedWords.length,
        progressedTimeInSeconds,
        wpm
      })
      return wpm || 0
    }
  },
  components: {
    Word
  }
}
</script>

<style lang="scss" scoped>

.debug {
  position: fixed;
  bottom: 0;
  left: 0;
  background: #00de80;
  z-index: 999;
  padding: 4px;
}

.game-wrap {
  --game-size: calc(var(--game-font-size) * var(--game-line-height) * var(--game-show-rows) + var(--game-word-gap-v) * 2 * (var(--game-show-rows) - 1));
  position: relative;
  margin: auto;
  width: 100%;
  padding: 0 32px;
  display: flex; 
  height: 100%;
  .before {
    position: absolute;
    content: '';
    top: 0;
    left: 0;
    height: calc(50% - calc(var(--game-size) * 0.5));
    width: 100%;
    background: linear-gradient(0deg, transparent -200%, var(--game-bg-color));
    // mask: linear-gradient(0deg, transparent -200%, var(--game-bg-color));
    // background-color: red;
    backdrop-filter: blur(1.25px);
    z-index: 99;
    overflow: hidden
  }
  .before.blur {
    mask: linear-gradient(black, transparent 200%);
    backdrop-filter: blur(5px)
  }
  .after { 
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: calc(50% - calc(var(--game-size) * 0.5));
    content: '';
    background: linear-gradient(transparent -20%, var(--game-bg-color) 80%);
    z-index: 99;
    overflow: hidden;
    backdrop-filter: blur(1.25px);
  }
  .after.blur {
    mask: linear-gradient(transparent -100%, transparent);
    backdrop-filter: blur(5px)
  }
}


.game {
  position: relative;
  width: 100%;
  margin: auto;
  color: white;
  max-width: 1200px;
  
  // font-size: 24px;
  // line-height: 1.35;

  height: calc(
    (
      var(--game-font-size) * var(--game-line-height) * var(--game-show-rows)
    ) + 
    (
      var(--game-word-gap-v) * 2
      * (var(--game-show-rows) - 1)
    )
  );
}


.scroll-wrap {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: var(--game-size);
  
}
 .scroll {
  position: absolute;  
}

.words {
  display: flex;
  flex-wrap: wrap;
  margin: -2px -5px;
  user-select: none;
  pointer-events: none;
}



.caret {
  position: absolute;
  top: 0;
  left: 0;
  height: calc(24px * 1.35);
  width: 3px;
  transform: translate3d(-2px, 0, 0);
  // background-color: #00de80;
  background-image: linear-gradient(
    #00de80,
    #067145
  );
  transition: 100ms;
  &.caret--blink {
    animation: blink 1.5s infinite;
  }
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
  visibility: hidden;
}

</style>
