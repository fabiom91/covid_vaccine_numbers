<template>
    <section>
        <b-navbar class="is-primary color-white">
            <template #start>
                <b-navbar-item>
                    <h1 class="title is-2">{{ title }}</h1>
                </b-navbar-item>
                <b-navbar-item>
                    <h3 class="subtitle is-3">{{subtitle}}</h3>
                </b-navbar-item>
            </template>
        </b-navbar>

        <b-tabs type="is-boxed" position="is-centered">
            <b-tab-item label="Data" icon="chart-arc">
                <div class="columns">
                    <div class="column field">Population</div>
                    <div class="column value">{{ population }}</div>
                </div>
                <div class="columns">
                    <div class="column field">Actual Vaccinated</div>
                    <div class="column value">
                        <p>{{ vaccinated }}</p>
                        <p><b>{{ percentage_vaccinated }}%</b> of total population</p>
                        <p> <i>Expected <b>Herd Immunity</b> (60%) reach by: </i><b>{{ predictedHerdImmunityDate }}</b></p>
                        <p> <i>Expected <b>100% Immunity</b> reach by: </i><b>{{ predictedEndDate }}</b></p>
                    </div>
                </div>
                <div class="columns">
                    <div class="column field"><p>Expected Vaccinated </p><p style="font-size: small"> <i>(To reach 100% vaccinated by 30th September 2021)</i></p></div>
                    <div class="column value">
                        <p>{{ Math.round((population * forseen[forseen.length - 1].y) / 100) }}</p>
                        <p><b>{{ forseen[forseen.length - 1].y }}%</b> of total population</p>
                        <p> <i>Expected <b>Herd Immunity</b> (60%) reach by: </i><b>{{ forseenHerdImmunityDate }}</b></p>
                    </div>
                </div>
                <hr />
                <div class="columns">
                  <div class="column">
                    <div v-if="avgIncrease - avgIncreaseExp < 0">
                      <p style="color: red">AVG. daily gap between Actual and Expected number of people vaccinated: <b>{{Math.round((avgIncrease - avgIncreaseExp)*100)/100}}</b></p>
                      <p> We would need to vaccinate approximately: {{Math.round(Math.abs(avgIncrease - avgIncreaseExp) * avgIncreaseNumber / 1000)}}K more people each day to be able to reach Herd Immunity (60% of population) by {{ forseenHerdImmunityDate }} and Total Immunisation (100% of population) by: 30/09/2021</p>
                    </div>
                    <div v-else>
                      <p style="color: green">AVG. daily gap between Actual and Expected number of people vaccinated: <b>{{avgIncrease - avgIncreaseExp}}</b></p>
                    </div>
                  </div>
                </div>
                <line-chart :percentage="formattedData" :forseen="forseen"/>
                <h2>Last Update: {{ last_update }}</h2>
            </b-tab-item>
            <b-tab-item label="Sources" icon="database">
                <div class="columns">
                    <div class="column">
                        <b>Total Population Data</b>
                        <p>The population number displayed in the "Data" page, comes from https://restcountries.eu/ API: https://restcountries.eu/rest/v2/name/ireland</p>
                    </div>
                    <div class="column">
                        <b>Number of people vaccinated</b>
                        <p> This data comes from the HSE covid tracker app: https://covidtracker.gov.ie/ and is manually updated daily. The number of vaccinated people refers to those
                          who have complete the full vaccination cycle which for some vaccines translate to having received 2 doses.
                        </p>
                    </div>
                </div>
            </b-tab-item>
            <b-tab-item label="GitHub" icon="github">
              <div class="columns">
                <div class="column">
                  <b-button class="is-dark mb-5" icon-left="github" @click="redirect_github()">Open GitHub repository</b-button>
                  <p><i>This is a completely open-source project that welcomes all kind of developers to fork it on GitHub, play with it, fix bugs and make adjustments at pleasure.
                    However, being also completely free and not monetized in any way, don't expect to find an API to access the data pulled from my database. Instead, you'll
                    find a copy of the entire database below for you to copy and use it as you like:</i></p>
                  <b-message class="mt-5">{{allNumbers}}</b-message>
                </div>
              </div>
            </b-tab-item>
        </b-tabs>
    </section>
</template>

<script>
import db from './firebaseInit'
import LineChart from './LineChart'
const axios = require('axios')

export default {
  components: {
    LineChart
  },
  data () {
    return {
      title: 'Ireland COVID-19 Vaccine Monitor',
      subtitle: 'An unofficial monitoring app',
      country_data: null,
      allNumbers: [],
      formattedData: [],
      population: null,
      last_update: null,
      vaccinated: null,
      percentage_vaccinated: null,
      forseen: [],
      forseenHerdImmunityDate: null,
      predictedEndDate: null,
      predictedHerdImmunityDate: null,
      avgIncrease: null,
      avgIncreaseExp: null,
      avgIncreaseNumber: null
    }
  },
  mounted () {
    this.get_data()
  },
  methods: {
    async get_data () {
      try {
        const response = await axios.get('https://restcountries.eu/rest/v2/name/ireland')
        this.country_data = response.data
        const snapshot = await db.collection('vaccine_numbers').get()
        snapshot.forEach(doc => {
          this.allNumbers.push(doc.data())
        })
        let sumVaccinated = 0
        for (const obj of this.allNumbers) {
          sumVaccinated = sumVaccinated + obj.vaccinated
          const percentage = Math.round((obj.vaccinated * 10000) / this.country_data[0].population) / 100
          const formatted = {
            x: new Date(obj.date.seconds * 1000).toLocaleDateString(),
            y: percentage
          }
          this.formattedData.push(formatted)
          const daysFrom20Feb = (obj.date.seconds - 1611705600) / (3600 * 24)
          const forseenPercentage = Math.round((daysFrom20Feb * 10000) / 223) / 100
          const formattedForseen = {
            x: new Date(obj.date.seconds * 1000).toLocaleDateString(),
            y: forseenPercentage
          }
          this.forseen.push(formattedForseen)
        }
        this.avgIncreaseNumber = Math.round(sumVaccinated / this.allNumbers.length)
        this.population = this.country_data[0].population
        this.allNumbers = this.allNumbers.sort((a, b) => a.date - b.date)
        this.formattedData = this.formattedData.sort((a, b) => a.x - b.x)
        this.last_update = this.formattedData[this.formattedData.length - 1].x
        this.vaccinated = this.allNumbers[this.allNumbers.length - 1].vaccinated
        this.percentage_vaccinated = this.formattedData[this.formattedData.length - 1].y

        this.forseenHerdImmunityDate = new Date(((60 * 223) / 100) * 86400000 + 1611705600000).toLocaleDateString()
        let increaseArray = []
        for (let i = 2; i < this.formattedData.length; i++) {
          const increase = this.formattedData[i].y - this.formattedData[i - 1].y
          increaseArray.push(increase)
        }
        const sum = increaseArray.reduce((previous, current) => current + previous, 0)
        this.avgIncrease = sum / increaseArray.length

        let expectedIncreaseArray = []
        for (let i = 2; i < this.forseen.length; i++) {
          const increase = this.forseen[i].y - this.forseen[i - 1].y
          expectedIncreaseArray.push(increase)
        }
        const sumExp = expectedIncreaseArray.reduce((previous, current) => current + previous, 0)
        this.avgIncreaseExp = sumExp / expectedIncreaseArray.length

        let nOfDaysHerd = 0
        let getHerd = true
        let days = 0
        for (let count = this.formattedData[this.formattedData.length - 1].y; count <= 100; count = count + this.avgIncrease) {
          days++
          if (count >= 60 && getHerd === true) {
            nOfDaysHerd = days
            getHerd = false
          }
        }
        this.predictedEndDate = new Date(1611705600000 + (86400000 * days)).toLocaleDateString()
        this.predictedHerdImmunityDate = new Date(1611705600000 + (86400000 * nOfDaysHerd)).toLocaleDateString()
      } catch (error) {
        alert(error)
      }
    },
    redirect_github () {
      window.open('https://github.com/fabiom91/covid_vaccine_numbers')
    }
  }
}
</script>
