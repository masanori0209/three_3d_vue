<template>
  <div ref="stage" class="back">
    <!-- Status -->
    <div class="status">
      <p v-if="isDebug">{{camera.position.x}} - {{camera.position.y}} - {{camera.position.z}}</p>
      <h2 :style="getColor()">Score: {{score}}</h2>
      <p>R: <progress class="progress-red"   max="255" :value="colorHP.r">{{colorHP.r}}%</progress></p>
      <p>G: <progress class="progress-green" max="255" :value="colorHP.g">{{colorHP.g}}%</progress></p>
      <p>B: <progress class="progress-blue"  max="255" :value="colorHP.b">{{colorHP.b}}%</progress></p>
    </div>
    <!-- Action Menu -->
    <div class="action-menu">
      <button @click="move('left')">←</button>
      <button @click="jump()" :disabled="isJumping">Jump</button>
      <button @click="move('right')">→</button>
    </div>
    <KeyEvents @jump="jump()" @move="move($event, type)" :isJumping="isJumping"/>
  </div>
</template>

<script>
import * as threeObj from '@/three/main'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import KeyEvents from '@/components/KeyEvents'
export default {
  name: 'Three',
  components : {
    KeyEvents,
  },
  props: ['score', 'colorHP', 'isGameOver', 'isDebug'],
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
      deadzoneMesh  : threeObj.deadzoneMesh,
      colorMesh     : threeObj.colorMesh,
      gltfloader    : threeObj.gltfloader,
      clock         : threeObj.clock,
      mixer         : null,
      player        : null,
      controls      : null,
      animations    : null,
      isJumping     : false,
    }
  },
  created () {
    if (window.innerWidth < 500) {
      this.camera.position.set(4, 11, 21)
      this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    } else {
      this.camera.position.set(10, 7, 15)
      this.camera.lookAt(new THREE.Vector3(0, 0, 0))
    }
    this.scene.add(this.camera)
    this.scene.add(this.floor)
    this.scene.add(this.directlight)
    this.scene.add(this.ambientlight)
    if (this.isDebug) this.scene.add(this.lightHelper)
    this.loadModel()
    // object 初期配置
    for (var i = 0; i < this.MAXOBJ; i++) {
      this.deadzoneMesh[i].position.z -= this.getRandomInt(15, this.MAXLEN/2)
      this.deadzoneMesh[i].position.x = this.getRandomInt(-1, 1) * 3
      this.colorMesh[i].position.z -= this.getRandomInt(15, this.MAXLEN/2)
      this.colorMesh[i].position.x = this.getRandomInt(-1, 1) * 3
      this.colorMesh[i].position.y = 4
      this.colorMesh[i].material.color.set('rgb(' + this.getRandomInt(0, 255) + ',' + this.getRandomInt(0, 255) + ',' + this.getRandomInt(0, 255) +')')
      this.deadzoneMesh[i].receiveShadow = true
      this.deadzoneMesh[i].castShadow = true
      this.colorMesh[i].receiveShadow = true
      this.colorMesh[i].castShadow = true
      this.scene.add(this.deadzoneMesh[i])
      this.scene.add(this.colorMesh[i])
      console.log(this.scene)
    }
  },
  mounted () {
    this.startObj()
  },
  methods: {
    startObj () {
      this.$refs.stage.appendChild(this.renderer.domElement)
      if (this.isDebug) {
        this.controls = new OrbitControls( this.camera, this.renderer.domElement )
        this.controls.update()
      }
      // this.run()
      this.animate()
    },
    getColor () {
      return {
        color: 'rgb(' + (255 - this.colorHP.r)  + ',' + (255 - this.colorHP.g)  + ',' + (255 - this.colorHP.b) + ')'
      }
    },
    getRandomInt (min, max) {
      return Math.floor( Math.random() * (max + 1 - min) ) + min
    },
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
        this.player.useQuaternion = true
        let q = new THREE.Quaternion()
        q.setFromAxisAngle(new THREE.Vector3(0,1,0).normalize(), Math.PI);
        this.player.quaternion.copy(q)
        this.animations = obj.animations
        this.mixer.clipAction(obj.animations[6]).play()
        this.scene.add(this.player)
      },
      function ( xhr ) {
        console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );
      },
      function ( error ) {
        console.error( error );
      })
    },
    jump () {
      this.mixer.stopAllAction()
      let anime = this.mixer.clipAction(this.animations[0])
      anime.setLoop(THREE.LoopOnce)
      anime.clampWhenFinished = true
      anime.play()
      this.isJumping = true
      this.mixer.addEventListener('finished', event => {
        this.isJumping = false
        this.run()
      })
    },
    run () {
      this.mixer.stopAllAction()
      let anime = this.mixer.clipAction(this.animations[6])
      anime.play()
    },
    move (type) {
      switch (type) {
        case 'left':
          this.player.position.x <= -3 ? this.player.position.x = -3 : this.player.position.x -= 3
          break
        case 'right':
          this.player.position.x >= 3 ? this.player.position.x = 3 : this.player.position.x += 3
          break
        default:
          break
      }
    },
    animate () {
      this.renderer.render(this.scene, this.camera)
      if (this.controls && this.isDebug) this.controls.update()
      if (this.mixer) this.mixer.update(this.clock.getDelta())
      if (this.player && this.mixer && this.animations) this.getResult()
      for (var i = 0; i < this.MAXOBJ; i++) {
        this.deadzoneMesh[i].position.z >= Math.floor(this.MAXLEN/4) ? this.deadzoneMesh[i].position.z = -this.MAXLEN/4 : this.deadzoneMesh[i].position.z += 0.5
        this.colorMesh[i].position.z    >= Math.floor(this.MAXLEN/4) ? this.colorMesh[i].position.z    = -this.MAXLEN/4 : this.colorMesh[i].position.z    += 0.5
      }
      requestAnimationFrame(this.animate)
    },
    getResult () {
      // judge gameover
      if (!this.isJumping) {
        for (var i = 0; i < this.MAXOBJ; i++) {
          if (this.deadzoneMesh[i].position.z >= this.player.position.z - 0.2 && this.deadzoneMesh[i].position.z <= this.player.position.z + 0.2 && this.deadzoneMesh[i].position.x == this.player.position.x) {
            this.$emit('calcGameOver')
          } else if (this.deadzoneMesh[i].position.z >= Math.floor(this.MAXLEN/4)) {
            this.deadzoneMesh[i].position.z = -this.MAXLEN/4 + this.getRandomInt(-50, 5)
          }
        }
      }
      // color get
      for (var i = 0; i < this.MAXOBJ; i++) {
        if (this.colorMesh[i].position.z >= this.player.position.z - 0.2 && this.colorMesh[i].position.z <= this.player.position.z + 0.2 && this.colorMesh[i].position.x == this.player.position.x) {
          this.$emit('calcColor', {
            r: Math.floor((this.colorMesh[i].material.color.r) * 20),
            g: Math.floor((this.colorMesh[i].material.color.g) * 20),
            b: Math.floor((this.colorMesh[i].material.color.b) * 20)
          })
          this.colorMesh[i].position.z = -this.MAXLEN/4 + this.getRandomInt(-50, 5)
        }
      }
      // calc result
      this.$emit('calcScore')
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
.status > p {
  color: white;
  font-size: smaller;
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 95%;
  white-space: pre-line;
}
.progress-red[value], .progress-green[value], .progress-blue[value] {
  appearance: none;
  background-color: #22222299;
  border: none;
  border-radius: 4px;
  color: #fff;
  height: 8px;
}
.progress-red[value]::-webkit-progress-bar {
  background-color: #222;
}
.progress-red[value]::-webkit-progress-value {
  background-color: #f99;
}
.progress-red[value]::-moz-progress-bar {
  background-color: #f99;
}
.progress-green[value]::-webkit-progress-bar {
  background-color: #222;
}
.progress-green[value]::-webkit-progress-value {
  background-color: #9f9;
}
.progress-green[value]::-moz-progress-bar {
  background-color: #9f9;
}
.progress-blue[value]::-webkit-progress-bar {
  background-color: #222;
}
.progress-blue[value]::-webkit-progress-value {
  background-color: #99f;
}
.progress-blue[value]::-moz-progress-bar {
  background-color: #99f;
}
.action-menu {
  position: fixed;
  bottom: 0; left: 0; 
  display: flex; justify-content: space-between;
  width: 100%;
  height: 100px;
  background: #ddf;
}
.action-menu > button {
  width: 33%; height: 100%;
  font-size: 36px;
  font-weight: 600;
  background: #f2f2f2;
  color: #222222;
  border-radius: 16px;
}
</style>
