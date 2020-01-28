<template>
  <div>
    <div class="result">
      <h3 class="source">Data source</h3>
      <div class="source">
        <select v-model="selectedDataSource">
          <option disabled value="">Please select one</option>
          <option v-for="option in optionsDataSources" v-bind:key="option.value" v-bind:value="option.value">
            {{ option.text }}
          </option>
        </select>
        <div>
          Selected: {{ selectedDataSource }}
        </div>
      </div>
      <button v-on:click="detectAnomaly">Detect anomaly</button>
      <h3 class="status">Status</h3>
      <div class="status">{{ status }}</div>
      <h3 class="charts">Charts</h3>
      <div class="charts">
        <apexchart width="80%" type="line" align="center" :options="options" :series="series"></apexchart>
      </div>
      <h3 class="raw-result">Raw result</h3>
      <code class="raw-result">
        {{ rawResult }}
      </code>
      <h3 class="sammary">Sammary</h3>
      <code class="sammary">
        {{ sammary }}
      </code>
    </div>
  </div>
</template>

<script>
import * as msRest from '@azure/ms-rest-js'
import * as AnomalyDetector from '@azure/cognitiveservices-anomalydetector'
import parse from 'csv-parse/lib/sync'
import axios from 'axios'
import TsiClient from 'tsiclient'
import * as moment from 'moment'
import VueApexCharts from 'vue-apexcharts'

export default {
  components: {
    apexchart: VueApexCharts
  },
  data: function () {
    return {
      status: 'Not detected yet.',
      rawResult: null,
      sammary: null,
      optionsDataSources: [
        { text: 'Demo', value: 'demo' },
        { text: 'Azure Time Series Insights', value: 'tsi' }
      ],
      selectedDataSource: '',
      options: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: []
        },
        yaxis: {
          labels: {
            formatter: function (val, index) {
              return val.toFixed(2)
            }
          }
        },
        markers: {}
      },
      series: []
    }
  },
  methods: {
    fetchData: async function () {
      const uri = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-quickstart-code/master/javascript/AnomalyDetector/request-data.csv'
      const response = await axios.get(uri)
      const parsed = parse(response.data, { skip_empty_lines: true })
      const data = {
        timestamps: [],
        values: [],
        count: parsed.length
      }
      parsed.map(e => {
        data.timestamps.push(new Date(e[0]))
        data.values.push(parseInt(e[1]))
      })
      return data
    },
    fetchDataFromTsi: async function () {
      const tsiClient = new TsiClient()
      const token = process.env.VUE_APP_TSI_AAD_TOKEN
      const environmentFqdn = process.env.VUE_APP_TSI_ENV_FQDN
      const tsqArray = [
        {
          aggregateSeries: {
            timeSeriesId: [
              'remo'
            ],
            searchSpan: {
              from: moment().subtract(3, 'days').toISOString(),
              to: moment().toISOString()
            },
            interval: 'PT1H',
            filter: null,
            projectedVariables: [
              'Illumination'
            ]
          }
        }
      ]
      const results = await tsiClient.server.getTsqResults(token, environmentFqdn, tsqArray)
      console.log(results)
      const data = {
        timestamps: results[0].timestamps.map(e => new Date(e)),
        values: results[0].properties[0].values.map(e => parseInt(e)),
        count: results[0].properties[0].values.length
      }
      return data
    },
    convertDataToPoints: function (data) {
      const points = []
      for (let i = 0; i < data.count; i++) {
        points.push({ timestamp: data.timestamps[i], value: data.values[i] })
      }
      return points
    },
    updateChart: function (categories, series, markers) {
      let options = this.options
      options = {
        xaxis: {
          categories
        },
        markers
      }
      this.options = options
      this.series = series
    },
    createMakerDiscretes: function (result) {
      const base = {
        seriesIndex: 0,
        fillColor: '#5c6aae',
        strokeColor: '#fff',
        size: 8
      }
      const discretes = []
      result.isAnomaly.map((isAnomaly, index) => {
        if (isAnomaly) {
          discretes.push({
            dataPointIndex: index,
            ...base
          })
        }
      })
      return discretes
    },
    detectAnomaly: async function () {
      this.status = 'Detecting...'
      let data = []
      let granularity = 'daily'
      if (this.selectedDataSource === 'demo') {
        data = await this.fetchData()
      } else if (this.selectedDataSource === 'tsi') {
        data = await this.fetchDataFromTsi()
        granularity = 'hourly'
      } else {
        this.status = 'Did nothing.'
        return
      }

      const points = this.convertDataToPoints(data)
      console.log(points)

      const endpoint = process.env.VUE_APP_ANOMALY_DETECTOR_ENDPOINT
      const key = process.env.VUE_APP_ANOMALY_DETECTOR_KEY
      const credentials = new msRest.ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } })
      const anomalyDetectorClient = new AnomalyDetector.AnomalyDetectorClient(credentials, endpoint)
      const body = { series: points, granularity }
      const result = await anomalyDetectorClient.entireDetect(body)

      console.log(result)

      const makerDiscretes = this.createMakerDiscretes(result)
      this.updateChart(
        data.timestamps,
        [
          {
            name: 'originalData',
            data: data.values
          },
          {
            name: 'expectedValues',
            data: result.expectedValues
          }
        ],
        {
          size: 1,
          discrete: makerDiscretes
        }
      )

      const sammary = {
        anomalyNumber: result.isAnomaly.filter((e) => e).length
      }
      this.status = 'Detected!'
      this.rawResult = JSON.stringify(result)
      this.sammary = sammary
    }
  }
}
</script>

<style scoped>
</style>
