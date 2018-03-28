<template lang="pug">
    .dropdown.is-right(v-bind:class="{'is-active': isActive}")
        .dropdown-trigger
            //button.button(aria-haspopup='true', aria-controls='dropdown-menu2')
                //span Content
                //span.icon.is-small
                    //i.fas.fa-angle-down(aria-hidden='true')
            input.input(@focus="isActive = true" @blur="isActive = false")
        #dropdown-menu2.dropdown-menu(role='menu')
            .dropdown-content
                .columns
                    .column {{month1}} {{year1}}
                    .column {{month2}} {{year2}}
                .columns
                    .column
                        table.table.is-narrow
                            thead
                                tr
                                    th(v-for="day in days") {{day}}
                            tbody
                                tr(v-for="week in m1.weeks")
                                    td(v-for="d in week") {{d}}

                    .column
                        table.table.is-narrow
                            thead
                                tr
                                    th(v-for="day in days") {{day}}
                            tbody
                                tr(v-for="week in m2.weeks")
                                    td(v-for="d in week") {{d}}
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
            m1: {},
            m2: {},
            month1: '',
            year1: '',
            month2: '',
            year2: '',
            isActive: false,
            days: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
        }
    },
    created() {
        console.log("start: " + this.start);
        console.log("format: " + this.format);

        this.m1['weeks'] = [];
        this.m2['weeks'] = [];

        console.log(Moment(this.start, this.format).format('DD-MM-YYYY'));
        var m1 = Moment(this.start, this.format);
        this.month1 = m1.format('MMM');
        this.year1 = m1.format('YYYY');

        var m2 = m1.add(1, 'month');
        this.month2 = m2.format('MMM');
        this.year2 = m2.format('YYYY');

        this.render();
    },
    methods: {
        render: function() {
            //month of the starting date
            var mY1 = Moment().month(this.month1).year(this.year1);
            this.fillCalendar(mY1, this.m1['weeks']);

            //next month
            var mY2 = Moment().month(this.month2).year(this.year2);
            this.fillCalendar(mY2, this.m2['weeks']);
        },
        fillCalendar: function(m, weeks) {
            console.log("m: " + JSON.stringify(m));
            m = m.startOf('month').startOf('week').clone();
            var isToday = false;
            var isOtherMonth = false;
            var isSelected = false;
            //we want a 7x7 grid starting from the sunday of the week for the 1st of month
            for (var i = 0; i < ROWS; i++) {
                var week = []
                for (var j = 0; j < COLUMNS; j++) {
                    week.push(m.format('D'));
                    m.add(1, 'days');
                }
                weeks.push(week);
            }
        }
    }
}
</script>
