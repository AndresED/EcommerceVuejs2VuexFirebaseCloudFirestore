<template>
  <v-jumbotron color="blue-grey darken-4" dark height="450px">
    <v-container>
      <v-layout align-center>
        <v-flex>

          <h1 class="display-2">{{ $t('orders.id') }} {{ order.id }}</h1>
          <h3 class="display-1">{{ $t('orders.cost') }} {{ order.totalCost }} €</h3>
          <span class="subheading">{{ $t('orders.date') }} {{ order.createdAt }}</span>

          <v-divider class="my-3" />

          <h4 class="display-2">{{ $t('admin.products')}}</h4>

          <v-divider class="my-3" />

          <div class="title mb-3" v-for="product in order.products">
            {{ product.name }} ({{ product.qty }})
          </div>

          <v-btn
            large
            color="primary"
            class="mx-0"
            @click="$router.push('/orders')"
          >
            {{ $t('orders.return_orders_list') }}
          </v-btn>

        </v-flex>
      </v-layout>
    </v-container>
  </v-jumbotron>
</template>

<script>
  import {db} from '@/main';
	export default {
		name: "order-detail",
    data () {
		  return {
		    order: {}
      }
    },
    mounted () {
		  const orderId = this.$route.params.id;
      db.doc(`orders/${orderId}`).get().then((snapshot) => {
        const orderDoc = snapshot.data();
        this.order = {
          id: snapshot.id,
          products: orderDoc.products,
          totalProducts: orderDoc.products.length,
          totalCost: orderDoc.totalCost,
          createdAt: orderDoc.createdAt
        }
      })
    }
	}
</script>
