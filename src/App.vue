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
          <v-row v-for="(state, index) in horseStates" :key="state.id">
            <v-col cols="2">
              <div class="horse-name">{{ index + 1 }} {{ horses[state.id].name }}</div>
              <div>{{ (state.position / 5).toFixed(1) }}%</div>
              <div>{{ state.isFallen }}</div>
            </v-col>
            <v-col>
              <v-img
                :src="horses[state.id].image"
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
        <v-btn color="primary" v-if="!raceFinished" @click="startRace" :disabled="raceInProgress"> Start Race </v-btn>
        <v-btn v-if="raceFinished" @click="resetRace"> Reset Race </v-btn>
      </v-container>

      <v-alert v-if="raceFinished" type="success" class="result-alert">
        🏆 優勝: {{ rankings[0]?.name }} 🎉
      </v-alert>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

interface Horse {
  id: number
  name: string
  image: string
}

interface HorseState {
  id: number
  position: number
  isFallen: boolean
  fallCooldown: boolean
  finished: boolean
}

export default defineComponent({
  setup() {
    const rankings = ref<Horse[]>([])
    const horses: Horse[] = [
      { id: 0, name: 'ダンゴムシチャウヨ', image: '/horse1.png' },
      { id: 1, name: 'ウマウマチャハーン', image: '/horse2.png' },
      { id: 2, name: 'イチイチイチイチ', image: '/horse3.png' },
      { id: 3, name: 'キラキラキンヨービ', image: '/horse4.png' },
      { id: 4, name: 'ディーモアンター', image: '/horse5.png' },
    ]
    const horseStates = ref<HorseState[]>(
      horses.map((horse) => ({
        id: horse.id,
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      })),
    )

    const goalPosition = 500
    const raceInProgress = ref(false)
    const raceFinished = ref(false)

    const startRace = () => {
      raceInProgress.value = true
      raceFinished.value = false

      const raceInterval = setInterval(() => {
        let allFinished = true

        horseStates.value.forEach((state, i) => {
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
            return state
          }

          let newPosition = state.position + Math.random() * 7
          if (newPosition >= goalPosition) {
            newPosition = goalPosition
            state.finished = true
            rankings.value.push(horses[i])
          }

          state.position = newPosition
          allFinished = false

          return state
        })

        if (allFinished) {
          clearInterval(raceInterval)
          raceFinished.value = true
          raceInProgress.value = false
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
      rankings.value = []
    }

    return {
      horses,
      horseStates,
      rankings,
      raceInProgress,
      raceFinished,
      startRace,
      resetRace,
    }
  },
})
</script>

<style scoped>
.horse {
  transition:
    left 0.3s ease-out,
    transform 0.1s ease-out;
  width: 60px;
}

.goal-line {
  position: absolute;
  top: 0%;
  right: 75px;
  width: 5px;
  height: 100%;
  background-color: white;
}

.result-alert {
  text-align: center;
  font-size: 18px;
  margin-top: 10px;
}
</style>
