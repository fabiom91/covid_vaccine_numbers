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
                    <div class="column value">{{ (country_data) ? country_data[0].population : '' }}</div>
                </div>
                <div class="columns">
                    <div class="column field">Vaccinated</div>
                    <div class="column value">
                        <p>{{ (allNumbers.length > 0)? allNumbers.sort((a,b)=>a.date-b.date).reverse()[0].vaccinated : '' }}</p>
                        <p><b>{{ (formattedData.length > 0) ? formattedData.sort((a,b)=>a.x-b.x).reverse()[0].y : '' }}%</b> of total population</p>
                    </div>
                </div>
                <line-chart :percentage="formattedData"/>
                <h2>Last Update: {{ (formattedData.length > 0) ? formattedData.sort((a,b)=>a.x-b.x).reverse()[0].x : '' }}</h2>
            </b-tab-item>
            <b-tab-item label="Sources" icon="database">
                <div class="columns">
                    <div class="column">
                        <h4>Total Population Data</h4>
                        <p>The population number displayed in the "Data" page, comes from https://restcountries.eu/ API: https://restcountries.eu/rest/v2/name/ireland</p>
                    </div>
                    <div class="column">
                        <h4>Number of people vaccinated</h4>
                        <p> This data comes from the HSE covid tracker app: https://covidtracker.gov.ie/ and is manually updated daily. </p>
                    </div>
                </div>
            </b-tab-item>
            <b-tab-item label="GitHub" icon="github">
              <b-button class="is-dark" icon-left="github" @click="redirect_github()">Open GitHub repository</b-button>
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
      formattedData: []
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
        for (const obj of this.allNumbers) {
          const percentage = Math.round((obj.vaccinated * 10000) / this.country_data[0].population) / 100
          const formatted = {
            x: new Date(obj.date.seconds * 1000).toLocaleDateString(),
            y: percentage
          }
          this.formattedData.push(formatted)
        }
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
