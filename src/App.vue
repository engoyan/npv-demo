<template>
  <v-app :dark="dark">
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm10 md8>
            <v-card class="elevation-1">
              <v-toolbar dark color="primary">
                <v-toolbar-title>NPV Calculator</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-toolbar-items>
                  <v-switch
                    v-model="dark"
                    hide-details single-line
                  ></v-switch>
                </v-toolbar-items>
              </v-toolbar>
              <v-card-text>
                
                <v-layout row wrap>
                  <v-flex xs12 sm4 class="pa-3">
                    <v-layout row wrap>
                      <v-flex xs8>
                        <v-menu
                          ref="menu1"
                          :close-on-content-click="false"
                          v-model="menu"
                          :nudge-right="40"
                          lazy
                          transition="scale-transition"
                          offset-y
                          full-width
                          max-width="290px"
                          min-width="290px"
                        >
                          <v-text-field
                            slot="activator"
                            v-model="dateFormatted"
                            label="Analysis Date"
                            append-icon="event"
                            @blur="date = parseDate(dateFormatted)"
                          ></v-text-field>
                          <v-date-picker v-model="date" no-title @input="menu = false"></v-date-picker>
                        </v-menu>
                      </v-flex>
                      <v-flex xs6>
                        <v-text-field
                          slot="activator"
                          v-model="discountRate"
                          label="Discount Rate"
                          suffix="%"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12>
                        <v-text-field
                          label="Cash Flows"
                          v-model="cashFlows"
                          multi-line
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12>
                        <v-btn color="primary" @click="calculate" block>Calculate</v-btn>
                      </v-flex>
                    </v-layout>
                  </v-flex>
                  <v-flex xs12 sm8 class="pa-3">
                    <npv-table :data="data" />
                  </v-flex>
                </v-layout>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>

      <v-snackbar
        :timeout="10000"
        color="error"
        multi-line
        vertical
        v-model="snackbar"
      >
        There is an error in calucation with the input data. 
        Make sure every Cash Flow transaction is a new line 
        with a following format "{TransationDate} {CashFlow}"
        <v-btn dark flat @click.native="snackbar = false">Close</v-btn>
      </v-snackbar>

    </v-content>
  </v-app>
</template>

<script>
  var moment = require('moment')
  import NpvTable from './NpvTable'

  export default {
    data: () => ({
      snackbar: false,
      dark: false,
      dateFormatted: "04/01/2018",
      menu: false,
      date: "2018-04-01",
      discountRate: 4,
      cashFlows: "1/1/2019 -5,000.00\n1/1/2020 1,000.00\n1/1/2021 1,000.00\n1/1/2022 1,000.00\n1/1/2023 4,000.00",
      data: []
    }),

    components: {
      NpvTable
    },

    computed: {
      computedDateFormatted () {
        return this.formatDate(this.date)
      }
    },

    watch: {
      date (val) {
        this.dateFormatted = this.formatDate(this.date)
      }
    },

    mounted (){
      this.calculate()
    },

    methods: {
      calculate () {
        try {

          const flow = this.cashFlows.split("\n")
          this.data = flow.map(transaction => {
            const [date, cashFlow] = transaction.split(' ')
            const d = moment(date).diff(moment(this.date), 'days')
            const pv = parseFloat(cashFlow.replace(',','')) / Math.pow(1 + parseInt(this.discountRate)/100, d/365)
            return {
              date, 
              cashFlow,
              pv: Math.round(pv * 100) / 100
            }
          })
          
        } catch (error) {
          this.snackbar = true
        }
      },
      formatDate (date) {
        if (!date) return null

        const [year, month, day] = date.split('-')
        return `${month}/${day}/${year}`
      },
      parseDate (date) {
        if (!date) return null

        const [month, day, year] = date.split('/')
        return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
      }
    }


  }
</script>