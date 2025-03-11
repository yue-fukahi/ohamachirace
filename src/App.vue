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
                  left: `${positionRate(state)}%`,
                  transform: `translateX(-${positionRate(state)}%) rotate(${state.isFallen ? '90' : '0'}deg)`,
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
        <v-btn v-if="!raceFinished" @click="shuffleHorses" :disabled="raceInProgress">
          Shuffle
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

const getLuckStars = (luck: Luck): string => {
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
  { id: 1, name: 'マイニチオハマチコ', image: `${import.meta.env.BASE_URL}/horse01.png`, luck: Luck.NORMAL },
  { id: 2, name: 'ダンゴムシチャウヨ', image: `${import.meta.env.BASE_URL}/horse02.png`, luck: Luck.NORMAL },
  { id: 3, name: 'ディーモアンター', image: `${import.meta.env.BASE_URL}/horse03.png`, luck: Luck.NORMAL },
  { id: 4, name: 'ハシレバカロリゼロ', image: `${import.meta.env.BASE_URL}/horse04.png`, luck: Luck.NORMAL },
  { id: 5, name: 'タマヤーチャンス', image: `${import.meta.env.BASE_URL}/horse05.png`, luck: Luck.NORMAL },
  { id: 6, name: 'イノチノミズ', image: `${import.meta.env.BASE_URL}/horse06.png`, luck: Luck.NORMAL },
  { id: 7, name: 'ウマウマチャハーン', image: `${import.meta.env.BASE_URL}/horse07.png`, luck: Luck.NORMAL },
  { id: 8, name: 'ヤマタノオロチン', image: `${import.meta.env.BASE_URL}/horse08.png`, luck: Luck.GOOD },
  { id: 9, name: 'コノヨーニネ', image: `${import.meta.env.BASE_URL}/horse09.png`, luck: Luck.GOOD },
  { id: 10, name: 'キラキラキンヨービ', image: `${import.meta.env.BASE_URL}/horse10.png`, luck: Luck.VERY_GOOD },
  { id: 11, name: 'ダマーレーサンシタ', image: `${import.meta.env.BASE_URL}/horse11.png`, luck: Luck.BAD },
  { id: 12, name: 'メイク・デ・ブー', image: `${import.meta.env.BASE_URL}/horse12.png`, luck: Luck.BAD },
  { id: 13, name: 'ゲーミンロッピャク', image: `${import.meta.env.BASE_URL}/horse13.png`, luck: Luck.VERY_BAD },
  { id: 14, name: 'チェレン・コフコ', image: `${import.meta.env.BASE_URL}/horse14.png`, luck: Luck.NORMAL },
  { id: 15, name: 'マチコムシノバシ', image: `${import.meta.env.BASE_URL}/horse15.png`, luck: Luck.GOOD },
  { id: 16, name: 'シッキンレッド', image: `${import.meta.env.BASE_URL}/horse16.png`, luck: Luck.NORMAL },
  { id: 17, name: 'カキンブラック', image: `${import.meta.env.BASE_URL}/horse17.png`, luck: Luck.NORMAL },
  { id: 18, name: 'コーケンブルー', image: `${import.meta.env.BASE_URL}/horse18.png`, luck: Luck.NORMAL },
  { id: 19, name: 'モーシンイエロー', image: `${import.meta.env.BASE_URL}/horse19.png`, luck: Luck.NORMAL },
  { id: 20, name: 'バクチピンク', image: `${import.meta.env.BASE_URL}/horse20.png`, luck: Luck.NORMAL },
  { id: 21, name: 'マチコジルゴクゴク', image: `${import.meta.env.BASE_URL}/horse21.png`, luck: Luck.GOOD },
  { id: 22, name: 'ナニシテール・モー', image: `${import.meta.env.BASE_URL}/horse22.png`, luck: Luck.VERY_BAD },
  { id: 23, name: 'ヒダルマーチ', image: `${import.meta.env.BASE_URL}/horse23.png`, luck: Luck.BAD },
  { id: 24, name: 'ハッピーポリープ', image: `${import.meta.env.BASE_URL}/horse24.png`, luck: Luck.VERY_GOOD },
  { id: 25, name: 'アサオキレーヌ', image: `${import.meta.env.BASE_URL}/horse25.png`, luck: Luck.BAD },
  { id: 26, name: 'インシーシューカイ', image: `${import.meta.env.BASE_URL}/horse26.png`, luck: Luck.NORMAL },
]

const updateWeight = 100
const speedWeight = 6
const goalPosition = speedWeight * 100

const rankings = ref<HorseState[]>([])
const horseStates = ref<HorseState[]>([])
const raceInProgress = ref(false)
const raceFinished = ref(false)
const dialogVisible = ref(false)

onMounted(() => {
  shuffleHorses()
})

const positionRate = (state: HorseState) => state.position / speedWeight

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

const speed: Record<Luck, number> = {
  [Luck.VERY_BAD]: 6.6,
  [Luck.BAD]: 6.9,
  [Luck.NORMAL]: 7.0,
  [Luck.GOOD]: 7.1,
  [Luck.VERY_GOOD]: 8.0,
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

      const ending = positionRate(state) > 90
      const doFall = !ending && Math.random() < fallChances[state.luck]
      const progress = !ending ? Math.random() * speed[state.luck] : 0.5 * 7

      if (doFall) {
        state.isFallen = true
        setTimeout(() => {
          state.isFallen = false
        }, 8 * updateWeight)
      }

      let newPosition = state.position + progress
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
  }, 1 * updateWeight)
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
