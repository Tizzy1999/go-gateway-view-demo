<template>
  <div class="chart-container">
    <chart height="100%" width="100%" :data="chartData" />
  </div>
</template>

<script>
import Chart from './components/Charts/LineStat'
import { serviceDetail, serviceStat } from '@/api/service'

export default {
  name: 'Statistics',
  components: { Chart },
  data() {
    return {
      chartData: {
        'title': '',
        'today': [],
        'yesterday': []
      }
    }
  },
  created() {
    const id = this.$route.params && this.$route.params.id
    this.fetchStat(id)
  },
  methods: {
    fetchStat(id) {
      const query = { 'id': id }
      serviceStat(query).then(stat => {
        serviceDetail(query).then(detail => {
          console.log(stat.data)
          console.log(detail.data)
          this.chartData = {
            'title': detail.data.info.service_name + ' statistics',
            'today': stat.data.today,
            'yesterday': stat.data.yesterday
          }
        }).catch((err) => {
          console.log('fetch detail error: ' + err)
        })
      }).catch((err) => {
        console.log('fetch statistics error: ' + err)
      })
    }
  }
}
</script>

<style scoped>
.chart-container{
  position: relative;
  width: 100%;
  height: calc(100vh - 84px);
}
</style>

