<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";
import axios from "axios";
import MP3 from "../assets/muyu.mp3";

const is_qiao = ref(false);
const current_gongde = ref(100000000);
// const timer = ref();
const auto_update_timer = ref();
const audio = ref();

onMounted(() => {
  updateGongDe(0);
  onInitAudio();
  // timer.value = setInterval(() => current_gongde.value--, 1000);
  // auto_update_timer.value = setInterval(() => updateGongDe(-10), 10000); // 后端定时任务
});

onUnmounted(() => {
  // clearInterval(timer.value);
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
    setTimeout(() => {
      if (is_qiao.value === true) {
        is_qiao.value = false;
      }
    }, 100);
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
    <div v-if="current_gongde > 0">
      <h1>你功德没了</h1>
      <p class="info">
        由于你扶老奶奶过马路后，向其索要小费而不得后又将老奶奶送回马路对面，导致
        <b>你的功德会一直递减 (1堒/秒)，</b>
        但是不要惊慌！你可以<b>敲击下面的木鱼来增加功德！</b>
        <br />
        <b class="tip">注意！功德归零后网站将会关闭！</b>救赎自己吧！
      </p>
      <div class="rest-bar">
        当前剩余功德：
        <p>
          <b class="rest">{{ rest_gongde }}</b> 堒
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
    </div>
    <div v-else>
      <h2>您的功德已归零，请联系站长重开</h2>
    </div>
    <footer class="footer">
      该功德救赎器由
      <b><a href="https://github.com/yesmore" target="_blank">yesmore</a></b>
      提供
      <br />
      他一直这么善良的，不用谢

      <div style="margin-top: 8px">
        <img
          src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fmuyu.aoau.top&count_bg=%2379C83D&title_bg=%23555555&icon=actigraph.svg&icon_color=%23E7E7E7&title=%E8%A2%AB%E6%95%91%E8%B5%8E%E8%80%85&edge_flat=true"
        />
      </div>
    </footer>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  user-select: none;
}

.container .info {
  max-width: 350px;
}
.container .info .tip {
  color: red;
}
.container .rest {
  color: red;
  font-size: 30px;
}

.main-bar {
  margin-top: 70px;
  position: relative;
}

.main-bar .gunzi {
  position: absolute;
  top: -40px;
  right: 80px;
  transform: rotate(45deg);
  transition: all 0.05s ease-in-out;
}

.main-bar .rotate {
  transform: rotate(-45deg);
}

.footer {
  position: fixed;
  bottom: 15px;
}
</style>
