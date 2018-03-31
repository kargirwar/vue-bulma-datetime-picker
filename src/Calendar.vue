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
        store: {
            type: Object,
            required: true
        }
    },
    data () {
        return {
            weeks: [],
            refs: {},
            d1: null,
            d2: null,
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
        },
        store: {
            handler(n, o) {
                console.log("Updating");
                console.log("o: " + JSON.stringify(o));
                console.log("n: " + JSON.stringify(n));
                this.update(n, o);
            },
            deep: true
        }
    },
    methods: {
        isEmpty: function(obj) {
            //https://stackoverflow.com/questions/679915/how-do-i-test-for-an-empty-javascript-object
            for(var prop in obj) {
                if(obj.hasOwnProperty(prop))
                return false;
            }

            return JSON.stringify(obj) === JSON.stringify({});
        },
        update: function(n, o) {
            console.log("state: " + n.state);
            switch (n.state) {
            case F_1_S_0:
                this.f1s0(n, o);
                break;
            }
        },
        f1s0: function(n, o) {
            var d;
            if (this.d1) {
                //unselect already selected , if any
                d = this.refs[this.d1.format(REF_FORMAT)];
                if (d) {
                    d.style.selected = false;
                }
            }
            //save the new d1
            this.d1 = n.d1;
            d = this.refs[this.d1.format(REF_FORMAT)];
            console.log("d: " + JSON.stringify(d));
            if (d) {
                d.style.selected = true;
            }
        },
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
            this.$emit('dateSelected', d.moment);
        },
        fillCalendar: function() {
            this.weeks = [];
            this.refs = {};
            var m = Moment(this.mY).startOf('month').startOf('week').clone();

            var isToday = false;
            var isOtherMonth = false;
            var isSelected = false;

            //we want a 7x7 grid starting from the sunday of the week for the 1st of month
            for (var i = 0; i < ROWS; i++) {
                var week = []
                for (var j = 0; j < COLUMNS; j++) {
                    var d = {
                        moment: m.clone(),
                        style: this.getStyle(m)
                    };
                    week.push(d);
                    this.refs[m.format(REF_FORMAT)] = d;
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
