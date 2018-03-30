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
            d1: null,
            d2: null,
            month1: '',
            year1: '',
            month2: '',
            year2: '',
            isActive: false,
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
        onClick: function(d) {
            if (this.d1) {
                //unselect already selected one
                this.d1.style.selected = false;
            }
            this.d1 = d;
            this.d1.style.selected = true;
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
                    m.add(1, 'days');
                }
                weeks.push(week);
            }
        },
        getStyle: function(m) {
            var style = {
                today: false,
                selected: false,
            };
            var now = Moment();
            if (m.isSame(now, 'day')) {
                //today
                style.today = true;
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
        color: $info;
    }
</style>
