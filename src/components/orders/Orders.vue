<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="orders"
      :loading="loading"
      class="elevation-1"
      :no-data-text="$t('orders.empty')"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.totalProducts }}</td>
        <td class="text-xs-center">{{ props.item.totalCost }} €</td>
        <td class="text-xs-center">{{ props.item.createdAt }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon class="mx-2" @click="orderDetail(props.item)">
            <v-icon color="pink">call_made</v-icon>
          </v-btn>
        </td>
      </template>
    </v-data-table>
  </div>
</template>

<script>
  import {db} from '@/main';
  export default {
    name: "orders",
    data () {
      return {
        headers: [
          {text: 'ID', value: 'id', align: 'center'},
          {text: this.$t('orders.products_qty'), value: 'totalProducts', align: 'center'},
          {text: this.$t('orders.cost'), value: 'totalCost', align: 'center'},
          {text: this.$t('orders.date'), value: 'createdAt', align: 'center'},
          {text: '', value: 'name', sortable: false}
        ],
        loading: true,
        orders: [],
      }
    },
    mounted () {
      this.loading = true;
      setTimeout(() => {
        db.collection('orders').where('customer_id', '==', this.$store.state.authModule.user.uid)
          .onSnapshot(orderSnapshot => {
            if ( ! orderSnapshot.empty) {
              this.orders = [];
              orderSnapshot.forEach(order => {
                const orderDoc = order.data();
                this.orders.push({
                  id: order.id,
                  products: orderDoc.products,
                  totalProducts: orderDoc.products.length,
                  totalCost: orderDoc.totalCost,
                  createdAt: orderDoc.createdAt
                });
              });
            }
            this.loading = false;
          }, (error) => {
            console.log('listener orders off...');
          })
      }, 1500);
    },
    methods: {
      orderDetail (order) {
        this.$router.push({name: 'OrderDetail', params: {id: order.id}});
      }
    }
  }
</script>
