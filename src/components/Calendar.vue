<template>
  <div :class="className">
    <div class="calendar-header">
      <h2>{{ displayDate }}</h2>
      <div class="btn_area">
        <button @click="handlePrevMonthClick">&lt;</button>
        <button @click="handleNextMonthClick">&gt;</button>
      </div>
    </div>
    <div class="calendar-wrapper">
      <div
        v-for="(week, index) in calendar"
        :key="index"
        class="row"
      >
        <div
          v-for="(day, dayIndex) in week"
          :key="dayIndex"
          class="col"
          @click="handleDayClick(day)"
        >
          <div
            v-if="index === 0"
            class="dayofweek"
          >{{ getDisplayDayOfWeek(day) }}</div>
          <div :class="{'day': !!getDisplayDay(day), 'day--current': isCurrentDay(day), 'day--active': !isCurrentDay(day) && !isDisabledDay(day), 'day--disabled': isDisabledDay(day) }">
            {{ getDisplayDay(day) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs'
import 'dayjs/locale/ja'
import JapaneseHolidays from 'japanese-holidays'

export default {
  props: {
    className: {
      type: String,
      default: '',
    },
    isDisabledBeforeDay: {
      type: Boolean,
      default: true,
    },
    isDisabledWeekend: {
      type: Boolean,
      default: true,
    },
    isDisabledHoliday: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      currentDate: dayjs(),
    }
  },
  computed: {
    displayDate() {
      return `${this.currentDate.format('YYYY年')} ${this.currentDate.format('M月')}`
    },
    calendar() {
      return this.getCalendar()
    },
  },
  methods: {
    getStartDate() {
      // カレンダーの最初の日付を取得する
      const date = dayjs(this.currentDate).startOf('month')
      return date.subtract(date.day(), 'days')
    },
    getEndDate() {
      // カレンダーの最後の日付を取得する
      const date = dayjs(this.currentDate).endOf('month')
      return date.add(6 - date.day(), 'days')
    },
    getCalendar() {
      const startDate = this.getStartDate()
      const endDate = this.getEndDate()
      // カレンダーの縦列数を取得
      const weekCnt = Math.ceil(endDate.diff(startDate, 'days') / 7)
      // カレンダー表示用の配列を生成する
      let calendar = []
      for (let week = 0; week < weekCnt; week++) {
        let weekRow = []
        for (let day = 0; day < 7; day++) {
          const currentDay = startDate.add(day, 'days').add(week, 'week')
          weekRow.push(currentDay)
        }
        calendar.push(weekRow)
      }
      return calendar
    },
    getDisplayDay(dayjsObj) {
      return (dayjsObj?.get('month') === this.currentDate?.get('month')) ? String(dayjsObj?.get('date')) : ''
    },
    getDisplayDayOfWeek(dayjsObj) {
      return ['日', '月', '火', '水', '木', '金', '土'][dayjsObj?.day()]
    },
    getDisplayHoliday(dayjsObj) {
      // 祝日の場合は祝日名が返却される @see https://github.com/osamutake/japanese-holidays-js
      return JapaneseHolidays.isHoliday(new Date(dayjsObj), true) || ''
    },
    isBeforeDay(dayjsObj) {
      return !this.isCurrentDay(dayjsObj) && dayjsObj.isBefore(dayjs())
    },
    isCurrentDay(dayjsObj) {
      return dayjsObj.format('YYYY/MM/DD') === dayjs().format('YYYY/MM/DD')
    },
    isWeekend(dayjsObj) {
      return ['日', '土'].includes(this.getDisplayDayOfWeek(dayjsObj))
    },
    isHolyday(dayjsObj) {
      return !!this.getDisplayHoliday(dayjsObj)
    },
    isDisabledDay(dayjsObj) {
      // 過去日が非活性指定、かつ過去日の場合は何もしない
      if (this.isDisabledBeforeDay && this.isBeforeDay(dayjsObj)) {
        return true
      }
      // 土日が非活性指定、かつ土日の場合は何もしない
      if (this.isDisabledWeekend && this.isWeekend(dayjsObj)) {
        return true
      }
      // 祝日が非活性指定、かつ祝日の場合は何もしない
      if (this.isDisabledHoliday && this.getDisplayHoliday(dayjsObj)) {
        return true
      }
      return false
    },
    handlePrevMonthClick() {
      this.currentDate = dayjs(this.currentDate).subtract(1, 'month')
    },
    handleNextMonthClick() {
      this.currentDate = dayjs(this.currentDate).add(1, 'month')
    },
    handleDayClick(dayjsObj) {
      if (this.isDisabledDay(dayjsObj)) {
        return
      }
      this.$emit('click', dayjsObj)
    },
  },
}
</script>

<style lang="scss" scoped>
$border-color: #6b6b6b;

$white: #ffffff;
$blue-base: #7e9caa;
$blue-light: #f8f8f8;

.calendar {

  &-header {
    display: flex;

    h2 {
      width: 100%;
      font-size: 1.8rem;
    }

    .btn_area {
      display: flex;
      align-items: center;
      width: 100%;
      justify-content: flex-end;

      button:not(:last-of-type) {
        margin-right: 4px;
      }
    }
  }

  &-wrapper {
    width: 100%;
    $fs-small  : 1.4rem;

    .row {
      display:flex;
    }

    .col {
      flex: 1;
      text-align: center;
      padding: 2px;

      .day {
          width: 28px;
          height: 28px;
          justify-content: center;
          align-items: center;
          display: inline-flex;
          cursor: pointer;

          &--current {
            border-radius: 50%;
            color: $white;
            background-color: $blue-base;
          }

          &--active {
            border-radius: 50%;
            background-color: $blue-light;
          }

          &--disabled {
            opacity: .5;
            cursor: initial;
          }
      }
    }
  }
}
</style>