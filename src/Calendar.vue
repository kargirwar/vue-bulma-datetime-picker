<template lang="pug">
    table.table.is-narrow
        thead
            tr
                th(v-for="day in days") {{day}}
        tbody
            tr(v-for="week in weeks")
                td(
                    v-bind:class="d.style"
                    @click="onClick(d)"
                    v-for="d in week") {{d.moment.format('D')}}

</template>

<script>
import Moment from 'moment';
const ROWS = 6;
const COLUMNS = 7;
const F_0_S_0 = "f-0-s-0";
const F_1_S_0 = "f-1-s-0";
const F_1_S_1 = "f-1-s-1";
const F_0_S_1 = "f-1-s-1";//invalid state
const REF_FORMAT = 'YYYY-MM-DD';

export default {
    props: {
        mY: {
            type: Object,
            required: true
        },
        state: {
            type: Object,
            required: true
        }
    },
    data () {
        return {
            weeks: [],
            refs: {},
            days: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
        }
    },
    created() {
        this.fillCalendar();
    },
    watch: {
        mY: function(n, o) {
            console.log('Changed mY');
            this.fillCalendar();
        }
    },
    methods: {
        highlightRange: function(curr) {
            switch (this.state) {
            case F_0_S_0:
            case F_1_S_1:
                return;
            }

            //if a date has been selected highlight all 
            //dates upto d
            var weeks = [this.m1.weeks, this.m2.weeks];	

            for (var k = 0; k < weeks.length; k++) {
                for (var i = 0; i < weeks[k].length; i++) {
                    for (var j = 0; j < weeks[k][i].length; j++) {
                        var d = weeks[k][i][j];
                        var m = Moment(d.moment);
                        if (m.isAfter(this.d1.moment) && m.isSameOrBefore(curr.moment)) {
                            d.style.range = true;
                        } else {
                            d.style.range = false;
                        }

                        //console.log(
                        //"d1: " + Moment(this.d1.moment).format('DD-MMM') +
                        //" m: " + m.format('DD-MMM') +
                        //" curr: " + Moment(curr.moment).format('DD-MMM') + 
                        //" range: " + d.style.range);
                    }
                }
            }
        },
        onClick: function(d) {
            console.log(d.moment.format(REF_FORMAT));
            //switch (this.state) {
            //case F_0_S_0:
                //this.f0s0(d);
                //break;
            //case F_1_S_0:
                //this.f1s0(d);
                //break;
            //case F_1_S_1:
                //this.f1s1(d);
                //break;
            //case F_0_S_1:
                //return;
            //}
        },
        f0s0: function(d) {
            //no date selected yet
            this.d1 = d;
            this.d1.style.selected = true;
            this.state = F_1_S_0;
        },
        f1s0: function(d) {
            //check if second date is after first, otherwise mark this as the first date
            if (Moment(d.moment).isSameOrAfter(this.d1.moment)) {
                this.d2 = d;
                this.d2.style.selected = true;
                this.d2.style.range = false;
                this.state = F_1_S_1;
                this.showSelectedRange();
                return;
            }

            //unselect the original d1 and set a new d1
            this.d1.style.selected = false;
            this.d1 = d;
            this.d2 = null;
            this.d1.style.selected = true;
            this.state = F_1_S_0;
        },
        f1s1: function(d) {
            //unselect already selected ones. start afresh
            this.clearRange();
            this.d1.style.selected = null;
            this.d2.style.selected = null;
            this.d1 = null;
            this.d2 = null;
            this.f0s0(d);
        },
        clearRange: function() {
            var start = Moment(this.d1.moment);
            var m = start.add(1, 'days');
            while (m.isBefore(this.d2.moment)) {
                var d = this.refs[m.format(REF_FORMAT)];
                d.style.range = false;
                m.add(1, 'days');
            }
        },
        showSelectedRange: function() {
            //from d1 through d2 set range = true
            var start = Moment(this.d1.moment);
            var m = start.add(1, 'days');
            while (m.isBefore(this.d2.moment)) {
                var d = this.refs[m.format(REF_FORMAT)];
                d.style.range = true;
                m.add(1, 'days');
            }
        },
        //render: function() {
            ////month of the starting date
            //var mY1 = Moment().month(this.month1).year(this.year1);
            //this.fillCalendar(mY1, this.m1['weeks']);
//
            //next month
            //var mY2 = Moment().month(this.month2).year(this.year2);
            //this.fillCalendar(mY2, this.m2['weeks']);
        //},
        fillCalendar: function() {
            this.weeks = [];
            var m = Moment(this.mY).startOf('month').startOf('week').clone();

            var isToday = false;
            var isOtherMonth = false;
            var isSelected = false;

            //we want a 7x7 grid starting from the sunday of the week for the 1st of month
            for (var i = 0; i < ROWS; i++) {
                var week = []
                for (var j = 0; j < COLUMNS; j++) {
                    //isToday = Moment().isSame(m, 'day');
                    //isOtherMonth = m.isSame(this.mY, 'month') ? false : true;
                    //isSelected = (m.isSame(this.dt1) || m.isSame(this.dt2)) ? true : false;

                    week.push({
                        moment: m.clone(),
                        style: this.getStyle(m)
                    });
                    m.add(1, 'days');
                }

                this.weeks.push(week);
            }
        },
        getStyle: function(m) {
            var style = {
                today: false,
                selected: false,
                range: false
            };
            var now = Moment();
            if (m.isSame(now, 'day')) {
                //today
                style.today = true;
            }

            switch (this.state) {
            case F_1_S_0:
                if (m.isSame(this.d1.moment)) {
                    style.selected = true;
                }
                break;

            case F_1_S_1:
                if (m.isSame(this.d1.moment) || m.isSame(this.d2.moment)) {
                    style.selected = true;
                }

                if (m.isAfter(this.d1.moment) && m.isBefore(this.d2.moment)) {
                    style.range = true;
                }
                break;
            }
            return style;
        }
    }
}
</script>
<style lang="scss" scoped>
    @import "../node_modules/bulma/bulma.sass";
    td,
    .prevMonth,
    .nextMonth {
        cursor: pointer
    }
    .today {
        color: $danger;
    }
    .selected {
        background: #357ebd;
        color:#fff
    }
    .range {
        background: #ebf4f8;
    }
</style>
