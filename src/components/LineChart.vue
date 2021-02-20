<template>
    <div>
        <apexchart type="line" height="350" :options="chartOptions" :series="series"></apexchart>
    </div>
</template>
<script>
export default {
  props: {
    percentage: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      series: [{
        name: 'percentage_vaccinated',
        data: this.percentage
      }],
      chartOptions: {
        chart: {
          height: 350,
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
          }],
          points: [{
            x: new Date('30 Sep 2021').getTime(),
            y: 100,
            marker: {
              size: 8,
              fillColor: '#fff',
              strokeColor: 'red',
              radius: 2,
              cssClass: 'apexcharts-custom-class'
            },
            label: {
              borderColor: '#FF4560',
              offsetY: 0,
              style: {
                color: '#fff',
                background: '#FF4560'
              },

              text: 'All Vaccinated'
            }
          }]
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
        labels: this.percentage.map(obj => obj.date),
        xaxis: {
          categories: this.percentage.map(obj => obj.date)
        }
      }
    }
  },
  methods: {
    forseen () {
      return {
        x: new Date('30 Sep 2021').getTime(),
        y: 100
      }
    }
  }
}
</script>
