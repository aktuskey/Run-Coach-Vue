<template>
  <div class="wrapper">
    <header class="header">
      <div class="profile">
        <div class="profile__img" v-bind:style="{ 'background-image': 'url(' + profile.profileImage + ')' }"></div> 
        <div class="profile__name">{{ profile.firstName }}</div>
      </div>
      <div class="tracker">
      <!-- <div class="tracker__week">Week 1</div> -->
      <div class="tracker__count" v-html="getDaysUntil + ' Days to Go!'"></div>
      <div class="tracker__progress">
          <div class="tracker__progress__bar" v-bind:style="progressBarStyle"></div>
        </div>
      <div class="tracker__date" v-html="getDayPlan.date"></div>
    </div>
    </header>
    <div class="plan-card">
      <div class="plan-card__text-wrapper">
        <h1 v-html="getDistance(getDayPlan)" class="plan-card__distance"></h1>
        <div v-if="isARun(getDayPlan)" class="plan-card__units">Miles</div>
        <div v-if="isARestDay(getDayPlan)" class="plan-card__note">You earned it.</div>
        <div v-if="isSpeedwork(getDayPlan)" class="plan-card__note" v-html="'Repetitions: ' + ' / ' + 'Rest: ' + getDayPlan.restInterval"></div>
        <!-- <div v-html="getDayPlan.pace"></div> -->
      </div>
      <button class="log-btn" v-on:click="logRun(getDayPlan)">
        <img v-if="isLogged(getDayPlan)" src="/src/assets/check-green.svg">
        <img v-else src="/src/assets/check.svg">
      </button>
      <div v-if="isLogged(getDayPlan)">Completed!</div>
      <div v-else>Log Run</div>
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
  mounted () {
    this.plan = JSON.parse(window.localStorage.getItem('plan')) || this.plan
  },
  computed: {
    raceDate () {
      return this.profile.targetRaceDate
    },
    getPlanStartDate () {
      let firstWeek = this.plan.weeks[0]
      let start = firstWeek[0].date
      return start
    },
    getTotal () {
      let race = moment(this.raceDate, 'MMMM Do YYYY')
      let start = moment(this.getPlanStartDate, 'MMMM Do YYYY')
      let total = race.diff(start, 'days')
      return total
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
    },
    percentage () {
      let total = this.getTotal - this.getDaysUntil
      let grandTotal = this.getTotal
      return total / grandTotal
    },
    progressBarStyle () {
      return {
        'width': (this.percentage * 100) + '%'
      }
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
      } else if (plan.activity == 'Rest') {
        return plan.activity
      } else {
        return plan.activity
      }
    },
    isARun(plan) {
      return plan.activity == 'Run'
    },
    isARestDay(plan) {
      return plan.activity == 'Rest'
    },
    isSpeedwork(plan) {
      return plan.activity == 'Speedwork'
    },
    isLogged (plan) {
      return plan.isLogged == true
    },
    logRun (plan) {
      if (!plan.isLogged) {
        plan.isLogged = true
        window.localStorage.setItem('plan', JSON.stringify(this.plan))
      } else {
        plan.isLogged = false
        window.localStorage.clear()
      }
    }
  }
}
</script>