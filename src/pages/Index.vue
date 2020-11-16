<template>
    <div>
      <div class="btn-item" @click="open">
          <div>选择时间</div>
      </div>
      <div class="time-wrapper">
        <div class="time-item">
            <div class="placeholder" v-show="!startTime">开始时间</div>
            <div v-show="startTime" v-text="startTime"></div>
        </div>
        <div class="space">-</div>
        <div class="time-item">
            <div class="placeholder" v-show="!endTime">结束时间</div>
            <div v-show="endTime" v-text="endTime"></div>
        </div>
      </div>
      <date-range-picker
          ref="dateRangePicker"
          v-model="dateValue"
          :startDate="startDate"
          :endDate="endDate"
          type="datetime"
          year-format="{value} 年"
          month-format="{value} 月"
          date-format="{value} 日"
          @confirm="dateConfirm"
        >
      </date-range-picker>
    </div>
</template>

<script>
import DateRangePicker from '@/components/DateRangePicker.vue'
export default {
  components: {
    DateRangePicker
  },
  data () {
    return {
      startTime: '',
      endTime: '',
      dateValue: new Date(new Date().getFullYear(), new Date().getMonth(), 1),
      startDate: new Date(new Date().getFullYear() - 10, 0, 1),
      endDate: new Date()
    }
  },

  mounted () {},

  methods: {
    open () {
      this.$refs.dateRangePicker.open()
    },
    dateConfirm (e) {
      this.startTime = e.st
      this.endTime = e.et
    }
  }
}
</script>

<style scoped>
  .btn-item {
    width: 2rem;
    height: 0.8rem;
    border: 1px solid #26A2FF;
    color: #26A2FF;
    border-radius: 0.2rem;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    margin: 0.6rem auto;
  }
  .time-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.4rem;
  }
  .space {
    padding: 0 0.2rem;
  }
  .placeholder {
    color: #999999;
  }
</style>
