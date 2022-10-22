<template>
  <div class="page-container">
    <!-- 背景颜色/背景图片 -->
    <div class="page-bgc"></div>

    <div class="page" id="page">
      <div class="game" id="game"></div>
      <!-- 内容-容器 -->
      <!-- <div class="header">1</div>
      <div class="content">2</div>
      <div class="footer">3</div> -->
    </div>
  </div>
</template>

<script>
// application用于初始化pixi.js

// 按需引入，个人不是很喜欢
// import { Application, Sprite } from "pixi.js";

import * as PIXI from "pixi.js";

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
  name: "Index",

  //变量区
  data() {
    return {
      test: "123",
      loader: null, //用于装Pixi的loader
      isPlayGame: false, //是否开始游戏
      isTurnRight: true, //是否往右走
    };
  },
  //生命周期区
  mounted() {
    this.initPixi();

    //测试用的代码，已废弃

    // let page = document.getElementById('page')
    // // let gameContainerInfo = document.querySelector('.game')
    // console.log(game)

    //     const gameConfig = new Application({
    //   width:page.offsetWidth,
    //   height:page.offsetHeight
    // })
    // console.log(page.offsetHeight)
    // game.append(gameConfig.view)

    // console.log(this.$store);
    // this.JKTest();
  },
  //方法区
  methods: {
    JKTest() {
      console.log("测试一下");
    },
    //初始化pixi
    initPixi() {
      // const renderer = PIXI.autoDetectRenderer();

      // 创建loader
      this.loader = new PIXI.Loader();

      // 通过Pixi创建容器
      let app = new PIXI.Application({
        width: 256, // default: 800 宽度
        height: 256, // default: 600 高度
        antialias: true, // default: false 反锯齿
        backgroundAlpha: 0.5, //设置背景透明度
        // backgroundColor: 0xffffff, //设置背景颜色

        resolution: 1, // default: 1 分辨率
        // forceCanvas: true, //使用Canvas来渲染
      });

      // 自适应屏幕
      // app.renderer.view.style.position = "absolute";
      app.renderer.view.style.display = "block";
      // app.renderer.autoResize = true;

      // 利用这个resize函数+windows 自带的resize函数 我们可以自适应任何屏幕了
      app.renderer.resize(window.innerWidth, window.innerHeight);

      app.renderer.resize(window.innerWidth, window.innerHeight); //改变舞台大小
      // 此处可以优化，卸载页面时，可以取消该监听
      window.onresize = function () {
        app.renderer.resize(window.innerWidth, window.innerHeight);
      };

      // 查询我们要放在哪个元素下面
      let gameContainer = document.querySelector(".game");
      // console.log(gameContainer);
      //将创建的PIVIAPP 放到容器下面
      gameContainer.append(app.view);

      // 创建图片
      // 从图像路径创建新的精灵,需要图片放在pulic里面
      // 不推荐该方法来创建图片

      // const bunny = PIXI.Sprite.from("/img/bunny.png");

      // 推荐方法创建图片
      this.loader
        .add([
          { name: "bunny", url: "img/bunny.png" },
          { name: "bunny2", url: "img/bunny.png" },
        ])

        .load((loader, resources) => {
          console.log(resources);
          console.log(loader);
          // 小熊1
          const bunny = new PIXI.Sprite(resources.bunny.texture);
          // 设置旋转的中心点
          bunny.anchor.set(0.5);

          // 将精灵移到屏幕中央
          bunny.x = app.screen.width / 2;
          bunny.y = app.screen.height / 2;
          bunny.vx = 0;
          bunny.vy = 0;

          app.stage.addChild(bunny);

          // 小熊2
          const bunny2 = new PIXI.Sprite(resources.bunny2.texture);
          // 设置旋转的中心点
          bunny2.anchor.set(0.5);

          // 将精灵移到屏幕中央
          bunny2.x = app.screen.width / 4;
          bunny2.y = app.screen.height / 4;
          // vx vy 代表的是方向与速度
          //vx =1 往右移动 距离为1
          //vy =1 往下移动 距离为1
          //vx = 1 vy = 1  往右下角移动
          bunny2.vx = 0;
          bunny2.vy = 0;

          app.stage.addChild(bunny2);

          console.log(bunny2);

          // renderer.render(bunny)
          // console.log(renderer);

          // app.stage.addChild(bunny);

          setTimeout(() => {
            this.isPlayGame = true;
          }, 2000);

          app.ticker.add(() => {
            // 为了好玩，让我们旋转兔子先生一点
            bunny.rotation = bunny.rotation + 0.05;

            // this.gameLoop(bunny)

            //控制是否开始游戏
            if (this.isPlayGame) {
              this.bearGameLoop(bunny2);
            }
          });
          // this.keyMove(bunny);
        });

      // 监听src加载事件
      this.loader.onProgress.add((loader, resource) => {
        console.log("当前进度：", loader.progress, "%");
        console.log(resource.url);
      });

      this.loader.onError.add(() => {
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
    bearGameLoop(person) {
      // 最大宽度移动距离 = 屏幕的宽度 -物体的宽度
      let maxWidth = window.innerWidth - person.width;
      // 最大高度移动距离 = 屏幕的宽度 -物体的宽度

      let maxHeight = window.innerHeight - person.height;

      let minWidth = person.width;
      let minHeight = person.height;

      // console.log(person.x < maxWidth,person.y < maxHeight)

      // if (person.x < maxWidth && person.y < maxHeight ) {
      //   person.vx = 1;
      //   person.vy = 1;

      //   person.x = person.x + person.vx;
      //   person.y = person.y + person.vy;
      // }

      // if (person.x -maxWidth>0 ) {
      //   isTurnRight = false;
      // }

      // if  (person.x-person.width  <0 ) {
      //   isTurnRight = true;
      // }

      if (this.isTurnRight === true) {
        // console.log(maxWidth > person.x)
        console.log(this.isTurnRight);
        if (maxWidth > person.x) {
          person.vx = 1;
          person.x = person.x + person.vx;
        } else {
          console.log("执行");
          this.isTurnRight = false;
        }
      } else {
        if (person.x > minWidth) {
          person.vx = -1;
          person.x = person.x + person.vx;
        } else {
          this.isTurnRight = true;
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.page-container {
  box-sizing: border-box;
  position: relative;
  //   height: fit-content;
  width: 100vw;
  height: 100vh;
  //   background-color: rgba(grey, 0.1);

  //不加的话 iphone5会多出来一块
  overflow-x: hidden;

  //   背景颜色/背景图片
  .page-bgc {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: url("@/assets/img/ready-step-bgc.jpg");
    background-size: 100% 100%;
    background-repeat: no-repeat;
  }

  .page {
    box-sizing: border-box;
    position: relative;
    //   height: fit-content;
    width: 100vw;
    height: 100vh;
    background-color: rgba(grey, 0.1);

    //不加的话 iphone5会多出来一块
    overflow-x: hidden;
    //   设置背景
  }
}
</style>
