<template lang="pug">
    .columns
        .column
        .column
        .column
        .column
        .column
        .column
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
                            .column
                                table.table.is-narrow
                                    thead
                                        tr
                                            th(v-for="day in days") {{day}}
                                    tbody
                                        tr(v-for="week in weeks")
                                            td(v-for="d in week") {{d}}

                            .column
                                table.table.is-narrow
                                    thead
                                        tr
                                            th(v-for="day in days") {{day}}
                                    tbody
                                        tr(v-for="week in weeks")
                                            td(v-for="d in week") {{d}}


</template>

<script>
export default {
  name: 'app',
  data () {
    return {
        weeks: [],
        isActive: false,
        days: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
    }
  },
  methods: {
      onClick: function() {
          this.isActive = !this.isActive;
      }
  },
  created() {
      var d = 1;
      for (var i = 0; i < 4; i++) {
          this.weeks[i] = [];
          for (var j = 0; j < 7; j++) {
              this.weeks[i].push(d);
              d++;
          }
      }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

</style>
