<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height text-h4 text-teal-darken-4">
        OHAMACHIKO Kasakasa Derby
      </v-container>

      <v-btn v-if="!raceFinished" @click="startRace" :disabled="raceInProgress">
        Start Race
      </v-btn>

      <v-btn v-if="raceFinished" @click="resetRace">
        Reset Race
      </v-btn>
    </v-app-bar>

    <v-main class="bg-green-lighten-3">
      <v-container>
        <v-container class="border rounded-lg bg-green position-relative">
          <div class="goal-line"></div>
          <v-row v-for="horse in horses" :key="horse.id">
            <v-col cols="2">
              <div class="horse-name">{{ horse.name }}</div>
              <div>{{ (horse.position / 5).toFixed(1) }}%</div>
            </v-col>
            <v-col>
              <v-img
                :src="horse.image"
                class="horse"
                :style="{ left: `${horse.position/5}%`, transform: `translateX(-${horse.position/5}%)` }"
              />
            </v-col>
          </v-row>
        </v-container>
      </v-container>

      <v-alert v-if="raceFinished" type="success" class="result-alert">
        🏆 優勝: {{ rankings[0]?.name }} 🎉
      </v-alert>
    </v-main>
  </v-app>
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
  { id: 1, name: 'ダンゴムシチャウヨ', image: '/horse1.png', position: 0, finished: false },
  { id: 2, name: 'ウマウマチャハーン', image: '/horse2.png', position: 0, finished: false },
  { id: 3, name: 'イチイチイチイチ', image: '/horse3.png', position: 0, finished: false },
  { id: 4, name: 'キラキラキンヨービ', image: '/horse4.png', position: 0, finished: false },
  { id: 5, name: 'ディーモアンター', image: '/horse5.png', position: 0, finished: false },
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

const resetRace = () => {
  raceFinished.value = false;
  rankings.value = [];
  horses.value.forEach((horse) => {
    horse.position = 0;
    horse.finished = false;
  });
};
</script>


<style scoped>
.race-container {
  position: relative;
  max-width: 1200px;
  height: 300px;
  border: 2px solid black;
  overflow: hidden;
  background-color: green;
}

.horse {
  transition: left 0.1s linear;
  width: 60px;
}

.goal-line {
  position: absolute;
  top: 0%;
  right: 70px;
  width: 5px;
  height: 100%;
  background-color: white;
}

/* 結果表示 */
.result-alert {
  text-align: center;
  font-size: 18px;
  margin-top: 10px;
}
</style>
