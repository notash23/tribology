// TODO: use asynchronous functions for calling 'step' and timing them right // TODO: use scrollbar
as parameters in step // TODO: design better arrows (and make them resizable) //TODO: make a reload button

<script setup lang="ts">
import { onUnmounted, ref } from 'vue'
import Chart from 'primevue/chart'
import { Chart as ChartType } from 'chart.js/auto';
import getWasm from './assets/main.wasm?init'

interface Point {
  x: number
  y: number
}

const items = 4
const x_steps = 21
const y_steps = 21

const height = ref(50)
const speed = ref(50)
const pressure_grad = ref(50)

const myChart = ref<InstanceType<typeof Chart>>(null!)
let chart: ChartType

let setupArray: (size: number) => void
let step: (uvpt: number) => void
let free: (size: number) => void
let ptr: number
let memory = new WebAssembly.Memory({
  initial: 256,
  maximum: 512,
})

function set_angles(arr: Float64Array) {
  const angles: Array<number> = []
  for (let i = 0; i < x_steps * y_steps; i++) {
    angles.push(arr[3 * (x_steps * y_steps) + i] ?? 0)
  }
  if (chart.data.datasets[0]){
    chart.data.datasets[0].pointRotation = angles
  } 
  chart.update('active')
  console.log(performance.now());
  console.log(Date.now());
}

getWasm({
  js: {
    mem: memory,
  },
  env: {
    _abort_js: () => {
      throw new WebAssembly.RuntimeError('Native code aborted')
    },
    emscripten_resize_heap: memory.grow,
  },
  wasi_snapshot_preview1: {
    fd_write: () => {
      throw new WebAssembly.RuntimeError('fd_write not implemented')
    },
    fd_close: () => {
      throw new WebAssembly.RuntimeError('fd_close not implemented')
    },
    fd_seek: () => {
      throw new WebAssembly.RuntimeError('fd_seek not implemented')
    },
  },
}).then((instance) => {
  memory = (instance.exports.memory ?? memory) as WebAssembly.Memory
  ptr = (instance.exports._malloc as (size: number) => number)(items * x_steps * y_steps * 8)

  const arr = new Float64Array(memory.buffer, ptr);

  
  setupArray = (instance.exports.setupArray ?? Function) as (size: number) => void
  step = (instance.exports.step ?? Function) as (uvpt: number) => void
  free = (instance.exports._free ?? Function) as (size: number) => void

  setupArray(ptr)
  let count = 10
  const func = () => {
    step(ptr)
    set_angles(arr)
    count--
    if (count > 0) {
      setTimeout(func, 300)
    }
  }
  setTimeout(func, 500)
})

const onChartLoaded = () => {
  chart = myChart.value?.getChart()
  chart.options = setChartOptions()
  chart.data = setChartData()
  chart.update()
}

onUnmounted(() => {
  free(ptr)
})

const points: Point[] = []
for (let j = 0; j < 21; j++) {
  for (let i = 0; i < 21; i++) {
    points.push({
      x: i,
      y: j,
    })
  }
}

const setChartData = () => {
  return {
    datasets: [
      {
        data: points,
        pointStyle: canvas,
        pointRadius: 2,
        backgroundColor: 'rgb(255, 99, 132)',
      },
    ],
  }
}

const setChartOptions = () => {
  return {
    maintainAspectRatio: false,
    scales: {
      x: {
        ticks: {
          display: false,
        },
      },
      y: {
        ticks: {
          display: false,
        },
      },
    },
    plugins: {
      legend: {
        display: false,
      },
      tooltip: {
        enabled: false,
      },
      title: {
        display: true,
        text: 'Simulation',
        padding: {
          bottom: 15,
        },
        font: {
          family: 'Roboto',
          size: 16,
        },
      },
    },
  }
}

const canvas = document.createElement('canvas')
canvas.width = 30
canvas.height = 30

const c = canvas.getContext('2d')!
c.translate(canvas.width / 2, canvas.height / 2)

const size = 10

c.beginPath()
c.moveTo(0, -size)
c.lineTo(size, size)
c.lineTo(0, size / 2)
c.lineTo(-size, size)
c.closePath()

c.fillStyle = 'red'
c.fill()
c.strokeStyle = 'red'
c.stroke()
</script>

<template>
  <div class="main">
    <section>
      <h1>Lubrication Theory</h1>
      <p>
        Welcome to more simulator of liquid in a box! Below you can find a video of the simulation
        done on python as a video.
      </p>
      <Card class="plot">
        <template #content>
          <video autoplay muted loop>
            <source src="./assets/animation.mp4" type="video/mp4" />
            \ Your browser does not support the video tag.
          </video>
        </template>
      </Card>
    </section>

    <section>
      <p>
        This is a simulator of the same algorithm running on this website. Play around and see what
        you find!
      </p>
      <div class="slider-horizontal">
        <h3>L</h3>
        <Slider v-model="height" style="width: 100%" />
        <h3>{{ height }}</h3>
      </div>

      <div class="flex-horizontal">
        <div class="slider-vertical">
          <h3>U</h3>
          <Slider class="" v-model="speed" orientation="vertical" />
          <h3>{{ speed }}</h3>
        </div>

        <Card class="plot chart">
          <template #content>
            <Chart ref="myChart" type="scatter" @loaded="onChartLoaded" />
          </template>
        </Card>

        <div class="slider-vertical">
          <h3>P</h3>
          <Slider v-model="pressure_grad" orientation="vertical" />
          <h3>{{ pressure_grad }}</h3>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
body {
  margin: 0px;
}

.main {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  overflow-y: hidden;
  gap: 80px;
  padding-bottom: 30px;
}

section {
  display: flex;
  flex-direction: column;
  align-items: center;
}

h1 {
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
  color: #91939c;
  margin-top: 20pt;
  margin-left: 20pt;
  align-self: baseline;
}

p,
h3 {
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
  margin: 10pt;
}

.plot {
  width: 90%;
  max-width: calc(100vh - 80px);
}

.chart {
  aspect-ratio: 1;
}

.plot * {
  width: 100%;
  height: 100%;
}

.slider-vertical {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  width: 62px;
  max-height: calc(100% - 40px);
}
.slider-vertical > div {
  flex: 1;
}

.slider-horizontal {
  /* width: calc(100% - 100px); */
  max-width: calc(100vh);
  width: 100%;
  padding: 0 60px;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  /* margin-bottom: 15pt; */
}

.flex-horizontal {
  display: flex;
  justify-content: center;
  /* align-items: center; */
  width: 100%;
}

.flex-horizontal .plot {
  flex: 1;
}
</style>
