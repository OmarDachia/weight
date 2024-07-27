<script setup>
  import {ref, shallowRef, computed, watch, nextTick} from 'vue'
  import Chart from 'chart.js/auto';

  const weights = ref([])
  const weightChartEl = ref(null)
  const weightChart = shallowRef(null)
  const weightInput = ref(60.0)

  const currentweight = computed(()=>{
    return weights.value.sort((a,b)=>b.date - a.date)[0] || {weights: 0}
  })

  const addWeight = () =>{
    weights.value.push({
      weight:weightInput.value,
      date: new Date().getDate()
    })
  }

  watch(weights, newWeights => {
    const ws = [...newWeights]
    if(weightChart.value){
      weightChart.value.data.labels = ws
            .sort((a,b)=> a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString())
            .slice(-7)
      
      weightChart.value.data.datasets[0].data = ws
            .sort((a,b)=> a.date - b.date)
            .map(w => w.weight)
            .slice(-7)

        weightChart.value.update()
        return
    }


    nextTick(()=>{
      weightChart.value = new Chart(weightChartEl.value.getContext('2d'),{
        type:'line',
        data:{
          labels: ws
            .sort((a,b)=> a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString()),
          datasets: [
              {
                label: 'Weight',
                data: ws
                  .sort((a,b)=> a.date - b.date)
                  .map(w => w.weight),
                backgroundColor: 'rgba(255,105,180,0.2)',
                borderColor: 'rgb(255,105,180)',
                borderWidth:1,
                fill:true
              }
            ]
        },
        options:{
          responsive:true,
          maintainAspectRatio:false
        }

      })
    })
  },{deep:true})
</script>

<template>
    <main>

      <h1>Weight Tracker</h1>

      <div class="current">
        <span>{{ currentweight.weight }} </span>
        <small> Current weight (kg)</small>
      </div>

      <form @submit.prevent="addWeight">
        <input type="number" step="0.1" v-model="weightInput"/>

        <input type="submit" value="Add Weight"/>
      </form>

      <div v-if="weights && weights.length>0">
        <h2>Last 7 days</h2>
        <div class="canvas-box">
          <canvas ref="weightChartEl"></canvas>
        </div>

        <div class="weight-history">
          <h2>Weight History</h2>
          <ul>
            <li v-for="weigth in weights">
              <span>{{ weigth.weight }}kg</span>
              <small>{{ new Date(weigth.date).toLocaleDateString() }}</small>
            </li>
          </ul>
        </div>
      </div>
    </main>
</template>

<style scoped>

</style>
