<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Chart from 'primevue/chart';

const height = ref(50)
const speed = ref(50)
const pressure_grad = ref(50)

const chartData = ref();
const chartOptions = ref();

const points = ref<object[]>([]);
const rotation = ref<number[]>([]);

for (let i = 0; i < 21; i++) {
  for (let j = 0; j < 21; j++) {
    points.value.push({
      x: i,
      y: j
    });
    rotation.value.push((i+j)*5)
  }
}

onMounted(() => {
    chartData.value = setChartData();
    chartOptions.value = setChartOptions();
});

const setChartData = () => {
  return {
    datasets: [{
      data: points,
      pointStyle: 'triangle',
      pointRadius: 3,
      pointRotation: rotation,
      backgroundColor: 'rgb(255, 99, 132)'
    }]
  }
}

const setChartOptions = () => {
  return {
    aspectRatio: 1,
    scales: {
      x: {
        ticks: {
          display: false
        }
      },
      y: {
        ticks: {
          display: false
        }
      }
    },
    plugins: {
      legend: {
        display: false
      },
      tooltip: {
        enabled: false
      },
      title: {
          display: true,
          text: 'Simulation',
          padding: {
              bottom: 15
          },
          font: {
            family: 'Roboto',
            size: 16,
            weight: 'bold'
          }
      },
    }
  }
}
</script>

<template>
  <h1>Lubrication Theory</h1>
  <p>This is a simulator of Navier Stokes with the top of the lid moving at the speed specified. Play around and see what you find!</p>

  <div class="slider-horizontal">
    <h3>Height</h3>
    <Slider v-model="height" style="width: 75%;"/>
    <h3>{{ height }}</h3>
  </div>
  
  <div class="flex-horizontal">
    <div class="slider-vertical">
      <h3>Speed</h3>
      <Slider class="" v-model="speed" orientation="vertical"/>
      <h3>{{ speed }}</h3>
    </div>
    
    <Card class="plot">
      <template #content>
        <Chart type="scatter" :data="chartData" :options="chartOptions" />
      </template>
    </Card>
    
    
    <div class="slider-vertical">
      <h3>Pressure Gradient</h3>
      <Slider v-model="pressure_grad" orientation="vertical"/>
      <h3> {{ pressure_grad }}</h3>
    </div>
  </div>
</template>

<style scoped>
h1 {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  color: #91939c;
  margin-top: 20pt;
  margin-left: 20pt;
}

p, h3 {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 10pt;
}

.plot {
  width: 75%;
  margin: auto;
}

.slider-vertical {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  
}
.slider-vertical > Slider {
  width: 800px;
}

.slider-horizontal {
  width: 100%;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  margin-bottom: 15pt;
}

.flex-horizontal {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
}
</style>
