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
              <div>運値: {{ getLuckStars(state.luck) }}</div>
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

enum Luck {
  VERY_BAD = 1,
  BAD = 2,
  NORMAL = 3,
  GOOD = 4,
  VERY_GOOD = 5,
}

const getLuckStars = (luck: Luck) => {
  switch (luck) {
    case Luck.VERY_GOOD:
      return '👼'
    case Luck.GOOD:
      return '🐰🐰🐰'
    case Luck.NORMAL:
      return '🐰🐰'
    case Luck.BAD:
      return '🐰'
    case Luck.VERY_BAD:
      return '💩'
  }
}

interface Horse {
  id: number
  name: string
  image: string
  luck: Luck
  speed: number
}

interface HorseState extends Horse {
  postPosition: number
  position: number
  isFallen: boolean
  fallCooldown: boolean
  finished: boolean
}

// prettier-ignore
const horses: Horse[] = [
  { id: 0, name: 'ダンゴムシチャウヨ', image: '/horse1.png', luck: Luck.NORMAL, speed: 7.0 },
  { id: 1, name: 'ウマウマチャハーン', image: '/horse2.png', luck: Luck.GOOD, speed: 6.9 },
  { id: 2, name: 'ハシレバカロリゼロ', image: '/horse3.png', luck: Luck.NORMAL, speed: 7.0 },
  { id: 3, name: 'キラキラキンヨービ', image: '/horse4.png', luck: Luck.VERY_GOOD, speed: 6.6 },
  { id: 4, name: 'ディーモアンター', image: '/horse5.png', luck: Luck.NORMAL, speed: 7.0 },
  { id: 5, name: 'ニテンノート', image: '/horse5.png', luck: Luck.BAD, speed: 7.1 },
  { id: 6, name: 'ダマーレーサンシタ', image: '/horse5.png', luck: Luck.BAD, speed: 7.1 },
  { id: 7, name: 'ゲーミンロッピャク', image: '/horse5.png', luck: Luck.VERY_BAD, speed: 8.0 },
  { id: 8, name: 'マイニチオハマチコ', image: '/horse5.png', luck: Luck.NORMAL, speed: 7.0 },
  { id: 9, name: 'ヤマタノオロチン', image: '/horse5.png', luck: Luck.GOOD, speed: 6.9 },
]

const gameWeight = 100
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

const fallChances: Record<Luck, number> = {
  [Luck.VERY_BAD]: 0.05,
  [Luck.BAD]: 0.025,
  [Luck.NORMAL]: 0.02,
  [Luck.GOOD]: 0.015,
  [Luck.VERY_GOOD]: 0,
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

      if (Math.random() < fallChances[state.luck]) {
        state.isFallen = true
        setTimeout(() => {
          state.isFallen = false
        }, 8 * gameWeight)
      }

      let newPosition = state.position + Math.random() * state.speed
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
  }, 1 * gameWeight)
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
  right: 60px;
  width: 5px;
  height: 100%;
  background-color: white;
}
</style>
