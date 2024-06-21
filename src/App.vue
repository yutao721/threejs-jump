<!--
 * @Descripttion: 
 * @Author: yutao
 * @Date: 2024-05-20 19:07:35
 * @FilePath: \threejs-demo\src\App.vue
 * @LastEditors: yutao
 * @LastEditTime: 2024-06-21 14:48:01
-->
<template>
  <div id="container">
    <div id="score">0</div>
    <div id="fail">您的分数为 <span id="score2">0</span></div>
  </div>
</template>

<script setup lang="ts">
import * as THREE from "three";
import { onMounted } from "vue";

onMounted(() => {
  // 获取容器元素
const container = document.getElementById("container") as HTMLElement;
  // 获取浏览器窗口的内部宽度
  const width = window.innerWidth;
  // 获取浏览器窗口的高度
  const height = window.innerHeight;
  // 创建透视相机
  const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000);

  // 创建场景
  const scene = new THREE.Scene();
  // 创建渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true });

  // 设置渲染器的大小
  renderer.setSize(width, height);
  // 设置渲染器的清除颜色
  renderer.setClearColor(0x333333);

  // 设置相机的位置和朝向
  camera.position.set(100, 100, 100);
  camera.lookAt(scene.position);

  // 创建方向光
  const directionalLight = new THREE.DirectionalLight(0xffffff);
  // 设置方向光的的位置
  directionalLight.position.set(40, 100, 60);

  // 将场景添加到场景中
  scene.add(directionalLight);

  // 将渲染器添加到容器中
  container.appendChild(renderer.domElement);

  // const axesHelper = new THREE.AxesHelper( 1000 );
  // axesHelper.position.set(0,0,0);
  // scene.add( axesHelper );

  const targetCameraPos = { x: 100, y: 100, z: 100 };

  const cameraFocus = { x: 0, y: 0, z: 0 };
  const targetCameraFocus = { x: 0, y: 0, z: 0 };

  const playerPos = { x: 0, y: 17.5, z: 0 };
  const targetPlayerPos = { x: 0, y: 17.5, z: 0 };

  let player;
  let speed = 0;
  let speedY = 0;

  const currentCubePos = { x: 0, y: 0, z: 0 };
  let direction = "right";

  let pressed = false;


 // 定义一个函数，用于创建一个立方体
 function createCube(x, z) {
    // 创建一个立方体的几何体
    const geometry = new THREE.BoxGeometry(30, 20, 30);
    // 创建一个立方体的材质
    const material = new THREE.MeshPhongMaterial({ color: 0xffffff });
    // 创建一个立方体的网格
    const cube = new THREE.Mesh(geometry, material);
    // 设置立方体的位置
    cube.position.x = x;
    cube.position.z = z;
    // 将立方体添加到场景中
    scene.add(cube);
  }

  // 创建玩家
function create() {
    // 创建玩家
    function createPlayer() {
      // 创建一个立方体几何体
      const geometry = new THREE.BoxGeometry(5, 15, 5);
      // 创建一个材质
      const material = new THREE.MeshPhongMaterial({ color: 0xffff00 });
      // 创建一个立方体网格
      const player = new THREE.Mesh(geometry, material);
      // 设置立方体网格的位置
      player.position.x = 0;
      player.position.y = 17.5;
      player.position.z = 0;
      // 将立方体网格添加到场景中
      scene.add(player);
      // 返回立方体网格
      return player;
    }

    // 创建玩家
    player = createPlayer();

    // 创建一个立方体
    createCube(0, 0);

    // 监听鼠标按下事件
    document.body.addEventListener("mousedown", () => {
      // 设置速度为0
      speed = 0;
      // 设置速度Y为0
      speedY = 0;
      // 设置按下状态为true
      pressed = true;
    });

    // 监听鼠标放下事件
    document.body.addEventListener("mouseup", () => {
      // 设置按下状态为false
      pressed = false;
    });
  }

  function moveCamera() {
    if (player.position.y > 17.5) {
      if (direction === "right") {
        camera.position.z -= speed;
        cameraFocus.z -= speed;
      } else {
        camera.position.x -= speed;
        cameraFocus.x -= speed;
      }

      camera.lookAt(cameraFocus.x, cameraFocus.y, cameraFocus.z);
    }
  }

  function playerInCube() {
    if (direction === "right") {
      if (
        player.position.z < currentCubePos.z + 15 &&
        player.position.z > currentCubePos.z - 15
      ) {
        return true;
      }
    } else if (direction === "left") {
      if (
        player.position.x < currentCubePos.x + 15 &&
        player.position.x > currentCubePos.x - 15
      ) {
        return true;
      }
    }
    return false;
  }

  let score = -1;
  let stopped = true;
  function movePlayer() {
    player.position.y += speedY;

    if (player.position.y < 17.5) {
      player.position.y = 17.5;

      if (stopped === false) {
        if (playerInCube()) {
          score++;
          document.getElementById("score").innerHTML = score;

          const distance = Math.floor(50 + Math.random() * 100);

          const num = Math.random();
          if (num > 0.5) {
            currentCubePos.z -= distance;
            direction = "right";
          } else {
            currentCubePos.x -= distance;
            direction = "left";
          }
          createCube(currentCubePos.x, currentCubePos.z);
        } else {
          document.getElementById("fail").style.display = "block";
          document.getElementById("score2").innerHTML = score;
        }
      }

      stopped = true;
    } else {
      stopped = false;
      if (direction === "right") {
        player.position.z -= speed;
      } else {
        player.position.x -= speed;
      }
    }
    speedY -= 0.3;
  }

  function speedUp() {
    if (pressed) {
      speed += 0.1;
      speedY += 0.1;

      if (player.scale.y > 0) {
        player.scale.y -= 0.001;
      }
      player.position.y -= 15 - 15 * player.scale.y;

      if (player.position.y < 10) {
        player.position.y = 10;
      }
    } else {
      player.scale.y = 1;
    }
  }

  function render() {
    if (!pressed) {
      moveCamera();
      movePlayer();
    }
    speedUp();

    renderer.render(scene, camera);
    requestAnimationFrame(render);
  }

  create();
  render();
});
</script>

<style>
#container {
  width: 100%;
  height: 100%;
}
#score {
  font-size: 50px;
  color: #fff;
  position: fixed;
  top: 10%;
  left: 10%;
}
#fail {
  position: absolute;
  background: rgba(0, 0, 0, 0.5);
  left: 0;
  right: 0;
  width: 100%;
  height: 100%;
  font-size: 70px;
  color: #fff;
  display: none;
  text-align: center;
  padding-top: 400px;
}
</style>
