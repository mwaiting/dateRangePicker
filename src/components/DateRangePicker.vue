<template>
  <div class="date-range-picker" v-show="visible">
    <div class="date-range-picker-mask" @click="close"></div>
    <div class="date-range-picker-content">
      <div class="operation">
        <div
          class="cancel"
          @click="
            visible = false;
            $emit('cancel');
          "
          v-text="cancelText"
        ></div>
        <div class="confirm" @click="confirm" v-text="okText"></div>
      </div>
      <div class="time-range">
        <div class="range-item" :class="startActive" @click="selectItem('start')">
          <div class="text" v-text="startText"></div>
          <div v-text="startTime"></div>
        </div>
        <div class="range-item" :class="endActive" @click="selectItem('end')">
          <div class="text" v-text="endText"></div>
          <div v-text="endTime"></div>
        </div>
      </div>
      <div class="picker-view"><mt-picker :slots="dateSlots" @change="onChange" :visible-item-count="visibleItemCount" class="mint-datetime-picker" ref="picker"></mt-picker></div>
    </div>
  </div>
</template>

<script>
import { Toast } from 'mint-ui'
const FORMAT_MAP = {
  Y: 'year',
  M: 'month',
  D: 'date',
  H: 'hour',
  m: 'minute'
}
export default {
  name: 'DateRangePicker',
  props: {
    startText: {
      type: String,
      default: '开始时间'
    },
    endText: {
      type: String,
      default: '结束时间'
    },
    okText: {
      type: String,
      default: '确认'
    },
    toast: {
      type: String,
      default: '结束时间不能早于开始时间'
    },
    dateUnits: {
      type: Array,
      default () {
        return ['', '', '']
      }
    },
    cancelText: {
      type: String,
      default: '取消'
    },
    confirmText: {
      type: String,
      default: '确定'
    },
    type: {
      type: String,
      default: 'datetime'
    },
    startDate: {
      type: Date,
      default () {
        return new Date(new Date().getFullYear() - 10, 0, 1)
      }
    },
    endDate: {
      type: Date,
      default () {
        return new Date(new Date().getFullYear() + 10, 11, 31)
      }
    },
    startHour: {
      type: Number,
      default: 0
    },
    endHour: {
      type: Number,
      default: 23
    },
    yearFormat: {
      type: String,
      default: '{value}'
    },
    monthFormat: {
      type: String,
      default: '{value}'
    },
    dateFormat: {
      type: String,
      default: '{value}'
    },
    hourFormat: {
      type: String,
      default: '{value}'
    },
    minuteFormat: {
      type: String,
      default: '{value}'
    },
    visibleItemCount: {
      type: Number,
      default: 7
    },
    closeOnClickModal: {
      type: Boolean,
      default: true
    },
    value: {
      type: Date,
      default () {
        return new Date(new Date().getFullYear(), new Date().getMonth(), 1)
      }
    }
  },

  data () {
    return {
      visible: false,
      startYear: null,
      endYear: null,
      startMonth: 1,
      endMonth: 12,
      startDay: 1,
      endDay: 31,
      currentValue: null,
      selfTriggered: false,
      dateSlots: [],
      shortMonthDates: [],
      longMonthDates: [],
      febDates: [],
      leapFebDates: [],
      startActive: 'active',
      endActive: '',
      startTime: '',
      endTime: '',
      currentEdit: 'start'
    }
  },

  computed: {
    rims () {
      if (!this.currentValue) return { year: [], month: [], date: [], hour: [], min: [] }
      let result
      if (this.type === 'time') {
        result = {
          hour: [this.startHour, this.endHour],
          min: [0, 59]
        }
        return result
      }
      result = {
        year: [this.startDate.getFullYear(), this.endDate.getFullYear()],
        month: [1, 12],
        date: [1, this.getMonthEndDay(this.getYear(this.currentValue), this.getMonth(this.currentValue))],
        hour: [0, 23],
        min: [0, 59]
      }
      this.rimDetect(result, 'start')
      this.rimDetect(result, 'end')
      return result
    },

    typeStr () {
      if (this.type === 'time') {
        return 'Hm'
      } else if (this.type === 'date') {
        return 'YMD'
      } else {
        return 'YMDHm'
      }
    }
  },

  watch: {
    value (val) {
      this.currentValue = val
      if (this.currentEdit == 'start') {
        this.formatTime('start')
      } else {
        this.formatTime('end')
      }
    },

    rims () {
      this.generateSlots()
    },

    visible (val) {
      this.$emit('visible-change', val)
    }
  },

  mounted () {
    this.currentValue = this.value
    this.startTime = this.formatTimeValue(this.value, 'yyyy-MM-dd')
    this.endTime = this.formatTimeValue(this.endDate, 'yyyy-MM-dd')
    if (!this.value) {
      if (this.type.indexOf('date') > -1) {
        this.currentValue = this.startDate
        this.startTime = this.formatTimeValue(this.startDate, 'yyyy-MM-dd')
      } else {
        this.currentValue = `${('0' + this.startHour).slice(-2)}:00`
      }
    }
    this.generateSlots()
  },
  methods: {
    onCancel () {},
    onOk () {},
    formatTime (type, isCn) {
      let d = this.formatTimeValue(this.currentValue)
      if (type == 'start') {
        this.startTime = d
      } else {
        this.endTime = d
      }
    },
    formatTimeValue (value, rule, isCn) {
      let yyyy = value.getFullYear()
      let MM = value.getMonth() + 1
      let dd = value.getDate()
      let HH = value.getHours()
      let mm = value.getMinutes()
      let date = ""
      if (MM < 10) {
        MM = '0' + MM
      }
      if (dd < 10) {
        dd = '0' + dd
      }
      if (HH < 10) {
        HH = '0' + HH
      }
      if (mm < 10) {
        mm = '0' + mm
      }
      if (this.type == 'date') {
        date = yyyy + '-' + MM + '-' + dd
      } else {
        date = yyyy + '-' + MM + '-' + dd + ' ' + HH + ':' + mm
      }
      return date
    },
    selectItem (type) {
      if (this.currentEdit == type) {
        return
      }
      switch (type) {
        case 'start':
          this.startActive = 'active'
          this.endActive = ''
          this.currentEdit = 'start'
          this.currentValue = this.startTime
          break
        case 'end':
          this.startActive = ''
          this.endActive = 'active'
          this.currentEdit = 'end'
          this.currentValue = this.endTime
          break
        default:
          break
      }
    },
    open () {
      this.visible = true
    },

    close () {
      this.visible = false
    },

    isLeapYear (year) {
      return year % 400 === 0 || (year % 100 !== 0 && year % 4 === 0)
    },

    isShortMonth (month) {
      return [4, 6, 9, 11].indexOf(month) > -1
    },

    getMonthEndDay (year, month) {
      if (this.isShortMonth(month)) {
        return 30
      } else if (month === 2) {
        return this.isLeapYear(year) ? 29 : 28
      } else {
        return 31
      }
    },

    getTrueValue (formattedValue) {
      if (!formattedValue) return
      while (isNaN(parseInt(formattedValue, 10))) {
        formattedValue = formattedValue.slice(1)
      }
      return parseInt(formattedValue, 10)
    },

    getValue (values) {
      let value
      if (this.type === 'time') {
        value = values.map(value => ('0' + this.getTrueValue(value)).slice(-2)).join(':')
      } else {
        let year = this.getTrueValue(values[0])
        let month = this.getTrueValue(values[1])
        let date = this.getTrueValue(values[2])
        let maxDate = this.getMonthEndDay(year, month)
        if (date > maxDate) {
          this.selfTriggered = true
          date = 1
        }
        let hour = this.typeStr.indexOf('H') > -1 ? this.getTrueValue(values[this.typeStr.indexOf('H')]) : 0
        let minute = this.typeStr.indexOf('m') > -1 ? this.getTrueValue(values[this.typeStr.indexOf('m')]) : 0
        value = new Date(year, month - 1, date, hour, minute)
      }
      return value
    },

    onChange (picker) {
      let values = picker.$children.filter(child => child.currentValue !== undefined).map(child => child.currentValue)
      if (this.selfTriggered) {
        this.selfTriggered = false
        return
      }
      if (values.length !== 0) {
        this.currentValue = this.getValue(values)
        this.handleValueChange()
      }
    },

    fillValues (type, start, end) {
      let values = []
      for (let i = start; i <= end; i++) {
        if (i < 10) {
          values.push(this[`${FORMAT_MAP[type]}Format`].replace('{value}', ('0' + i).slice(-2)))
        } else {
          values.push(this[`${FORMAT_MAP[type]}Format`].replace('{value}', i))
        }
      }
      return values
    },

    pushSlots (slots, type, start, end) {
      slots.push({
        flex: 1,
        values: this.fillValues(type, start, end)
      })
    },

    generateSlots () {
      let dateSlots = []
      const INTERVAL_MAP = {
        Y: this.rims.year,
        M: this.rims.month,
        D: this.rims.date,
        H: this.rims.hour,
        m: this.rims.min
      }
      let typesArr = this.typeStr.split('')
      typesArr.forEach(type => {
        if (INTERVAL_MAP[type]) {
          this.pushSlots.apply(null, [dateSlots, type].concat(INTERVAL_MAP[type]))
        }
      })
      if (this.typeStr === 'Hm') {
        dateSlots.splice(1, 0, {
          divider: true,
          content: ':'
        })
      }
      this.dateSlots = dateSlots
      this.handleExceededValue()
    },

    handleExceededValue () {
      let values = []
      if (this.type === 'time') {
        const currentValue = this.currentValue.split(':')
        values = [this.hourFormat.replace('{value}', currentValue[0]), this.minuteFormat.replace('{value}', currentValue[1])]
      } else {
        values = [
          this.yearFormat.replace('{value}', this.getYear(this.currentValue)),
          this.monthFormat.replace('{value}', ('0' + this.getMonth(this.currentValue)).slice(-2)),
          this.dateFormat.replace('{value}', ('0' + this.getDate(this.currentValue)).slice(-2))
        ]
        if (this.type === 'datetime') {
          values.push(
            this.hourFormat.replace('{value}', ('0' + this.getHour(this.currentValue)).slice(-2)),
            this.minuteFormat.replace('{value}', ('0' + this.getMinute(this.currentValue)).slice(-2))
          )
        }
      }
      this.dateSlots
        .filter(child => child.values !== undefined)
        .map(slot => slot.values)
        .forEach((slotValues, index) => {
          if (slotValues.indexOf(values[index]) === -1) {
            values[index] = slotValues[0]
          }
        })
      this.$nextTick(() => {
        this.setSlotsByValues(values)
      })
    },

    setSlotsByValues (values) {
      const setSlotValue = this.$refs.picker.setSlotValue
      if (this.type === 'time') {
        setSlotValue(0, values[0])
        setSlotValue(1, values[1])
      }
      if (this.type !== 'time') {
        setSlotValue(0, values[0])
        setSlotValue(1, values[1])
        setSlotValue(2, values[2])
        if (this.type === 'datetime') {
          setSlotValue(3, values[3])
          setSlotValue(4, values[4])
        }
      }
      [].forEach.call(this.$refs.picker.$children, child => child.doOnValueChange())
    },

    rimDetect (result, rim) {
      let position = rim === 'start' ? 0 : 1
      let rimDate = rim === 'start' ? this.startDate : this.endDate
      if (this.getYear(this.currentValue) === rimDate.getFullYear()) {
        result.month[position] = rimDate.getMonth() + 1
        if (this.getMonth(this.currentValue) === rimDate.getMonth() + 1) {
          result.date[position] = rimDate.getDate()
          if (this.getDate(this.currentValue) === rimDate.getDate()) {
            result.hour[position] = rimDate.getHours()
            if (this.getHour(this.currentValue) === rimDate.getHours()) {
              result.min[position] = rimDate.getMinutes()
            }
          }
        }
      }
    },

    isDateString (str) {
      return /\d{4}(\-|\/|.)\d{1,2}\1\d{1,2}/.test(str)
    },

    getYear (value) {
      return this.isDateString(value) ? value.split(' ')[0].split(/-|\/|\./)[0] : value.getFullYear()
    },

    getMonth (value) {
      return this.isDateString(value) ? value.split(' ')[0].split(/-|\/|\./)[1] : value.getMonth() + 1
    },

    getDate (value) {
      return this.isDateString(value) ? value.split(' ')[0].split(/-|\/|\./)[2] : value.getDate()
    },

    getHour (value) {
      if (this.isDateString(value)) {
        const str = value.split(' ')[1] || '00:00:00'
        return str.split(':')[0]
      }
      return value.getHours()
    },

    getMinute (value) {
      if (this.isDateString(value)) {
        const str = value.split(' ')[1] || '00:00:00'
        return str.split(':')[1]
      }
      return value.getMinutes()
    },

    confirm () {
      let st = new Date(this.startTime)
      let et = new Date(this.endTime)
      if (st > et) {
        Toast('开始时间不能大于结束时间')
        return
      }
      this.visible = false
      this.$emit('confirm', { st: this.startTime, et: this.endTime })
    },

    handleValueChange () {
      this.$emit('input', this.currentValue)
    }
  }
}
</script>

<style lang="less" scoped>
.date-range-picker {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  z-index: 998;
  .date-range-picker-mask {
    position: absolute;
    height: 100%;
    width: 100%;
    background: rgba(0, 0, 0, 0.2);
  }
  .date-range-picker-content {
    position: fixed;
    width: 100%;
    left: 0;
    bottom: 0;
    z-index: 999;
  }
  .operation,
  .time-range {
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-sizing: border-box;
  }
  .operation {
    height: 1.2rem;
    line-height: 1.2rem;
    padding: 0 0.2rem;
    background: #ffffff;
    color: #2386ee;
    font-size: 0.4rem;
    border-bottom: 1px solid #d4d4d4;
    .cancel {
    }
    .confirm {
    }
  }
  .time-range {
    height: 1.2rem;
    font-size: 0.4rem;
    display: flex;
    .range-item {
      flex: 1;
      height: 100%;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      justify-content: center;
      background-color: #f4f5f9;
      align-items: center;
      font-weight: 400;
      color: #383838;
      .text {
        text-align: center;
        color: #333333;
        font-size: 0.36rem;
        padding: 0 0 0.14rem 0;
      }
      &.active {
        background-color: #ffffff;
      }
    }
  }
  .picker-view {
    width: 100%;
    display: flex;
    background: #ffffff;
    padding: 0.1rem 0;
    box-sizing: border-box;
    .mint-datetime-picker {
      width: 100%;
      /deep/ .picker-items {
        display: flex;
        overflow: hidden;
        position: relative;
        .picker-slot {
          .picker-slot-wrapper {
            .picker-item {
              font-size: 0.4rem;
            }
            .picker-selected {
              color: #000000;
              font-size: 0.4rem;
              font-weight: 400;
            }
          }
        }
        .picker-center-highlight {
          box-sizing: border-box;
          position: absolute;
          left: 0;
          width: 100%;
          top: 50%;
          margin-top: -18px;
          pointer-events: none;
          &::before,
          &::after {
            content: '';
            position: absolute;
            height: 1px;
            width: 100%;
            background-color: #eaeaea;
            display: block;
            z-index: 15;
            -webkit-transform: scaleY(0.5);
            transform: scaleY(0.5);
          }
          &::after {
            bottom: 0;
          }
        }
      }
    }
  }
}
.active {
  background-color: #ffffff;
}
</style>
