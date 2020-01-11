<template>
  <div ref="stage" class="back">
    <div class="retry">
      <h2 v-if="colorHP.r + colorHP.g + colorHP.b >= 255 * 3">Game Clear</h2>
      <h2 v-else>Game Over</h2>
      <h2>Score: {{score}}</h2>
      <a href="/" class="retry-button">Retry</a>
    </div>
  </div>
</template>

<script>
import * as threeObj from '@/three/end'
import * as THREE from 'three'
export default {
  name: 'Result',
  props: ['score', 'colorHP'],
  data () {
    return {
      publicPath    : process.env.BASE_URL,
      MAXOBJ        : threeObj.MAXOBJ,
      MAXLEN        : threeObj.MAXLEN,
      scene         : threeObj.scene,
      renderer      : threeObj.renderer,
      camera        : threeObj.camera,
      ambientlight  : threeObj.ambientlight,
      directlight   : threeObj.directlight,
      lightHelper   : threeObj.lightHelper,
      floor         : threeObj.floor,
      gltfloader    : threeObj.gltfloader,
      clock         : threeObj.clock,
      mixer         : null,
      player        : null,
      controls      : null,
      animations    : null,
    }
  },
  created () {
    this.camera.position.set(0, 12, 20)
    this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    this.scene.add(this.camera)
    this.scene.add(this.floor)
    this.scene.add(this.directlight)
    this.scene.add(this.ambientlight)
    if (this.isDebug) this.scene.add(this.lightHelper)
    this.loadModel()
  },
  mounted () {
    this.$refs.stage.appendChild(this.renderer.domElement)
    this.animate()
  },
  methods: {
    loadModel () {
      this.gltfloader.load(`${this.publicPath}model/stickman.glb`, (obj) => {
        this.mixer = new THREE.AnimationMixer(obj.scene)
        console.log(obj)
        this.player = obj.scene
        this.player.traverse( ( node ) => {
          if ( node instanceof THREE.Mesh ) { 
            node.castShadow = true
            node.receiveShadow = true
          }
        } )
        this.animations = obj.animations
        this.mixer.clipAction(obj.animations[2]).play()
        this.scene.add(this.player)
      },
      function ( xhr ) {
        console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );
      },
      function ( error ) {
        console.error( error );
      })
    },
    animate () {
      this.renderer.render(this.scene, this.camera)
      if (this.mixer) this.mixer.update(this.clock.getDelta())
      requestAnimationFrame(this.animate)
    }
  }
}
</script>

<style scoped>
.back {
  position: relative;
  height: 100%;
  width: 100%;
  background-image: url('../assets/background/sky.jpg');
  background-size: cover;
}
.background {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -998;
  height: 100%;
  width: 100%;
}
.menu {
  position: fixed; top: 0; right: 0; width: 100%; height: 100%; z-index: 100; background: white;
}
.status {
  position: fixed; top: 0; right: 0; width: 200px;
  background: rgba(0, 0, 0, 0.5);
}
.action-menu {
  position: fixed;
  bottom: 0; left: 0; 
  display: flex; justify-content: space-between;
  width: 100%;
  height: 100px;
  background: #999999;
}
.action-menu > button {
  width: 33%; height: 100%;
  font-size: 36px;
  font-weight: 600;
  background: #f2f2f2;
  color: #009999;
}
.retry {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  background: #f0f0f099;
  width: 100%;
  height: 100%;
  border-radius: 16px;
  padding: 24px;
  z-index: 2;
}
.retry > h2 {
  font-size: 24px;
  font-weight: 600;
}
.retry > button {
  width: 80%;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
