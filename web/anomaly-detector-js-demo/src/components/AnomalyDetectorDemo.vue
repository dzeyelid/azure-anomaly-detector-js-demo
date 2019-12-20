<template>
  <div class="result">
    {{ result }}
  </div>
</template>

<script>
import msRest from '@azure/ms-rest-js'
import AnomalyDetector from '@azure/cognitiveservices-anomalydetector'
import parse from 'csv-parse'

export default {
  data: function () {
    return {
      result: 'Not detected yet.'
    }
  },
  methods: {
    fetchData: () => {},
    detectAnomaly: () => {
      this.data.result = 'detecting...';
    }
  },
}
</script>

