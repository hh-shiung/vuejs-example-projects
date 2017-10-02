<template>
  <div class="content">
    <div class="container">
      <div class="search-container">
        <input
          class="search-input"
          @keyup.enter="requestData"
          placeholder="npm package name"
          type="search"
          name="search"
          v-model="package"
        />
        <button class="search-button" @click="requestData">
          Search
        </button>
        <datepicker placeholder="Start Date" v-model="periodStart" name="start-date"></datepicker>
        <datepicker placeholder="End Date" v-model="periodEnd" name="end-date"></datepicker>
      </div>
      <div class="error-message" v-if="showError">
        {{ errorMessage }}
      </div>
      <hr />
      <h1 class="title" v-if="loaded">
        {{ packageName }}
      </h1>
      
      <div class="chart-container" v-if="loaded">
        <div class="chart-title">
          Downloads per Day <span>{{ formattedPeriod }}</span>
          <hr />
        </div>
        <div class="chart-content">
          <line-chart 
            v-if="loaded"
            :chart-data="downloads"
            :chart-labels="labels"
          ></line-chart>
        </div>
      </div>

      <div class="chart-container" v-if="loaded">
        <div class="chart-title">
          Downloads per Week <span>{{ formattedPeriod }}</span>
          <hr />
        </div>
        <div class="chart-content">
          <line-chart 
            v-if="loaded"
            :chart-data="downloadsWeek"
            :chart-labels="labelsWeek"
          ></line-chart>
        </div>
      </div>

      <div class="chart-container" v-if="loaded">
        <div class="chart-title">
          Downloads per Month <span>{{ formattedPeriod }}</span>
          <hr />
        </div>
        <div class="chart-content">
          <line-chart 
            v-if="loaded"
            :chart-data="downloadsMonth"
            :chart-labels="labelsMonth"
          ></line-chart>
        </div>
      </div>

      <div class="chart-container" v-if="loaded">
        <div class="chart-title">
          Downloads per Year <span>{{ formattedPeriod }}</span>
          <hr />
        </div>
        <div class="chart-content">
          <line-chart 
            v-if="loaded"
            :chart-data="downloadsYear"
            :chart-labels="labelsYear"
          ></line-chart>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Datepicker from 'vuejs-datepicker'
import LineChart from '@/components/LineChart'

import {
  dateToYear,
  dateToMonth,
  dateToWeek,
  dateToDay,
  dateBeautify
} from '../utils/dateFormatter.js'

import {removeDuplicate, groupData} from '../utils/downloadFormatter.js'

export default {
  components: {
    Datepicker,
    LineChart
  },
  data () {
    return {
      package: null,
      packageName: '',
      loaded: false,
      downloads: [],
      downloadsWeek: [],
      downloadsMonth: [],
      downloadsYear: [],
      labels: [],
      labelsWeek: [],
      labelsMonth: [],
      labelsYear: [],
      showError: false,
      errorMessage: 'Please enter a package name',
      periodStart: '',
      periodEnd: new Date(),
      rawData: '',
      totalDownloads: ''
    }
  },
  mounted () {
    if (this.$route.params.package) {
      this.package = this.$route.params.package
      this.requestData()
    }
  },
  computed: {
    _endDate () {
      return dateToDay(this.periodEnd)
    },
    _startDate () {
      return dateToDay(this.periodStart)
    },
    period () {
      return this.periodStart
      ? `${this._startDate}:${this._endDate}`
      : 'last-month'
    },
    formattedPeriod () {
      return this.periodStart
      ? `${dateBeautify(this._startDate)} - ${dateBeautify(this._endDate)}`
      : 'last-month'
    }
  },
  methods: {
    resetState () {
      this.loaded = false
      this.showError = false
    },
    requestData () {
      if (this.package === null || this.package === '' || this.package === 'undefined') {
        this.showError = true
        return
      }
      this.resetState()
      axios.get(`https://api.npmjs.org/downloads/range/${this.period}/${this.package}`)
        .then(response => {
          this.rawData = response.data.downloads
          this.downloads = response.data.downloads.map(entry => entry.downloads)
          this.labels = response.data.downloads.map(entry => entry.day)
          this.packageName = response.data.package
          this.totalDownloads = this.downloads.reduce((total, download) => total + download)
          this.setURL()
          this.groupDataByDate()
          this.loaded = true
        }).catch(err => {
          this.errorMessage = err.response.data.error
          this.showError = true
        })
    },
    groupDataByDate () {
      this.formatYear()
      this.formatMonth()
      this.formatWeek()
    },
    formatYear () {
      this.labelsYear = this.rawData.map(entry => dateToYear(entry.day)).reduce(removeDuplicate, [])
      this.downloadsYear = groupData(this.rawData, dateToYear)
    },
    formatMonth () {
      this.labelsMonth = this.rawData.map(entry => dateToMonth(entry.day)).reduce(removeDuplicate, [])
      this.downloadsMonth = groupData(this.rawData, dateToMonth)
    },
    formatWeek () {
      this.labelsWeek = this.rawData.map(entry => dateToWeek(entry.day)).reduce(removeDuplicate, [])
      this.downloadsWeek = groupData(this.rawData, dateToWeek)
    },
    setURL () {
      history.pushState({
        info: `npm-stats ${this.package}`},
        this.package,
        `/#/${this.package}`
      )
    }
  }
}
</script>
