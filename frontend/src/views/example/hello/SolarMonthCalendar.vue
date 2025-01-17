<script setup lang="ts">
import {reactive, watch} from 'vue'
import {Solar, SolarMonth, HolidayUtil} from 'lunar-typescript'

const now = Solar.fromDate(new Date())

class Day {
  public day: number = 0
  public text: string = ''
  public isFestival: boolean = false
  public isToday: boolean = false
  public isOther: boolean = false
  public isHoliday: boolean = false
  public isRest: boolean = false
}

class Week {
  public index: number = 0
  public days: Day[] = []
}

const state = reactive({
  year: now.getYear(),
  month: now.getMonth(),
  weekStart: 1,
  heads: ['一', '二', '三', '四', '五', '六', '日'],
  weeks: new Array<Week>()
})

function buildDay(d: Solar) {
  const lunar = d.getLunar()
  const day = new Day()
  day.day = d.getDay()
  let text = lunar.getDayInChinese()
  if (1 === lunar.getDay()) {
    text = lunar.getMonthInChinese() + '月'
  }
  let otherFestivals = d.getOtherFestivals()
  if (otherFestivals.length > 0) {
    text = otherFestivals[0]
    day.isFestival = true
  }
  otherFestivals = lunar.getOtherFestivals()
  if (otherFestivals.length > 0) {
    text = otherFestivals[0]
    day.isFestival = true
  }
  let festivals = d.getFestivals()
  if (festivals.length > 0) {
    text = festivals[0]
    day.isFestival = true
  }
  festivals = lunar.getFestivals()
  if (festivals.length > 0) {
    text = festivals[0]
    day.isFestival = true
  }
  const jq = lunar.getJieQi()
  if (jq) {
    text = jq
    day.isFestival = true
  }
  day.text = text
  if (d.toYmd() === now.toYmd()) {
    day.isToday = true
  }
  if (d.getMonth() !== state.month) {
    day.isOther = true
  }
  const h = HolidayUtil.getHoliday(d.getYear(), d.getMonth(), d.getDay())
  if (h) {
    day.isHoliday = true
    day.isRest = !h.isWork()
  }
  return day
}

function render() {
  const month = SolarMonth.fromYm(state.year, state.month)
  const weeks: Week[] = []
  month.getWeeks(state.weekStart).forEach(w => {
    const week = new Week()
    week.index = w.getIndexInYear()
    const days: Day[] = []
    w.getDays().forEach(d => {
      days.push(buildDay(d))
    })
    week.days = days
    weeks.push(week)
  })
  state.weeks = weeks
}

render()

watch(() => state.year, () => {
  render()
})

watch(() => state.month, () => {
  render()
})

function onPrevMonth() {
  const month = SolarMonth.fromYm(state.year, state.month)!.next(-1)!
  state.year = month.getYear()
  state.month = month.getMonth()
}

function onNextMonth() {
  const month = SolarMonth.fromYm(state.year, state.month)!.next(1)!
  state.year = month.getYear()
  state.month = month.getMonth()
}

</script>

<template>
  <div class="calendar">
    <div class="title">
      <a href="javascript:void(0);" @click="onPrevMonth">&lt;</a>
      {{ state.year }}年 {{ state.month }}月
      <a href="javascript:void(0);" @click="onNextMonth">&gt;</a>
    </div>
    <div class="body">
      <ul class="week">
        <li v-for="(head, index) in state.heads" :class="{first: index === 0}">星期{{ head }}</li>
      </ul>
      <ul class="day" v-for="week in state.weeks">
        <li class="row">{{ week.index }}<br>周</li>
        <li v-for="day in week.days"
            :class="{festival: day.isFestival, today: day.isToday, other: day.isOther, rest: day.isRest}">
          {{ day.day }}
          <i>{{ day.text }}</i>
          <u v-if="day.isHoliday"> {{ day.isRest ? '休' : '班' }}</u>
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.calendar * {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.calendar {
  margin: 0;
  padding: 0;
  box-sizing: border-box;

  a {
    color: #409EFF;
    text-decoration: none;
  }

  div.title {
    height: 48px;
    line-height: 48px;
    font-size: 15px;
    color: #606266;
    width: 160px;
    margin: 0 auto;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }

  div.body {
    overflow: hidden;
    width: 725px;
    border-right: 1px solid #DDD;
    border-bottom: 1px solid #DDD;

    li {
      position: relative;
      display: block;
      float: left;
      width: 98px;
      text-align: center;
      border-left: 1px solid #DDD;
      border-top: 1px solid #DDD;
      overflow: hidden;
    }

    ul {
      list-style: none;

      li {
        float: left;
      }

      li.first {
        width: 136px;
      }
    }

    ul.week {
      li {
        height: 45px;
        line-height: 45px;
        color: #252525;
        background-color: #EEE;
        font-size: 14px;
      }
    }

    ul.day {
      li {
        height: 86px;
        font-size: 24px;
        font-weight: bold;
        color: #1F80A9;
        padding-top: 16px;

        i {
          display: block;
          font-style: normal;
          font-size: 13px;
          color: #8C8C8C;
          padding: 0 10px;
          line-height: 13px;
        }

        u {
          font-size: 12px;
          font-style: normal;
          text-decoration: none;
          color: #FFF;
          position: absolute;
          right: 0;
          top: 0;
          width: 26px;
          height: 26px;
          text-align: center;
          line-height: 26px;
        }

        u::before {
          content: '';
          position: absolute;
          right: 0;
          top: 0;
          border: 20px solid #D9534F;
          border-left-color: transparent;
          border-bottom-color: transparent;
          z-index: -1;
        }
      }

      li.row {
        width: 38px;
        color: #A0A0A0;
        font-size: 14px;
        padding-top: 24px;
        font-weight: normal;
        overflow: hidden;
      }

      li.other {
        color: #AAA;

        * {
          filter: alpha(opacity=40);
          -moz-opacity: 0.4;
          opacity: 0.4;
        }
      }

      li.festival {
        i {
          color: #D02F12;
          font-size: 9px;
        }
      }

      li.rest {
        u::before {
          border-right-color: #5CB85C;
          border-top-color: #5CB85C;
        }
      }

      li.today {
        background: #E6A23C;
        color: #FFFFFF;

        i {
          color: #FFFFFF;
        }

        u {
          color: #FFFFFF;
        }
      }
    }
  }
}
</style>
