<script setup lang="ts">
import {reactive, watch} from 'vue'
import {Solar, SolarYear, HolidayUtil} from 'lunar-typescript'

const now = Solar.fromDate(new Date())

class Day {
  public day: string = ''
  public text: string = ''
  public isFestival: boolean = false
  public isToday: boolean = false
  public isHoliday: boolean = false
  public isRest: boolean = false
  public isCurrentMonth: boolean = false
}

class Month {
  public month: number = 0
  public days: Day[] = []
}

const state = reactive({
  year: now.getYear(),
  weekStart: 0,
  heads: new Array<String>(),
  months: new Array<Month>()
})

function buildDay(d: Solar, month: number) {
  const lunar = d.getLunar()
  const day = new Day()
  day.day = d.getDay() + ''
  let text = lunar.getDayInChinese()
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
  if (1 === lunar.getDay()) {
    text = lunar.getMonthInChinese() + '月'
    day.isFestival = false
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
  const h = HolidayUtil.getHoliday(d.getYear(), d.getMonth(), d.getDay())
  if (h) {
    day.isHoliday = true
    day.isRest = !h.isWork()
  }
  day.isCurrentMonth = d.getMonth() == month
  return day
}

function render() {
  const heads = ['日', '一', '二', '三', '四', '五', '六']
  state.heads = heads.slice(state.weekStart).concat(heads.slice(0, state.weekStart - 7))
  const year = SolarYear.fromYear(parseInt(state.year + '', 10))
  const months: Month[] = []
  year.getMonths().forEach(m => {
    const month = new Month()
    month.month = m.getMonth()
    const weeks = m.getWeeks(state.weekStart)
    weeks.forEach(w => {
      w.getDays().forEach(d => {
        month.days.push(buildDay(d, month.month))
      })
    })
    months.push(month)
  })
  state.months = months
}

render()

watch(() => state.year, () => {
  render()
})

function setStart(start: number) {
  state.weekStart = start
  render()
}

</script>

<template>
  <div class="calendar">
    <div class="start">
      <a :class="{active: state.weekStart === 0}" @click="setStart(0)">周日起</a>
      <a :class="{active: state.weekStart === 1}" @click="setStart(1)">周一起</a>
      <a :class="{active: state.weekStart === 6}" @click="setStart(6)">周六起</a>
    </div>
    <div class="title">
      <input type="text" v-model="state.year" placeholder="阳历年">
    </div>
    <div class="body">
      <ul v-for="month in state.months">
        <h3>{{ month.month }}月</h3>
        <li class="week" v-for="head in state.heads">{{ head }}</li>
        <li v-for="day in month.days"
            :class="{festival: day.isFestival, today: day.isToday, rest: day.isRest, other: !day.isCurrentMonth}">
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
  width: 1110px;

  div.start {
    position: absolute;
    right: 20px;
    top: 20px;
    border: 1px solid #646cff;
    border-radius: 6px;
    overflow: hidden;

    a {
      padding: 5px;
      cursor: pointer;
    }

    a.active {
      background: #646cff;
      color: #ffffff;
    }
  }

  div.title {
    font-size: 15px;
    color: #606266;
    margin-bottom: 10px;

    input {
      border: 1px solid #D7D9E0;
      padding: 7px;
      border-radius: 6px;
      background: #FFFFFF;
      width: 100px;
      text-align: center;
      font-size: 20px;
    }
  }

  div.body {
    overflow: hidden;

    ul {
      position: relative;
      display: block;
      width: 350px;
      height: 334px;
      float: left;
      list-style: none;
      background-color: #FAFAFA;
      margin: 10px;

      h3 {
        position: absolute;
        left: 0;
        top: 0;
        font-size: 80px;
        width: 100%;
        height: 334px;
        text-align: center;
        line-height: 334px;
        color: #333;
        opacity: 0.1;
        filter: alpha(opacity=10);
      }

      li {
        position: relative;
        float: left;
        width: 50px;
        height: 50px;
        cursor: default;
        color: #444;
        display: flex;
        flex-direction: column;
        justify-content: center;

        i {
          display: block;
          font-size: 9px;
          font-style: normal;
          color: #999999;
          white-space: nowrap;
          overflow: hidden;
        }

        u {
          font-size: 9px;
          font-style: normal;
          text-decoration: none;
          position: absolute;
          right: 4px;
          top: 0;
          color: #333;
        }
      }

      li.week {
        height: 34px;
        line-height: 33px;
        font-size: 12px;
        color: #333333;
        border-bottom: 1px solid #E3E3E3;
      }

      li.rest {
        u {
          color: #5CB85C;
        }
      }

      li.festival {
        i {
          color: #D02F12;
        }
      }

      li.today {
        font-weight: bold;
        color: #1f80a9;

        i {
          color: #1f80a9;
        }
      }

      li.other {
        opacity: 0.3;
      }
    }
  }
}
</style>
