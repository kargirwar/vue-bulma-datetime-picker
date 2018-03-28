<template lang="pug">
    .columns
        .column
        .column
        .column
        .column
        .column
        .column
            date-time-picker(
                :format="format"
                :start="start"
            )
</template>

<script>
import DateTimePicker from "./DateTimePicker.vue";

export default {
  name: 'app',
  data () {
    return {
        format: 'DD-MM-YYYY',
        start: '29-03-2018',
        end: '29-03-2018'
    }
  },
  components: {
        'date-time-picker': DateTimePicker
  }
}
</script>

<style lang="scss">
</style>
