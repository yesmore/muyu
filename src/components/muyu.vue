<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";
import axios from "axios";
import MP3 from "../assets/muyu.mp3";

const is_qiao = ref(false);
const current_gongde = ref(-1);
const timer = ref();
const auto_update_timer = ref();
const audio = ref();

onMounted(() => {
  updateGongDe(0);
  onInitAudio();
  timer.value = setInterval(() => current_gongde.value--, 1000);
  auto_update_timer.value = setInterval(() => updateGongDe(-10), 10000); // 后端定时任务
});

onUnmounted(() => {
  clearInterval(timer.value);
  clearInterval(auto_update_timer.value);
});

const onInitAudio = () => {
  audio.value = new Audio();
  audio.value.src = MP3;
};

const rest_gongde = computed(() =>
  current_gongde.value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
);

const handleQiao = () => {
  if (is_qiao.value === false) {
    is_qiao.value = true;
    audio.value.play();
    current_gongde.value++;
    updateGongDe(1);
    setTimeout(() => (is_qiao.value = false), 100);
  }
};

const updateGongDe = async (current_count: number) => {
  try {
    const res = await axios.post("https://gcloud.aoau.top/gongde/update", {
      current_count,
    });
    if (res.data.msg === "success") {
      current_gongde.value = res.data.count;
    }
  } catch (error) {
    console.error(error);
  }
};
</script>

<template>
  <div class="container">
    <h1>你功德没了</h1>
    <p class="info">
      由于你扶老奶奶过马路后，向其索要小费而不得后又将老奶奶送回马路对面，导致
      <b>你的功德会一直递减，</b>
      但是不要惊慌！你可以<b>敲击下面的木鱼来增加功德！</b>
      <br />
      <b class="tip">注意！功德归零后网站将会关闭！</b>救赎自己吧！
    </p>
    <div class="rest-bar">
      当前剩余功德：
      <p>
        <b class="rest">{{ rest_gongde }}</b>
      </p>
    </div>

    <div class="main-bar">
      <div class="gunzi" :class="[is_qiao ? 'rotate' : '']">
        <img
          width="90"
          src="../assets/gunzi.png"
          alt="gunzi"
          @click="handleQiao"
        />
      </div>
      <div class="muyu-img">
        <img width="100" src="../assets/muyu.png" alt="muyu" />
      </div>
      <!-- <audio ref="audio" controls="true" :src="MP3" :autoplay="false"></audio> -->
    </div>

    <footer class="footer">
      该功德救赎器由
      <b><a href="https://github.com/yesmore" target="_blank">yesmore</a></b>
      提供
      <br />
      他一直这么善良的，不用谢
    </footer>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.container .info {
  max-width: 375px;
}
.container .info .tip {
  color: red;
}
.container .rest {
  color: red;
  font-size: 30px;
}

.main-bar {
  margin-top: 100px;
  position: relative;
}

.main-bar .gunzi {
  position: absolute;
  top: -40px;
  left: 50px;
  transform: rotate(45deg);
  transition: all 0.05s ease-in-out;
}

.main-bar .rotate {
  transform: rotate(-45deg);
}

.container .footer {
  position: fixed;
  bottom: 10px;
}
</style>
