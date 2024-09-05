<template>
  <a-trigger position="top" auto-fit-position :unmount-on-close="false">
    <span
      ><a href="http://192.168.56.1:8000/my_chart">
        <section>
          <div class="box"></div>
          <div class="box-circle"></div>
          <div class="box-circle1"></div>
          <div class="box-circle2"></div>
        </section> </a
    ></span>
    <template #content>
      <div class="demo-basic">
        <a-space :style="{ margin: '10px 0' }">
          <a href="http://192.168.56.1:8000/my_chart" target="_blank">
            <a-button type="outline" @click="addChart('submit')"
              >分析提交数</a-button
            >
          </a>
          <a href="http://192.168.56.1:8000/my_chart" target="_blank">
            <a-button type="primary" @click="addChart('accept')"
              >分析通过率</a-button
            >
          </a>
        </a-space>
      </div>
    </template>
  </a-trigger>
</template>

<script setup lang="ts">
import axios from "axios";
import message from "@arco-design/web-vue/es/message";
import { ref } from "vue";

let comment2 = ref();
const addChart = async (param: any) => {
  const data = {
    param: param,
  };
  const config = {
    headers: {
      "Content-Type": "application/json",
      "Referrer-Policy": "unsafe-url",
    },
  };
  axios
    .post(
      "https://www.xianyuwang.online/api/comment/add/commentrelation",
      JSON.stringify(data),
      config
    )
    .then((response) => {
      console.log(response.data);
      console.log("success");
      message.success("ai分析中...");
    })
    .catch((error) => {
      console.log(error);
      console.log("error");
    });
};
</script>

<style scoped>
* {
  --color: orange;
  --lineColor: rgba(102, 163, 224, 0.2);
  //background-color: #222;
}

body {
  background-color: #222;
  overflow: hidden;
}

section {
  position: relative;
  width: 200px;
  height: 200px;
  background-color: #222222;
  cursor: pointer;
  border-radius: 100px;
}

section::before {
  content: "";
  position: absolute;
  height: 10px;
  width: 10px;
  border-radius: 100%;
  border-top: 1px solid orange;
  top: 50%;
  left: 50%;
  margin-top: -5px;
  margin-left: -5px;
  animation: turn 1s infinite linear;
  filter: drop-shadow(0 0 2px var(--color)) drop-shadow(0 0 5px var(--color))
    drop-shadow(0 0 10px var(--color)) drop-shadow(0 0 20px var(--color));
}

.box,
.box::after,
.box::before {
  border: 2px solid var(--lineColor);
  border-left: 2px solid var(--color);
  border-right: 2px solid var(--color);
  border-radius: 50%;
}

.box::after,
.box::before {
  position: absolute;
  content: "";
  left: 50%;
  top: 50%;
}

.box {
  width: 200px;
  height: 200px;
  position: relative;
  animation: turn 1s linear infinite;
  transform-origin: 50% 50%;
}

.box::before {
  width: 180px;
  height: 180px;
  margin-top: -90px;
  margin-left: -90px;
  animation: turn2 1.25s linear infinite;
}

.box::after {
  width: 160px;
  height: 160px;
  margin-top: -80px;
  margin-left: -80px;
  animation: turn 1.5s linear infinite;
}

.box-circle,
.box-circle1,
.box-circle2 {
  border: 2px solid var(--color);
  opacity: 0.9;
  border-radius: 50%;
  position: absolute;
  left: 50%;
  top: 50%;
  transform-origin: 50% 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
  animation: rotate 3s linear infinite;
}

.box-circle {
  animation-delay: 0.2s;
}

.box-circle1 {
  animation-delay: 1.2s;
}

.box-circle2 {
  animation-delay: 2.2s;
  z-index: 99;
}
@keyframes turn {
  100% {
    transform: rotateZ(-1turn);
  }
}

@keyframes turn2 {
  100% {
    transform: rotateZ(1turn);
  }
}

@keyframes rotate {
  100% {
    border: none;
    border-top: 2px solid var(--color);
    border-bottom: 2px solid var(--color);
    transform: translate(-50%, -50%) rotate3d(0.5, 0.5, 0.5, -720deg);
  }
}
</style>
