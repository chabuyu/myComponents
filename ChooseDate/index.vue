<template>
  <div class="block">
    <el-date-picker
      class="choosedate"
      v-model="value2"
      type="daterange"
      align="right"
      unlink-panels
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
      @blur='showdate'
      :picker-options="pickerOptions">
    </el-date-picker>
  </div>
</template>

<script>
  export default {
    name: 'ChooseDate',
    props: {
    },
    data () {
      return {
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              end.setTime(start.getTime() + 3600 * 1000 * 24 * 7)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近一个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              end.setTime(start.getTime() + 3600 * 1000 * 24 * 30)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近三个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              end.setTime(start.getTime() + 3600 * 1000 * 24 * 90)
              picker.$emit('pick', [start, end])
            }
          }]
        },
        value2: ''
      }
    },
    methods: {
      showdate () {
        var startTime = this.changeDate(this.value2[0])
        var endTime = this.changeDate(this.value2[1])
        // 调用页面使用  @getDate="事件名称" 获取数据
        this.$emit('getDate', [startTime, endTime])
      },
      changeDate (date) {
        var d = new Date(date)
        var datetime
        if (d.getMonth() < 9) {
          if (d.getDate() < 10) {
            datetime = d.getFullYear() + '-0' + (d.getMonth() + 1) + '-0' + d.getDate()
          } else {
            datetime = d.getFullYear() + '-0' + (d.getMonth() + 1) + '-' + d.getDate()
          }
        } else {
          if (d.getDate() < 10) {
            datetime = d.getFullYear() + '-' + (d.getMonth() + 1) + '-0' + d.getDate()
          } else {
            datetime = d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate()
          }
        }
        return datetime
      }
    }
  }
</script>
