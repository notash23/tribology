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
  <div class="main">
    <section>
        <h1>Lubrication Theory</h1>
        <p>Welcome to more simulator of liquid in a box! Below you can find a video of the simulation done on python as a video.</p>
        <Card class="plot">
            <template #content>
                <video autoplay muted loop >
                    <source src="./assets/animation.mp4" type="video/mp4">\
                    Your browser does not support the video tag.
                </video>
            </template>
        </Card>
    </section>
    

    <section>
        <p>This is a simulator of the same algorithm running on this website. Play around and see what you find!</p>
        <div class="slider-horizontal">
            <h3>L</h3>
            <Slider v-model="height" style="width: 100%;"/>
            <h3>{{ height }}</h3>
        </div>
        
        <div class="flex-horizontal">
            <div class="slider-vertical">
            <h3>U</h3>
            <Slider class="" v-model="speed" orientation="vertical"/>
            <h3>{{ speed }}</h3>
            </div>
            
            <Card class="plot">
            <template #content>
                <Chart type="scatter" :data="chartData" :options="chartOptions" />
            </template>
            </Card>
            
            
            <div class="slider-vertical">
            <h3>P</h3>
            <Slider v-model="pressure_grad" orientation="vertical"/>
            <h3> {{ pressure_grad }}</h3>
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
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  color: #91939c;
  margin-top: 20pt;
  margin-left: 20pt;
  align-self:baseline;
}

p, h3 {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 10pt;
}

.plot {
  width: 90%;
    max-width: calc(100vh - 80px);
  overflow: hidden;
}

.plot * {
    width: 100%;
}

.slider-vertical {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  width: 62px;
  max-height: calc(100% - 40px);

}
.slider-vertical > div{
  flex:1;
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

.flex-horizontal .plot{
  flex:1;
}
</style>