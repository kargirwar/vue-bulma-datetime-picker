<template lang="pug">
    .dropdown.is-right(v-bind:class="{'is-active': isActive}")
        .dropdown-trigger
            input.input(@click="isActive = true")
        #dropdown-menu2.dropdown-menu(role='menu')
            .dropdown-content
                .columns
                    .column.is-1.prevMonth(v-on:click.prevent.stop="prevMonth")
                        i.fas.fa-chevron-left                   
                    .column.is-5.has-text-centered {{month1}} {{year1}}
                    .column.is-5.has-text-centered {{month2}} {{year2}}
                    .column.is-1.nextMonth(@click="nextMonth")
                        i.fas.fa-chevron-right                
                .columns
                    .column
                        table.table.is-narrow
                            thead
                                tr
                                    th(v-for="day in days") {{day}}
                            tbody
                                tr(v-for="week in m1.weeks")
                                    td(
                                        v-bind:class="d.style"
                                        @click="onClick(d)"
                                        @mouseover="highlightRange(d)"
                                        v-for="d in week") {{d.moment.date}}

                    .column
                        table.table.is-narrow
                            thead
                                tr
                                    th(v-for="day in days") {{day}}
                            tbody
                                tr(v-for="week in m2.weeks")
                                    td(
                                        v-bind:class="d.style"
                                        @click="onClick(d)"
                                        @mouseover="highlightRange(d)"
                                        v-for="d in week") {{d.moment.date}}
                .columns
                    .column
                        button.button.is-primary Apply
                        button.button(@click="isActive = false") Cancel
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
        format: {
            type: String,
            required: true
        },
        start: {
            type: String,
            required: true
        },
        end: {
            type: String,
        }
    },
    data () {
        return {
            m1: {
                weeks: []
            },
            m2: {
                weeks: []
            },
            refs: {},
            d1: null,
            d2: null,
            month1: '',
            year1: '',
            month2: '',
            year2: '',
            isActive: false,
            state: F_0_S_0,
            days: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
        }
    },
    created() {
        var m1 = Moment(this.start, this.format);
        this.month1 = m1.format('MMM');
        this.year1 = m1.format('YYYY');

        var m2 = m1.add(1, 'month');
        this.month2 = m2.format('MMM');
        this.year2 = m2.format('YYYY');

        this.render();
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
            switch (this.state) {
            case F_0_S_0:
                this.f0s0(d);
                break;
            case F_1_S_0:
                this.f1s0(d);
                break;
            case F_1_S_1:
                this.f1s1(d);
                break;
            case F_0_S_1:
                return;
            }
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
        prevMonth: function() {
            var mY1 = Moment().month(this.month1).year(this.year1);
            mY1.subtract(1, 'months');
            this.month1 = mY1.format('MMM');
            this.year1 = mY1.format('YYYY');
            this.m1['weeks'] = [];

            var mY2 = Moment().month(this.month2).year(this.year2);
            mY2.subtract(1, 'months');
            this.month2 = mY2.format('MMM');
            this.year2 = mY2.format('YYYY');
            this.m2['weeks'] = [];

            this.render();
        },
        nextMonth: function() {
            var mY1 = Moment().month(this.month1).year(this.year1);
            mY1.add(1, 'months');
            this.month1 = mY1.format('MMM');
            this.year1 = mY1.format('YYYY');
            this.m1['weeks'] = [];

            var mY2 = Moment().month(this.month2).year(this.year2);
            mY2.add(1, 'months');
            this.month2 = mY2.format('MMM');
            this.year2 = mY2.format('YYYY');
            this.m2['weeks'] = [];

            this.render();

        },
        render: function() {
            //month of the starting date
            var mY1 = Moment().month(this.month1).year(this.year1);
            this.fillCalendar(mY1, this.m1['weeks']);

            //next month
            var mY2 = Moment().month(this.month2).year(this.year2);
            this.fillCalendar(mY2, this.m2['weeks']);
        },
        fillCalendar: function(m, weeks) {
            var m = m.startOf('month').startOf('week').clone();
            //we want a 7x7 grid starting from the sunday of the week for the 1st of month
            for (var i = 0; i < ROWS; i++) {
                var week = []
                for (var j = 0; j < COLUMNS; j++) {
                    var d = {
                        moment: m.toObject(),
                        style: this.getStyle(m)
                    };
                    week.push(d);
                    this.refs[m.format(REF_FORMAT)] = d;
                    m.add(1, 'days');
                }
                weeks.push(week);
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
