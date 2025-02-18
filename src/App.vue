<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height text-h4 text-teal-darken-4">
        OHAMACHIKO Kasakasa Derby
      </v-container>
    </v-app-bar>

    <v-main class="bg-green-lighten-3">
      <v-container>
        <v-container class="rounded-lg bg-green position-relative">
          <div class="goal-line"></div>
          <v-row v-for="state in horseStates" :key="state.id">
            <v-col cols="2">
              <div class="horse-name">{{ state.postPosition }} {{ state.name }}</div>
              <div>{{ (state.position / 5).toFixed(1) }}%</div>
            </v-col>
            <v-col>
              <v-img
                :src="state.image"
                class="horse"
                :style="{
                  left: `${state.position / 5}%`,
                  transform: `translateX(-${state.position / 5}%) rotate(${state.isFallen ? '90' : '0'}deg)`,
                }"
              />
            </v-col>
          </v-row>
        </v-container>
      </v-container>

      <v-container class="text-center">
        <v-btn color="primary" v-if="!raceFinished" @click="startRace" :disabled="raceInProgress">
          Start Race
        </v-btn>
        <v-btn v-if="raceFinished" @click="resetRace"> Reset Race </v-btn>
      </v-container>

      <v-dialog v-model="dialogVisible" max-width="400px">
        <v-card>
          <template v-slot:title>
            <div class="d-flex justify-center">🏆 レース結果 🏆</div>
          </template>
          <v-card-text>
            <div v-if="rankings.length > 2">
              <p>🥇 1位: {{ rankings[0].postPosition }} {{ rankings[0].name }}</p>
              <p>🥈 2位: {{ rankings[1].postPosition }} {{ rankings[1].name }}</p>
              <p>🥉 3位: {{ rankings[2].postPosition }} {{ rankings[2].name }}</p>
            </div>
            <div v-else>
              <p>レース結果がありません</p>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" @click="dialogVisible = false">OK</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Horse {
  id: number
  name: string
  image: string
}

interface HorseState extends Horse {
  postPosition: number
  position: number
  isFallen: boolean
  fallCooldown: boolean
  finished: boolean
}

const horses: Horse[] = [
  { id: 0, name: 'ダンゴムシチャウヨ', image: '/horse1.png' },
  { id: 1, name: 'ウマウマチャハーン', image: '/horse2.png' },
  { id: 2, name: 'ハシレバカロリゼロ', image: '/horse3.png' },
  { id: 3, name: 'キラキラキンヨービ', image: '/horse4.png' },
  { id: 4, name: 'ディーモアンター', image: '/horse5.png' },
  { id: 5, name: 'ニテンノート', image: '/horse5.png' },
  { id: 6, name: 'ダマーレーサンシタ', image: '/horse5.png' },
  { id: 7, name: 'ゲーミンロッピャク', image: '/horse5.png' },
  { id: 8, name: 'マイニチオハマチコ', image: '/horse5.png' },
  { id: 9, name: 'ヤマタノオロチン', image: '/horse5.png' },
]
const goalPosition = 500

const rankings = ref<HorseState[]>([])
const horseStates = ref<HorseState[]>([])
const raceInProgress = ref(false)
const raceFinished = ref(false)
const dialogVisible = ref(false)

onMounted(() => {
  shuffleHorses()
})

const shuffleHorses = () => {
  horseStates.value = [...horses]
    .sort(() => Math.random() - 0.5)
    .slice(0, 7)
    .map((horse, i) => ({
      ...horse,
      postPosition: i + 1,
      position: 0,
      isFallen: false,
      fallCooldown: false,
      finished: false,
    }))
}

const startRace = () => {
  raceInProgress.value = true
  raceFinished.value = false
  dialogVisible.value = false

  const raceInterval = setInterval(() => {
    let allFinished = true

    horseStates.value.forEach((state) => {
      if (state.finished) {
        return state
      }

      if (state.isFallen) {
        allFinished = false
        return state
      }

      if (Math.random() < 0.01) {
        state.isFallen = true
        setTimeout(() => {
          state.isFallen = false
        }, 800)
        allFinished = false
        return state
      }

      let newPosition = state.position + Math.random() * 7
      if (newPosition >= goalPosition) {
        newPosition = goalPosition
        state.finished = true
        rankings.value.push(state)
      }

      state.position = newPosition
      allFinished = false

      return state
    })

    if (allFinished) {
      clearInterval(raceInterval)
      raceFinished.value = true
      raceInProgress.value = false
      dialogVisible.value = true
    }
  }, 100)
}

const resetRace = () => {
  horseStates.value.forEach((horse, i) => {
    id: horse.id = i
    horse.position = 0
    horse.isFallen = false
    horse.finished = false
    horse.fallCooldown = false
  })
  raceFinished.value = false
  dialogVisible.value = false
  rankings.value = []
}
</script>

<style scoped>
.horse {
  transition:
    left 0.3s ease-out,
    transform 0.1s ease-out;
  width: 50px;
}

.goal-line {
  position: absolute;
  top: 0%;
  right: 75px;
  width: 5px;
  height: 100%;
  background-color: white;
}
</style>
