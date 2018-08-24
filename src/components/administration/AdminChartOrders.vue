<template>
  <v-container fluid grid-list-md>
    <line-chart :chart-data="this.collectionData" :options="options"></line-chart>
  </v-container>
</template>

<script>
  import LineChart from './charts/LineChart';
  import {db} from '@/main';
  import * as _ from 'lodash';
  import moment from 'moment';
  export default {
    components: {
      LineChart
    },
    data () {
      return {
        collectionData: null,
        orders: [],
        options: {
          scales: {
            yAxes: [{
              ticks: {
                stepSize: 1,
                suggestedMin: 0
              }
            }],
            xAxes: [{
              ticks: {
                callback: value => {
                  return moment.unix(value).format('DD/MM/YYYY')
                }
              }
            }]
          },
          tooltips: {
            callbacks: {
              label: (tooltipItem, data) => {
                const money = data.datasets[tooltipItem.datasetIndex].money[tooltipItem.index] || '';
                const label = tooltipItem.yLabel;
                return [
                  `Pedidos: ${label}`, `Ganancias: ${money}`
                ];
              }
            }
          }
        }
      }
    },
    mounted () {
      this.fillData();
    },
    methods: {
      fillData () {
        db.collection('orders').onSnapshot(snapshot => {
          this.orders = [];
          snapshot.forEach(order => {
            const orderData = order.data();
            let date = moment(orderData.createdAt, 'DD/MM/YYYY').unix();
            let existsDay = _.find(this.orders, {label: date});
            if ( ! existsDay) {
              this.orders.push({
                label: date,
                value: 1,
                money: parseFloat(orderData.totalCost)
              });
            } else {
              existsDay.value += 1;
              existsDay.money += parseFloat(orderData.totalCost);
            }
          });

          this.collectionData = {
            labels: Object.values(_.mapValues(_.sortBy(this.orders, 'label'), 'label')),
            datasets: [
              {
                label: 'Pedidos de la tienda',
                backgroundColor: '#f87979',
                data: Object.values(_.mapValues(_.sortBy(this.orders, 'label'), 'value')),
                money: Object.values(_.mapValues(_.sortBy(this.orders, 'label'), 'money')),
              }
            ]
          };
        })
      }
    }
  }
</script>
