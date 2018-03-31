<template lang="pug">
    .dropdown.is-right(v-bind:class="{'is-active': isActive}")
        .dropdown-trigger
            input.input(@click="isActive = true")
        #dropdown-menu2.dropdown-menu(role='menu')
            .dropdown-content
                .columns
                    .column.is-1.prevMonth(@click="prevMonth")
                        i.fas.fa-chevron-left                   
                    .column.is-5.has-text-centered {{month1}} {{year1}}
                    .column.is-5.has-text-centered {{month2}} {{year2}}
                    .column.is-1.nextMonth(@click="nextMonth")
                        i.fas.fa-chevron-right                
                .columns
                    .column
                        calendar(
                            :mY="mY1"
                            :store="store"
                            v-on:dateSelected="onDateSelect"
                        )
                    .column
                        calendar(
                            :mY="mY2"
                            :store="store"
                            v-on:dateSelected="onDateSelect"
                        )
                .columns
                    .column
                        button.button.is-primary Apply
                        button.button(@click="isActive = false") Cancel
</template>

<script>
import Moment from 'moment';
import Calendar from "./Calendar.vue";

const ROWS = 6;
const COLUMNS = 7;
const F_0_S_0 = "f-0-s-0";
const F_1_S_0 = "f-1-s-0";
const F_1_S_1 = "f-1-s-1";
const F_0_S_1 = "f-1-s-1";//invalid state

export default {
    props: {
        format: {
            type: String,
        },
        start: {
            type: String,
        },
        end: {
            type: String,
        }
    },
    data () {
        return {
            mY1: null,
            mY2: null,
            isActive: false,
            store: {
                d1: null,
                d2: null,
                state: F_0_S_0
            }
        }
    },
    components: {
        'calendar': Calendar,
    },
    created() {
        //if start date has been provided setup first calendar based on that
        var m;
        if (this.start) {
            if (!this.format) {
                //if start date is provided format must be provided too
                throw "Format not specified";
            }

            m = Moment(this.start, this.format);
            this.mY1 = m.toObject()
        } else {
            m = Moment();
            this.mY1 = m.toObject();
        }

        //second calendar is always next month
        this.mY2 = m.add(1, 'months').toObject();
    },
    methods: {
        prevMonth: function() {
            console.log("prevMonth");
            this.mY1 = Moment(this.mY1).subtract(1, 'months').toObject();
            this.mY2 = Moment(this.mY2).subtract(1, 'months').toObject();
        },
        nextMonth: function() {
            console.log("nextMonth");
            this.mY1 = Moment(this.mY1).add(1, 'months').toObject();
            this.mY2 = Moment(this.mY2).add(1, 'months').toObject();
        },
        onDateSelect: function(m) {
            console.log(m.format('DD-MMM'));
            switch (this.store.state) {
            case F_0_S_0:
                this.f0s0(m);
                break;

            case F_1_S_0:
                this.f1s0(m);
                break;

            case F_1_S_1:
                this.f1s1(m);
                break;
            }
        },
        f0s0: function(m) {
            //initial click
            var o = this.store;
            Object.assign(o, {d1: m, state: F_1_S_0});
        },
        f1s0: function(m) {
            //one date has been selected
            var o = this.store;
            if (m.isSameOrBefore(o.d1)) {
                Object.assign(o, {d1: m, d2: null, state: F_1_S_0});
                return;
            }
            Object.assign(o, {d2: m, state: F_1_S_1});
        },
        f1s1: function(m) {
            //both dates have been selected. A new selection
            var o = this.store;
            Object.assign(o, {d1: m, d2: null, state: F_1_S_0});
        }
    },
    computed: {
        month1: function() {
            return Moment(this.mY1).format('MMM');
        },
        year1: function() {
            return Moment(this.mY1).format('YYYY');
        },
        month2: function() {
            return Moment(this.mY2).format('MMM');
        },
        year2: function() {
            return Moment(this.mY2).format('YYYY');
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
