<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height text-h4 text-teal-darken-4">
        OHAMACHIKO Kasakasa Derby
      </v-container>

      <v-btn v-if="!raceFinished" @click="startRace" :disabled="raceInProgress"> Start Race </v-btn>

      <v-btn v-if="raceFinished" @click="resetRace"> Reset Race </v-btn>
    </v-app-bar>

    <v-main class="bg-green-lighten-3">
      <v-container>
        <v-container class="rounded-lg bg-green position-relative">
          <div class="goal-line"></div>
          <v-row v-for="horse in horses" :key="horse.id">
            <v-col cols="2">
              <div class="horse-name">{{ horse.name }}</div>
              <div>{{ (horse.position / 5).toFixed(1) }}%</div>
              <div>{{ horse.isFallen }}</div>
            </v-col>
            <v-col>
              <v-img
                :src="horse.image"
                class="horse"
                :style="{
                  left: `${horse.position / 5}%`,
                  transform: `translateX(-${horse.position / 5}%) rotate(${horse.isFallen ? '90' : '0'}deg)`,
                }"
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

<script lang="ts">
import { defineComponent, ref } from 'vue'

interface Horse {
  id: number
  name: string
  image: string
  position: number
  isFallen: boolean
  fallCooldown: boolean
  finished: boolean
}

export default defineComponent({
  setup() {
    const rankings = ref<Horse[]>([])
    const horses = ref<Horse[]>([
      {
        id: 1,
        name: 'ダンゴムシチャウヨ',
        image: '/horse1.png',
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      },
      {
        id: 2,
        name: 'ウマウマチャハーン',
        image: '/horse2.png',
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      },
      {
        id: 3,
        name: 'イチイチイチイチ',
        image: '/horse3.png',
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      },
      {
        id: 4,
        name: 'キラキラキンヨービ',
        image: '/horse4.png',
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      },
      {
        id: 5,
        name: 'ディーモアンター',
        image: '/horse5.png',
        position: 0,
        isFallen: false,
        fallCooldown: false,
        finished: false,
      },
    ])

    const goalPosition = 500
    const raceInProgress = ref(false)
    const raceFinished = ref(false)

    const startRace = () => {
      raceInProgress.value = true
      raceFinished.value = false

      const raceInterval = setInterval(() => {
        let allFinished = true

        horses.value.forEach((horse) => {
          if (horse.finished) {
            return horse
          }

          if (horse.isFallen) {
            allFinished = false
            return horse
          }

          if (Math.random() < 0.02) {
            horse.isFallen = true
            setTimeout(() => {
              horse.isFallen = false
            }, 800)
            return horse
          }

          let newPosition = horse.position + Math.random() * 10
          if (newPosition >= goalPosition) {
            newPosition = goalPosition
            horse.finished = true
            rankings.value.push(horse)
          }

          horse.position = newPosition
          allFinished = false

          return horse
        })

        if (allFinished) {
          clearInterval(raceInterval)
          raceFinished.value = true
          raceInProgress.value = false
        }
      }, 100)
    }

    const resetRace = () => {
      horses.value.forEach((horse) => {
        horse.position = 0
        horse.isFallen = false
        horse.finished = false
      })
      raceFinished.value = false
      rankings.value = []
    }

    return {
      horses,
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
