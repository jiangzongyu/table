<style scoped>
  th {
    width: 150px;
    height: 80px;
  }
  td {
    width: 150px;
    height: 80px;
    text-align: center;
  }
  .available {
    background: #00b38a;
    cursor: pointer;
  }
  .unavailable {
    background: #eee;
    cursor: not-allowed;
  }
</style>
<template>
  <div>
    <table cellspacing="0" border="1" rules="all">
      <tr v-for="(item, index) in tableData" :key="index">
        <th v-if="item.date">{{item.date}}</th>
        <th v-if="item.type == 'thead'" v-for="th in item.columns">
          <p v-if="th.date">{{th.date}}</p>
          <p v-if="th.day">{{th.day}}</p>
        </th>
        <td v-if="item.time">{{item.time}}</td>
        <td v-if="item.type == 'tbody'"
            :class="{available: isAvailable(item.time, td.date), unavailable: !isAvailable(item.time, td.date)}"
            v-for="(td, tdIndex) in item.columns"
            @click.prevent.stop="cellClick(item.time, td.date, index, tdIndex, isAvailable(item.time, td.date))">
          <i class="el-icon-check" v-show="td.checked"></i>
        </td>
      </tr>
    </table>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    data() {
      return {
        tableData: [
          {
            type: 'thead',
            date: '时间',
            columns: []
          },
          {
            type: 'tbody',
            time: '09:00-10:00',
            columns: []
          },
          {
            type: 'tbody',
            time: '09:30-10:30',
            columns: []
          },
          {
            type: 'tbody',
            time: '10:00-11:00',
            columns: []
          },
          {
            type: 'tbody',
            time: '21:00-22:00',
            columns: []
          },
          {
            type: 'tbody',
            time: '21:30-22:30',
            columns: []
          }
        ]
      }
    },
    props: {
      now: {
        type: [Date],
        default: function () {
          return new Date()
        }
      },
      valid: {
        type: [Array],
        default: function () {
          return []
        }
      }
    },
    computed: {},
    methods: {
      cellClick(time, date, itemIndex, tdIndex, available) {
        const tableData = this.tableData
        if (!available) return false
        this.resetChecked()
        tableData[itemIndex].columns[tdIndex].checked = true
        this.$emit('receiveScheme', {
          time: tableData[itemIndex].time,
          date: tableData[itemIndex].columns[tdIndex].date
        })
      },
      addZero(num) {
        return num < 10 ? '0' + num : num
      },
      dateFormat(d) {
        let chn = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
        let year = d.getFullYear()
        let month = d.getMonth() + 1
        let day = d.getDate()
        let week = d.getDay()
        return {
          date: year + '-' + this.addZero(month) + '-' + this.addZero(day),
          day: chn[week]
        }
      },
      isAvailable(time, date) {
        // the code in comments are designed for limit the selection when the time of table cell is less than 3 hours, thus you may extend it in your case
//        let start = time.slice(0, time.indexOf('-'))
//        let d = date + ' ' + start
//        let spanExpected = 3 * 60 * 60 * 1000
//        let span = new Date(d) - new Date()
//        return span > spanExpected
        return this.valid.find(item => item.time === time && item.date === date)
      },
      resetChecked() {
        const tableData = this.tableData
        tableData.forEach((item) => {
          item.columns.forEach(col => col.checked = false)
        })
      },
      calculate(now) {
        var ary = []
        var today = new Date(now.getTime())
        let d = this.dateFormat(today)
        ary.push({
          type: 'date',
          date: d.date,
          day: d.day,
          checked: false
        })
        for (let i = 0; i < 6; i++) {
          today.setDate(today.getDate() + 1)
          let d = this.dateFormat(today)
          ary.push({
            type: 'date',
            date: d.date,
            day: d.day,
            checked: false
          })
        }
        this.tableData.forEach((item, index) => {
          item.columns = JSON.parse(JSON.stringify(ary))
        })
      }
    },
    mounted() {
      this.$on('resetScheme', function (d) {
        this.resetChecked()
        this.$emit('receiveScheme', {})
      })
      this.calculate(this.now)
    },
    watch: {
      now(newVal, oldVal) {
        this.calculate(newVal)
        this.$emit('receiveScheme', {})
      }
    }
  }
</script>
