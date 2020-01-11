<template>
  <div ref="stage" class="back" @click="$emit('startGame')">
    <!-- Start Menu -->
    <div class="menu">
      <h2>Simple Action Game</h2>
      <p class="desc" style="padding: 24px; text-align: left;">
        カラフルなドーナツ状のアイテムを取って、スコアをどんどんあげていこう！
        赤い三角の障害物に当たるとゲームオーバー。
        色によってスコアが上昇、全ての色がMAXになるとゲームクリア。
      </p>
      <div class="click-start">
        クリックでスタート
      </div>
    </div>
  </div>
</template>

<script>
import * as threeObj from '@/three/start'
import * as THREE from 'three'
export default {
  name: 'StartMenu',
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
  position: fixed;
  top: 0; right: 0;
  width: 100%; height: 100%; z-index: 100;
  background: #f0f0f099;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.menu > h2 {
  padding: 24px;
  font-weight: 600;
}
.menu > .desc {
  padding: 24px;
  text-align: left;
  background: #f2f2f2;
  width: 80%;
  border-left: solid #ccf 10px;
}
.menu > .click-start {
  font-size: 24px;
  font-weight: 600;
  animation: 2s huwanhuwan infinite;
}
@keyframes huwanhuwan{
  0% { opacity: 0; }
  100% { opacity :1; }
}
</style>
