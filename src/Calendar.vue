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
                    @mouseover="onMouseOver(d)"
                    @mouseout="onMouseOut(d)"
                    v-for="d in week") {{d.moment.format('D')}}

</template>

<script>
import Moment from 'moment';
import {EventBus} from './EventBus.js';

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
        },
        id: {
            type: Number,
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
        console.log(this.id);
        this.fillCalendar();
        EventBus.$on('mouse-in', (o) => {
            console.log(JSON.stringify(o));
            this.updateRange(o.moment);
        });
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
        onMouseOver: function(d) {
            EventBus.$emit('mouse-in', {
                id: this.id,
                moment: d.moment
            });
        },
        onMouseOut: function(d) {
        },
        clearRange: function() {
            for (var i = 0; i < this.weeks.length; i++) {
                for (var j = 0; j < this.weeks[i].length; j++) {
                    var c = this.weeks[i][j];
                    c.style.range = false;
                }
            }
        },
        updateRange: function(m) {
            if (!m) {
                return;
            }

            switch (this.store.state) {
                case F_0_S_0:
                case F_1_S_1:
                    return;
            }

            //highlight everything from d1 upto m
            for (var i = 0; i < this.weeks.length; i++) {
                for (var j = 0; j < this.weeks[i].length; j++) {
                    var c = this.weeks[i][j];
                    if (c.moment.isAfter(this.d1) && c.moment.isSameOrBefore(m) && c.moment.isSame(this.mY, 'month')) {
                        c.style.range = true;  
                    } else {
                        c.style.range = false;
                    }
                }
            }
        },
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

            case F_1_S_1:
                this.f1s1(n, o);
                break;
            }
        },
        select: function(selected) {
            var dates = [this.d1, this.d2];
            for (var i = 0; i < dates.length; i++) {
                var d = dates[i];
                if (d) {
                    d = this.refs[d.format(REF_FORMAT)];
                    if (d) {
                        //select only if this date is part of this month
                        if (d.moment.isSame(this.mY, 'month')) {
                            d.style.selected = selected;
                        }
                    }
                }
            }
        },
        f1s0: function(n, o) {
            //unselect older ones if any
            this.select(false);
            //save the new d1
            this.d1 = n.d1;
            this.d2 = n.d2;
            this.select(true);
            this.clearRange();
        },
        f1s1: function(n, o) {
            //unselect older ones if any
            this.select(false);
            //save the new d1
            this.d1 = n.d1;
            this.d2 = n.d2;
            this.select(true);
            var d = this.refs[this.d2.format(REF_FORMAT)];
            if (d) {
                this.updateRange(d.moment);
                d.style.range = false;
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

            if (!m.isSame(this.mY, 'month')) {
                return style;
            }

            switch (this.store.state) {
            case F_1_S_0:
                if (m.isSame(this.store.d1)) {
                    style.selected = true;
                }
                break;

            case F_1_S_1:
                if (m.isSame(this.store.d1) || m.isSame(this.store.d2)) {
                    style.selected = true;
                }

                if (m.isAfter(this.store.d1) && m.isBefore(this.store.d2)) {
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
