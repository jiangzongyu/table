<template>
  <div>
    <el-button @click="reset">重置</el-button>
    <el-button @click="thisWeek">本周</el-button>
    <el-button @click="nextWeek">下周</el-button>
    <scheme :now="now"
            :valid="validDate"
            @receiveScheme="handleScheme"
            ref="schemeRef" ></scheme>
  </div>
</template>

<script type="text/ecmascript-6">
  import scheme from '~/components/layout/scheme'
  export default {
    data() {
      return {
        now: new Date(),
        validDate: [
          {
            date: '2018-06-29',
            time: '09:30-10:30'
          },
          {
            date: '2018-07-08',
            time: '21:30-22:30'
          },
          {
            date: '2018-07-08',
            time: '09:00-10:00'
          }
        ]
      }
    },
    methods: {
      handleScheme(d) {
        console.log(d)
      },
      reset() {
        this.$refs['schemeRef'].$emit('resetScheme', true)
      },
      nextWeek() {
        if (this.now.getDate() === (new Date().getDate() + 7)) return false
        this.now = new Date(new Date().getTime() + 7 * 24 * 60 * 60 * 1000)
      },
      thisWeek() {
        this.now = new Date()
      }
    },
    components: {
      scheme
    },
    mounted() {}
  }
</script>
