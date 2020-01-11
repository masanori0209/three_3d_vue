<template>
  <div class="key"></div>
</template>

<script>
export default {
  name: 'KeyEvents',
  props: ['isJumping'],
  data () {
    return {
    }
  },
  created () {
    window.addEventListener('keyup', this.handler)
  },
  beforeDestroy() {
    window.removeEventListener('keyup', this.handler);
  },
  methods: {
    handler (e) {
      console.log(e.keyCode)
      if (!this.isJumping) {
        switch (e.keyCode) {
          case 32:
            this.$emit('jump')
            break
          case 37:
          case 65:
            console.log('A')
            this.$emit('move', 'left')
            break
          case 39:
          case 68:
            console.log('D')
            this.$emit('move', 'right')
            break
          default:
            break
        }
      }
    }
  }
}
</script>

<style scoped>
.key {
  display: none;
}
</style>
