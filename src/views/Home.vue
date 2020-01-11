<template>
  <div class="home">
    <StartMenu
      @startGame="startGame"
      v-if="isStart"
    />
    <Main
      :score="score"
      :colorHP="colorHP"
      :isGameOver="isGameOver"
      :isDebug="isDebug"
      @calcColor="calcColor"
      @calcScore="calcScore"
      @calcGameOver="calcGameOver"
      v-if="!isStart&&!isGameOver"
    />
    <Result
      :score="score"
      :colorHP="colorHP"
      v-if="isGameOver"
    />
  </div>
</template>

<script>
import Main from "@/components/Main.vue"
import StartMenu from "@/components/StartMenu.vue"
import Result from "@/components/Result.vue"

export default {
  name: "home",
  components: {
    StartMenu,
    Main,
    Result
  },
  data () {
    return {
      colorHP    : {
        r: 0,
        g: 0,
        b: 0,
      },
      score      : 0,
      isGameOver : false,
      isStart    : true,
      isDebug    : false
    }
  },
  methods: {
    calcColor (color) {
      this.colorHP.r += color.r
      this.colorHP.g += color.g
      this.colorHP.b += color.b
      if (this.colorHP.r >= 255) this.colorHP.r = 255
      if (this.colorHP.g >= 255) this.colorHP.g = 255
      if (this.colorHP.b >= 255) this.colorHP.b = 255
      if (this.colorHP.r <= 0) this.colorHP.r = 0
      if (this.colorHP.g <= 0) this.colorHP.g = 0
      if (this.colorHP.b <= 0) this.colorHP.b = 0
      if (this.colorHP.r >= 255 && this.colorHP.g >= 255 && this.colorHP.b >= 255) this.calcGameOver()
    },
    calcScore () {
      this.score += Math.floor(1.5 * (this.colorHP.r / 100)) +
                    Math.floor(1.5 * (this.colorHP.g / 100)) + 
                    Math.floor(1.5 * (this.colorHP.b / 100)) + 1
    },
    calcGameOver () {
      this.isGameOver = true
    },
    startGame () {
      this.isStart = false
    }
  }
}
</script>
