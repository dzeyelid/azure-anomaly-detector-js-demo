<template>
  <div>
    <div class="result">
      <h3 class="status">Status</h3>
      <div class="status">{{ status }}</div>
      <h3 class="raw-result">Raw result</h3>
      <code class="raw-result">
        {{ rawResult }}
      </code>
      <h3 class="sammary">Sammary</h3>
      <code class="sammary">
        {{ sammary }}
      </code>
    </div>
    <button v-on:click="detectAnomaly">Detect anomaly</button>
  </div>
</template>

<script>
import * as msRest from '@azure/ms-rest-js'
import * as AnomalyDetector from '@azure/cognitiveservices-anomalydetector'
import parse from 'csv-parse/lib/sync'
import axios from 'axios'

export default {
  data: function () {
    return {
      status: 'Not detected yet.',
      rawResult: null,
      sammary: null
    }
  },
  methods: {
    fetchData: async function () {
      const uri = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-quickstart-code/master/javascript/AnomalyDetector/request-data.csv'
      const response = await axios.get(uri)
      const parsed = parse(response.data, { skip_empty_lines: true })
      let points = []
      parsed.forEach((e) => {
        points.push({ timestamp: new Date(e[0]), value: parseInt(e[1]) })
      })
      return points
    },
    detectAnomaly: async function () {
      this.status = 'Detecting...'
      const points = await this.fetchData()

      const endpoint = process.env.VUE_APP_ANOMALY_DETECTOR_ENDPOINT
      const key = process.env.VUE_APP_ANOMALY_DETECTOR_KEY
      const credentials = new msRest.ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } })
      const anomalyDetectorClient = new AnomalyDetector.AnomalyDetectorClient(credentials, endpoint)
      const body = { series: points, granularity: 'daily' }
      const result = await anomalyDetectorClient.entireDetect(body)

      console.log(result)

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
