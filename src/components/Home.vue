<template>
  <div id="home">
    <div class='content' ref="contentf">
      <!-- <div id="tan" :class="showData.probeStatus===1?'greendiv':showData.probeStatus===0?'graydiv':showData.probeStatus===3?'reddiv':'yellowdiv'" style="width:320px;minHeight:280px;" :style="{left:left + 'px',top:top + 'px'}" v-show="bool">
        <div class="box">
          <div class="equpic"><img src="../assets/equ.png" alt=""></div>
          <div class="gastype">{{showData.gasType}}</div>
          <div class="gasnum">{{showData.concentration}} <span>{{showData.gasUnit}}</span></div>
          <div class="equstatus">探头状态： <span :class="showData.probeStatus===1?'green':showData.probeStatus===0?'gray':showData.probeStatus===3?'red':'yellow'">{{showData.probeStatus===1?'正常':showData.probeStatus===0?'丢失':showData.probeStatus===3?'高报':'低报'}}</span></div>
          <div class="equposition">位置： {{showData.probePosition}}</div>
          <div class="equtime">时间： {{showData.createTime}}</div>
        </div>
      </div> -->
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OutlinePass } from "three/examples/jsm/postprocessing/OutlinePass.js";
// test
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass.js";
import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass.js";
export default {
  name: "Home",
  data() {
    return {
      scene: {},
      camera: {},
      renderer: {},
      allGroup: [],
      homeTop: [],
      bool: false,
      equList: [],
      showData: {},
      left: 0,
      top: 0,
      texture: {},
      composer: {},
      outlinePass: {},
      mixer: {},
      clock: {},
    };
  },
  methods: {
    init() {
      this.clock = new THREE.Clock();
      // 辅助坐标系
      let axesHelper = new THREE.AxesHelper(500);
      this.scene = new THREE.Scene();
      this.scene.add(axesHelper);
      this.loder();
      this.tree()
      this.light();

      let width = window.innerWidth; //窗口宽度
      let height = window.innerHeight; //窗口高度
      //创建相机对象
      this.camera = new THREE.PerspectiveCamera(60, width / height, 1, 3000);
      this.camera.position.set(0, 400, 0); //设置相机位置
      this.camera.lookAt(this.scene.position); //设置相机方向(指向的场景对象)
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight); //设置渲染区域尺寸
      this.renderer.setClearColor("skyblue", 1); //设置背景颜色
      this.$refs.contentf.appendChild(this.renderer.domElement);

      // 鼠标控制器
      new OrbitControls(this.camera, this.renderer.domElement);
      // 轮廓
      this.composer = new EffectComposer(this.renderer); //效果组合器
      let renderPass = new RenderPass(this.scene, this.camera); //原始场景渲染结果
      this.composer.addPass(renderPass);
      this.outlinePass = new OutlinePass(
        new THREE.Vector2(window.innerWidth, window.innerHeight),
        this.scene,
        this.camera
      ); //轮廓通道
      this.composer.addPass(this.outlinePass);
    },
    render() {
      this.renderer.render(this.scene, this.camera); //执行渲染操作
      this.composer.render();
      let delta = this.clock.getDelta();
      this.mixer.update(delta);
      requestAnimationFrame(this.render);
    },
    // 点光源
    light() {
      //点光源
      let point = new THREE.DirectionalLight("#fff");
      let point1 = new THREE.DirectionalLight("#fff", 0.7);
      point.position.set(-800, 800, 800); //点光源位置
      // point.position.set(0,100,500); //点光源位置
      point1.position.set(800, -800, -800); //点光源位置
      this.scene.add(point); //点光源添加到场景中
      // this.scene.add(point1); //点光源添加到场景中
      //环境光
      let ambient = new THREE.AmbientLight("#fff", 0.5);
      this.scene.add(ambient);
    },
    //树木
    tree() {
      let group = new THREE.Group();
      let w = 900,
        h = 510;
      let textureTree = new THREE.TextureLoader().load("/static/pic/tree.png");
      let spriteMaterial = new THREE.SpriteMaterial({
        map: textureTree,
      });
      let sprite = new THREE.Sprite(spriteMaterial);
      let H = 10;
      sprite.scale.set(6, H, 2);
      sprite.translateY(H / 2);
      for (let i = 0; i < 90 + 1; i++) {
        for (let j = 0; j < 51 + 1; j++) {
          if (j * 10 == 510 && i * 10 > 460 && i * 10 < 650) {
          } else if (
            i * 10 == 0 ||
            i * 10 == 900 ||
            j * 10 == 0 ||
            j * 10 == 510
          ) {
            let Sprite = sprite.clone();
            Sprite.translateX(i * 10 - w / 2);
            Sprite.translateZ(j * 10 - h / 2);
            group.add(Sprite);
          }
        }
      }
      group.scale.set(0.96, 0.96, 0.98);
      group.translateX(-5);
      this.scene.add(group);
    },
    //加载模型
    loder() {
      this.allGroup = new THREE.Group();
      let gltfLoader = new GLTFLoader();
      gltfLoader.load(
        "/static/gltf/Z51.gltf",
        (gltf) => {
          this.allGroup = gltf.scene
          console.log(this.allGroup,'this.allGroup');
          this.scene.add(gltf.scene);
          this.mixer = new THREE.AnimationMixer(gltf.scene);
          this.mixer.clipAction(gltf.animations[0]).play();
          this.render();
        },
        undefined,
        (error) => {
          console.log(error, "error");
        }
      );
    },
    // 探头克隆
    clone3d(obj3d, x, y, z, name) {
      let obj = obj3d.clone();
      obj.name = name;
      obj.position.set(x, y, z);
      return obj;
    },
    // 点击事件
    choose(event) {
      let Sx = event.clientX;
      let Sy = event.clientY;
      this.left = Sx + 20;
      this.top = Sy + 20;
      let x = (Sx / window.innerWidth) * 2 - 1;
      let y = -(Sy / window.innerHeight) * 2 + 1;
      let raycaster = new THREE.Raycaster();
      raycaster.setFromCamera(new THREE.Vector2(x, y), this.camera);
      let intersects = raycaster.intersectObjects(this.allGroup.children, true);
      if (intersects.length > 0) {
        let group = intersects[0].object.parent;
        if (group.name.includes('厂房')) {
          this.outlinePass.selectedObjects = [group];
          console.log("fangzi");
        } else {
          console.log("其他");
        }
      } else {
        console.log("模型外");
      }
    },
  },
  created() {
    this.$api.home.getData().then((res) => {
      if (res.data.errno === 0) {
        this.equList = res.data.data[0].probeData.map((item) => {
          for (const key in item) {
            if (item[key] === null || item[key] === undefined) {
              item[key] = "--";
            }
          }
          return item;
        });
      }
    });
  },
  mounted() {
    this.$nextTick(() => {
      this.init();
      addEventListener("click", this.choose);
    });
  },
  computed: {},
};
</script>

<style scoped lang="scss">
#home {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: skyblue;
  #tan {
    position: absolute;
    color: #fff;
    z-index: 102;
    padding-left: 17px;
    padding-top: 21px;
    padding-bottom: 21px;
    .box {
      width: 100%;
      height: 100%;
      font-size: 20px;
      line-height: 30px;
      color: #fff;
      position: relative;
      .equpic {
        width: 67px;
        height: 62px;
        position: absolute;
        top: 0px;
        right: 21px;
        img {
          width: 100%;
          height: 100%;
        }
      }
      .gastype {
        font-size: 24px;
        font-weight: 500;
      }
      .gasnum {
        font-size: 38px;
        font-weight: 700;
        line-height: 44px;
        margin-bottom: 30px;
        span {
          font-size: 20px;
          line-height: 24px;
        }
      }
      .equstatus {
        font-weight: 500;
      }
    }
  }

  .green {
    color: #00ff6d;
  }

  .red {
    color: #ff0000;
  }

  .gray {
    color: #dddddd;
  }

  .yellow {
    color: #ffb800;
  }
  .greendiv {
    background: rgba(0, 0, 0, 0.6);
    border: 1px solid #dddddd;
    box-sizing: border-box;
  }

  .reddiv {
    background: rgba(0, 0, 0, 0.6);
    border: 3px solid #ff0000;
    box-sizing: border-box;
    box-shadow: 0px 0px 11px rgba(255, 0, 0, 0.3);
  }

  .graydiv {
    background: rgba(0, 0, 0, 0.6);
    border: 1px solid #dddddd;
    box-sizing: border-box;
  }

  .yellowdiv {
    background: rgba(0, 0, 0, 0.6);
    border: 3px solid #ffb800;
    box-sizing: border-box;
    box-shadow: 0px 0px 11px rgba(255, 184, 0, 0.3);
  }
}
</style>
