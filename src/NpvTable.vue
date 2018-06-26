<template>
  <v-data-table
    :headers="headers"
    :items="data"
    hide-actions
    class="elevation-0"
  >
    <template slot="items" slot-scope="props">
      <td>{{ props.item.date }}</td>
      <td class="text-xs-right">{{ props.item.cashFlow }}</td>
      <td class="text-xs-right">{{ props.item.pv }}</td>
    </template>
    <template slot="footer">
      <td colspan="100%" class="text-xs-right">
        <strong>NPV: {{ npv }}</strong>
      </td>
    </template>
  </v-data-table>
</template>

<script>
  export default {
    props: ['data'],
    data () {
      return {
        headers: [
          {
            text: 'Date',
            align: 'left',
            sortable: true,
            value: 'date'
          },
          { 
              text: 'Cash Flow', 
              align: 'right',
              value: 'cashFlow' 
          },
          { 
              text: 'PV', 
              align: 'right',
              value: 'pv' 
          }
        ]
      }
    },
    computed: {
        npv () {
            if (!this.data.length) {
                return 0
            }
            return Math.round(this.data
                        .map(t => t.pv)
                        .reduce((total, num) => total + num)
                        * 100) / 100
        }
    }
  }
</script>