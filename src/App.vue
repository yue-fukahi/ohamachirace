<template>
  <v-container class="race-container">
    <!-- 馬の表示 -->
    <v-row v-for="horse in horses" :key="horse.id" class="horse-row">
      <v-img
        :src="horse.image"
        class="horse"
        :style="{ transform: `translateX(${horse.position}px)` }"
      />
    </v-row>

    <!-- ゴールライン -->
    <div class="goal-line"></div>

    <!-- レース開始ボタン -->
    <v-btn class="start-button" @click="startRace" :disabled="raceInProgress">
      Start Race
    </v-btn>

    <!-- 結果表示 -->
    <v-alert v-if="raceFinished" type="success" class="result-alert">
      🏆 優勝: {{ rankings[0]?.name }} 🎉
    </v-alert>
  </v-container>
</template>

<script setup lang="ts">
import { ref } from 'vue';

interface Horse {
  id: number;
  name: string;
  image: string;
  position: number;
  finished: boolean;
}

const goalPosition = 500; // ゴールラインの位置
const raceInProgress = ref(false);
const raceFinished = ref(false);
const rankings = ref<Horse[]>([]);

const horses = ref<Horse[]>([
  { id: 1, name: 'サクラ', image: '/horse1.png', position: 0, finished: false },
  { id: 2, name: 'ウィニング', image: '/horse2.png', position: 0, finished: false },
  { id: 3, name: 'スペシャル', image: '/horse3.png', position: 0, finished: false },
]);

const startRace = () => {
  raceInProgress.value = true;
  raceFinished.value = false;
  rankings.value = [];
  horses.value.forEach((horse) => {
    horse.position = 0;
    horse.finished = false;
  });

  const raceInterval = setInterval(() => {
    let allFinished = true;

    horses.value = horses.value.map((horse) => {
      if (horse.finished) return horse; // 既にゴールした馬は動かさない

      let newPosition = horse.position + Math.random() * 10;
      if (newPosition >= goalPosition) {
        newPosition = goalPosition; // ⬅️ はみ出さないよう制限
        horse.finished = true;
        rankings.value.push(horse); // 順位を記録
      } else {
        allFinished = false;
      }

      return { ...horse, position: newPosition };
    });

    if (allFinished) {
      clearInterval(raceInterval);
      raceInProgress.value = false;
      raceFinished.value = true;
    }
  }, 100);
};
</script>

<style scoped>
.race-container {
  position: relative;
  width: 600px;
  height: 300px;
  border: 2px solid black;
  overflow: hidden;
  background-color: green;
}

.horse-row {
  position: relative;
  height: 60px;
  width: 60px;
}

.horse {
  width: 60px;
  transition: transform 0.1s linear;
}

/* ゴールライン */
.goal-line {
  position: absolute;
  top: 0;
  right: 100px; /* ゴール地点の調整 */
  width: 5px;
  height: 100%;
  background-color: red;
}

/* レース開始ボタン */
.start-button {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
}

/* 結果表示 */
.result-alert {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  width: 250px;
  text-align: center;
}
</style>
