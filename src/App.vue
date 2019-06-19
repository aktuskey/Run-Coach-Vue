<template>
  <div class="wrapper">
    <header class="header"> 
      <div class="profile__name">{{ profile.firstName }}</div>
    </header>
    <div class="tracker">
      <!-- <div class="tracker__week">Week 1</div> -->
      <div class="tracker__count" v-html="getDaysUntil + ' Days to Go!'"></div>
    </div>
    <div class="plan-card">
      <div class="plan-card__date" v-html="getDayPlan.date"></div>
      <h1 v-html="getDistance(getDayPlan)"></h1>
      <!-- <div v-html="getDayPlan.pace"></div> -->
    </div>
    <button v-on:click="decrementDay()">Back</button>
    <button v-on:click="incrementDay()">Next</button>
  </div>
</template>

<script>
import profile from '../data/profile.json'
import plan from '../data/plan.json'
import moment from 'moment'

export default {
  name: 'App',
  data() {
    return {
      profile,
      plan,
      date: moment().format('MMMM Do YYYY')
    }
  },
  computed: {
    raceDate () {
      return this.profile.targetRaceDate
    },
    getDaysUntil () {
      let current = moment(this.date, 'MMMM Do YYYY')
      let race = moment(this.raceDate, 'MMMM Do YYYY')
      let daysUntil = race.diff(current, 'days')
      return daysUntil
    },
    getDayPlan () {
      // Starting with [[{},{}], [{},{}]]
      let matchedWeek = this.plan.weeks.map((week) => week.filter((day) => {
        return day.date == this.date
      }
      )) // Returns [[0],[{...}]]
      let matchedDate = matchedWeek.filter((week) => {
        return week.length > 0
      }) // Returns [[{...}]]
      let matchedPlan = matchedDate.map((week) => week.find((day) => {
        return day.date == this.date
      })) // Returns [{...}]
      let planData = matchedPlan.find((day) => {
        return day.date == this.date
      })
      return planData // Returns {...}
    }
  },
  methods: {
    incrementDay() {
      this.date = moment(this.date, 'MMMM Do YYYY').add(1, 'd').format('MMMM Do YYYY')
    },
    decrementDay() {
      this.date = moment(this.date, 'MMMM Do YYYY').subtract(1, 'd').format('MMMM Do YYYY')
    },
    getDistance(plan) {
      if (plan.activity == 'Run') {
        return plan.distance
      } else if (plan.activity == 'Speedwork') {
        return plan.distance.join(', ')
      } else if (plan.activity == 'Rest Day') {
        return plan.activity
      }
    }
  }
}
</script>

<style scoped>
  .wrapper {
    text-align: center;
  }
</style>