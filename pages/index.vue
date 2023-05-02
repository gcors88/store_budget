<template>
  <div>
    <v-container class="bg-surface-variant">
      <v-row no-gutters>
        <v-col
          v-for="product, index in products"
          :key="index"
          cols="12"
          sm="3"
        >
          <v-sheet class="ma-2 pa-2">
            <v-img
              height="250"
              cover
              class="bg-grey-lighten-2"
              :src="product.pathImage"
            ></v-img>
            <v-col class="mt-3" style="text-align: center;" cols="auto">
              <h4 class="mb-3">{{ product.description }}</h4>
              <v-btn 
                id="btn-cart"
                color="success" 
                variant="outlined" 
                @click="addProductInCart(product)"
              >
                Adicionar ao Carrinho
                <v-icon>mdi-cart</v-icon>
                <v-overlay
                  v-model="overlay"
                  contained
                  class="align-center justify-center"
                >
                  <v-row>
                    Adicionado ao carrinho para orçamento!
                  </v-row>
                </v-overlay>
              </v-btn>
            </v-col>
          </v-sheet>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'IndexPage',

  data() {
    return {
      products: [],
      categoryId: 0,
      overlay: false,
    }
  },

  computed: {
    ...mapGetters('products', ['getProducts']),
  },

  created() {
    this.$nuxt.$on('searchEvent', ($event) => this.search($event))
    this.$nuxt.$on('searchByCategoryId', ($event) => this.searchByCategoryId($event))
  },

  mounted() {
    this.products = this.getProducts
  },
  
  methods: {
    search(textToSearch) {
      const products = this.getProducts
      if(textToSearch.length >= 1) {
        this.products = products.filter(product => this.validateProductByDescriptionAndCategory(product, textToSearch)) 
      } else {
        this.searchByCategoryId(this.categoryId)
      }
    },

    searchByCategoryId(categoryId) {
      const products = this.getProducts
      this.categoryId = categoryId

      if(categoryId) {
        this.products = products.filter(product => product.categoryId === categoryId)
      } else {
        this.products = products
      }
    },

    validateProductByDescriptionAndCategory(product,textToSearch) {
      const isSameDescription = product.description.toLowerCase().includes(textToSearch.toLowerCase())

      return this.categoryId ? isSameDescription && product.categoryId === this.categoryId  
            : isSameDescription
    },

    addProductInCart(product) {
      this.$nuxt.$emit('addProductInCart', product)
      this.overlay = !this.overlay
      setTimeout(() => {
        this.overlay = !this.overlay
      }, 1000)
    }
  }
}
</script>

<style>
@media (max-width: 600px) {
  #btn-cart {
    font-size: 12pt;
  }
}

@media (min-width: 601px) and (max-width: 1265px) {
  #btn-cart {
    font-size: 6.2pt;
  }
}

@media (min-width: 1266px) {
  #btn-cart {
    font-size: 8.6pt;
  }
}
</style>

