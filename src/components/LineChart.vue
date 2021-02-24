<template>
    <div>
        <apexchart type="line" height="350px" :options="chartOptions" :series="series"></apexchart>
    </div>
</template>
<script>
export default {
  props: {
    percentage: {
      type: Array,
      default: () => []
    },
    forseen: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      series: [{
        name: 'Actual Vaccinated',
        data: this.percentage
      }, {
        name: 'Expected Vaccinated',
        data: this.forseen
      }]
    }
  },
  computed: {
    chartOptions () {
      return {
        chart: {
          height: 600,
          type: 'line',
          id: 'areachart-2'
        },
        annotations: {
          yaxis: [{
            y: 60,
            y2: 70,
            borderColor: '#00E396',
            fillColor: '#00E396',
            opacity: 0.2,
            label: {
              borderColor: '#775DD0',
              style: {
                color: '#fff',
                background: '#775DD0'
              },
              text: 'Herd Immunity'
            }
          }]
          // points: [{
          //   x: '30/09/2021',
          //   y: 100,
          //   marker: {
          //     size: 8,
          //     fillColor: '#fff',
          //     strokeColor: 'red',
          //     radius: 2,
          //     cssClass: 'apexcharts-custom-class'
          //   },
          //   label: {
          //     borderColor: '#FF4560',
          //     offsetY: 0,
          //     style: {
          //       color: '#fff',
          //       background: '#FF4560'
          //     },

          //     text: 'All Vaccinated'
          //   }
          // }]
        },
        dataLabels: {
          enabled: true
        },
        stroke: {
          curve: 'smooth'
        },
        grid: {
          padding: {
            right: 30,
            left: 20
          }
        },
        labels: this.percentage.map(obj => obj.x),
        xaxis: {
          categories: this.percentage.map(obj => obj.x)
        }
      }
    }
  }
}
</script>
