<template>
  <div id="pixi4"></div>
</template>

<script>
import * as pixi from "pixi.js";

function keyboard(value) {
  let key = {};
  key.value = value;
  key.isDown = false;
  key.isUp = true;
  key.press = undefined;
  key.release = undefined;
  //The `downHandler`
  key.downHandler = (event) => {
    if (event.key === key.value) {
      if (key.isUp && key.press) key.press();
      key.isDown = true;
      key.isUp = false;
      event.preventDefault();
    }
  };

  //The `upHandler`
  key.upHandler = (event) => {
    if (event.key === key.value) {
      if (key.isDown && key.release) key.release();
      key.isDown = false;
      key.isUp = true;
      event.preventDefault();
    }
  };

  //Attach event listeners
  const downListener = key.downHandler.bind(key);
  const upListener = key.upHandler.bind(key);

  window.addEventListener("keydown", downListener, false);
  window.addEventListener("keyup", upListener, false);

  // Detach event listeners
  key.unsubscribe = () => {
    window.removeEventListener("keydown", downListener);
    window.removeEventListener("keyup", upListener);
  };

  return key;
}

export default {
  name: "pixi4",
  components: {},
  props: {},
  data() {
    return {
      loader: null,
    };
  },
  created() {},
  mounted() {
    this.initState();
  },
  watch: {},
  computed: {},
  methods: {
    initState() {
      this.loader = new pixi.Loader();
      let Rectangle = pixi.Rectangle;
      //Create a Pixi Application   创建一个pixi应用
      let app = new pixi.Application({
        width: window.innerWidth, // default: 800 宽度
        height: window.innerHeight, // default: 600 高度
        antialias: true, // default: false 反锯齿
        // transparent: true, // default: false 透明度
        resolution: 1, // default: 1 分辨率
        backgroundAlpha: 0, // 设置背景颜色透明度   0是透明
      });

      document.getElementById("pixi4").appendChild(app.view);
      app.renderer.view.style.display = "block";
      app.renderer.autoResize = true;
      window.onresize = function () {
        app.renderer.resize(window.innerWidth, window.innerHeight);
      };

      // 加载纹理贴图，创造精灵，并将精灵添加到stage舞台上
      let src2 = "./img/baokemeng1.json";
      this.loader.add([{ name: "damu", url: src2 }]).load(() => {
        let textures = this.loader.resources.damu.textures;
        // 场景
        let changjingsprite = this.setTexture(textures["changjing.png"]);
        app.stage.addChild(changjingsprite);
        // 裁剪人物
        let rectangle = new Rectangle(229, 1, 16.7, 22);
        let texture = textures["anim.png"];
        texture.frame = rectangle;
        let damu = this.setTexture(texture);
        app.stage.addChild(damu);
        // 场景大小和位置
        changjingsprite.scale.set(2, 2);
        changjingsprite.x = 100;
        // 大木在场景大小、位置
        damu.width = 33;
        damu.height = 44;
        damu.x = 220;
        damu.y = 180;
        damu.vx = 0;
        damu.vy = 0;

        // 大木移动
        app.ticker.add(() => this.gameLoop(damu));
        this.keyMove(damu);
      });

      // 监听src加载事件
      this.loader.onProgress.add((loader, resource) => {
        console.log("当前进度：", loader.progress, "%");
        console.log(resource.url);
      });
      this.loader.onError.add((loader, resource) => {
        console.log("载入图片或json失败了");
      });
    
    
      this.loader.onStart.add((loader, resource) => {
        console.log("开始当前进度：", loader.progress, "%");
      });
      this.loader.onComplete.add((loader, resource) => {
        console.log("完成当前进度：", loader.progress, "%");
      });
    },

    keyMove(person) {
      let left = keyboard("a"),
        up = keyboard("w"),
        right = keyboard("d"),
        down = keyboard("s");
      left.press = () => {
        person.vx = -1;
        person.vy = 0;
      };
      left.release = () => {
        person.vx = 0;
        person.vy = 0;
      };
      right.press = () => {
        person.vx = 1;
        person.vy = 0;
      };
      right.release = () => {
        person.vx = 0;
        person.vy = 0;
      };
      up.press = () => {
        person.vx = 0;
        person.vy = -2;
      };
      up.release = () => {
        person.vx = 0;
        person.vy = 0;
      };
      down.press = () => {
        person.vx = 0;
        person.vy = 2;
      };
      down.release = () => {
        person.vx = 0;
        person.vy = 0;
      };
    },
    gameLoop(person) {
      //Move the cat 1 pixel
      person.x += person.vx;
      person.y += person.vy;
    },
    setTexture(texture) {
      // console.log(texture)
      let sprite = new pixi.Sprite(texture);
      return sprite;
    },
  },
};
</script>
<style scoped>
</style>
