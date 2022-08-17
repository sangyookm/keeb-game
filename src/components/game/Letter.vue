<template>
  <span :class="[
    `letter`,
    {
      'letter--correct': isCorrect,
      'letter--wrong': isWrong
    }
  ]">
    {{letter}}
  </span>
</template>


<script>
export default {
  props: {
    letter: {
      type: String
    },
    letterIndex: {
      type: Number
    },
    isCorrect: {
      type: Boolean
    },
    isWrong: {
      type: Boolean
    }
  },
  components: {
  }
}
</script>

<style lang="scss" scoped>

.letter {
  display: inline-block;
  opacity: 0.75;
}

.letter--correct {
  opacity: 1;
  color: white;
}
.letter--wrong {
  opacity: 1;
  color: red  ;
}

</style>