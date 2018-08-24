<template>
  <div>
    <v-layout column>
      <v-flex xs12 sm6>

        <v-toolbar color="indigo" dark>
          <v-toolbar-title>{{ $t('shop.product_list') }}</v-toolbar-title>

          <v-spacer />

          <v-menu
            origin="center center"
            transition="scale-transition"
            bottom
          >
            <v-btn color="primary" dark slot="activator">
              {{ $t('shop.products_qty') }} ({{ $store.state.shopModule.limit }})
              {{ $t('shop.total_products') }} ({{ products.length }})
            </v-btn>

            <v-list>
              <v-list-tile v-for="(limit, i) in [2, 4, 6, 8, 10, 20, 50]" :key="i">
                <v-list-tile-title @click="changeLimit(limit)">
                  {{ limit}}
                </v-list-tile-title>
              </v-list-tile>
            </v-list>
          </v-menu>
        </v-toolbar>

        <v-card>
          <v-container fluid grid-list-md>

            <v-layout row wrap>
              <v-flex
                :class="`xs${product.flex}`"
                v-for="product in products"
                :key="product.name"
              >
                <product
                  :product="product"
                  @addToCart="openSheet(product)"
                />
              </v-flex>
            </v-layout>

          </v-container>
        </v-card>

      </v-flex>
    </v-layout>

    <v-layout row>
      <infinite-scroll :handler="fetchProducts" :should-handle="!loading" />

      <v-progress-linear v-if="loading" color="purple" indeterminate />

    </v-layout>

    <product-sheet :product="productSheet" @closeSheet="closeSheet" :sheet="statusSheet" />
  </div>
</template>

<script>
  import InfiniteScroll from 'vue-mugen-scroll';
  import Product from './Product';
  import ProductSheet from './ProductSheet';
	export default {
		name: "products",
    components: {
		  InfiniteScroll, Product, ProductSheet
    },
    data () {
		  return {
		    loading: false,
        productSheet: {},
        statusSheet: false
      }
    },
    destroyed () {
		  this.$store.commit('resetProductsPagination');
    },
    computed: {
		  products () {
		    return this.$store.state.shopModule.products;
      },
      limit () {
		    return this.$store.state.shopModule.limit;
      }
    },
    methods: {
		  openSheet (product) {
		    this.productSheet = product;
		    this.statusSheet = true;
		    this.$store.commit('setActiveProductInCart', product);
      },
      fetchProducts () {
		    if ( !this.$store.state.shopModule.finish) {
		      this.loading = true;
		      setTimeout(() => {
		        this.$store.dispatch('fetchAllProductsPaginate').then(() => {
		          this.loading = false;
            })
          }, 2000);
        }
      },
      changeLimit (limit) {
		    if ( this.limit !== limit) {
		      this.$store.commit('resetProductsPagination');
		      this.$store.commit('updateLimitProductsPaginate', limit);
		      this.fetchProducts();
        }
      },
      closeSheet () {
		    this.statusSheet = false;
      }
    }
	}
</script>
