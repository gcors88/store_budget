<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <h3 class="ml-1 mt-2">Categorias</h3>
      <v-list>
        <v-list-item
          router
          exact
          @click="searchByCategoryId(0)"
        >
          <v-list-item-content>
            <v-list-item-title>Todas</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item
          v-for="(item, i) in categories"
          :key="i"
          router
          exact
          @click="searchByCategoryId(item.id)"
        >
          <v-list-item-content>
            <v-list-item-title>{{ item.name }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar id="app-bar" :clipped-left="clipped" fixed app :height="80">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <NuxtLink to="/" style="text-decoration: none; color:white">
        Inicio
      </NuxtLink>
      &nbsp;
      &nbsp;
      <NuxtLink to="/inspire" style="text-decoration: none; color:white">
        Sobre
      </NuxtLink>
      &nbsp;
      &nbsp;
      <NuxtLink to="/" style="text-decoration: none; color:white">
        Fornecedores
      </NuxtLink>
      &nbsp;
      &nbsp;
      <NuxtLink to="/" style="text-decoration: none; color:white">
        Localização
      </NuxtLink>
      &nbsp;
      &nbsp;
      <NuxtLink to="/" style="text-decoration: none; color:white">
        Contato
      </NuxtLink>
      <v-spacer />
      <v-btn v-if="window.width > 600" @click.stop="rightDrawer = !rightDrawer">
        <v-icon>mdi-cart</v-icon> Total de Itens: {{ totalItemsInCart }}
      </v-btn>
      <v-btn v-else icon @click.stop="rightDrawer = !rightDrawer">
        <v-icon>mdi-cart</v-icon> {{ totalItemsInCart }}
      </v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <v-container>
          <v-row>
            <v-col cols="8" sm="8">
              <v-img
                :width="200"
                :height="200"
                :max-width="200"
                :max-height="200"
                aspect-ratio="16/9"
                cover
                src="/img/generic.png"
              ></v-img>
            </v-col>
            <v-col cols="4" sm="4">
              <v-text-field v-model="search" label="Buscar Produtos" prepend-icon="mdi-magnify"></v-text-field>
            </v-col>
          </v-row>
        </v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-navigation-drawer v-model="rightDrawer" :width="350" :right="right" temporary fixed>
      <v-list>
        <v-list-item v-for="product, index in cartProducts" :key="index" style="background-color: green; border-radius: 12px; margin-bottom: 15px; margin-left: 6px; margin-right: 6px;">
          <v-list-item-title class="text-wrap">
            Produto: {{ product.description }} - Quantidade: {{ product.total }}
            <v-btn icon @click.stop="removeProductFromCart(index)">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-list-item-title>
        </v-list-item>
      </v-list>
      <v-col v-if="totalItemsInCart" class="mt-3" style="text-align: center;" cols="auto">
        <v-btn 
          id="btn-cart"
          color="success" 
          variant="outlined" 
          @click="sendBudget()"
        >
          Solicitar Orçamento
          <v-icon>mdi-whatsapp</v-icon>
        </v-btn>
      </v-col>
      <v-col v-else>
        <h4>Carrinho Vazio!</h4>
      </v-col>
    </v-navigation-drawer>
    <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
      <v-spacer />
      <v-icon>mdi-whatsapp</v-icon>
      &nbsp;
      <h5>(00) 99999-9999</h5>
      &nbsp;
      <a href="https://www.linkedin.com/in/gcolliveira/" target="_blank">
        <v-icon>mdi-linkedin</v-icon>
      </a>
    </v-footer>
  </v-app>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'DefaultLayout',
  data() {
    return {
      search:'',
      totalItemsInCart: 0,
      cartProducts: [],
      clipped: false,
      drawer: false,
      fixed: false,
      categories: [],
      items: [
        {
          icon: 'mdi-menu',
          title: 'Categorias',
          to: '/',
        },
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      categoryIdParam: 0, 
      window: {
          width: 0,
          height: 0
      },
      isMobile: ''
    }
  },

  computed: {
    ...mapGetters('categories', ['getCategories']),
  },

  watch: {
    search(text) {
      this.$nuxt.$emit('searchEvent', text)
    }
  },

  created() {
    this.$nuxt.$on('addProductInCart', ($event) => this.addProductInCart($event))
  },

  destroyed() {
    window.removeEventListener('resize', this.handleResize);
  },
  
  mounted() {
    this.categories = this.getCategories
    window.addEventListener('resize', this.handleResize);
    this.handleResize();
    this.isMobile = navigator.userAgentData.mobile
  },

  methods: {
    handleResize() {
        this.window.width = window.innerWidth;
        this.window.height = window.innerHeight;
    },

    searchByCategoryId(categoryId) {
      this.$nuxt.$emit('searchByCategoryId', categoryId)
    },

    isInBasePath() {
      return this.$route.path === '/'
    },

    addProductInCart(product) {
      if(!this.cartProducts.length) {
        this.cartProducts.push({
          ...product,
          total: 1
        })

        this.totalItemsInCart++
      } else {
        const indexOfProductInCart = this.cartProducts.findIndex(productInCart => productInCart.description === product.description)
        const productInCart = this.cartProducts[indexOfProductInCart]

        if(productInCart) {
          this.cartProducts[indexOfProductInCart] = {
            ...productInCart,
            total: productInCart.total + 1
          }

          this.totalItemsInCart++
        } else {
          this.cartProducts.push({
          ...product,
          total: 1
          })

          this.totalItemsInCart++
        }
      }
    },

    removeProductFromCart(index) {
      const productInCart = this.cartProducts[index]

      if(productInCart.total > 1) {
        this.cartProducts[index] = {
            ...productInCart,
            total: productInCart.total - 1
          }

          this.totalItemsInCart--  
      } else if(this.cartProducts.length) {
        this.cartProducts.splice(index, 1)
        this.totalItemsInCart--
      }
    },

    sendBudget() {
      const budget = this.cartProducts.reduce((text, product) => `${text}Produto: ${product.description} - Quantidade: ${product.total}\n`, 'Olá gostaria de solicitar o orçamento dos itens: \n');
      const phone = '5511111111111';

      if(this.isMobile) {
        window.open(`whatsapp://send?phone=${encodeURIComponent(phone)}&text=${encodeURIComponent(budget)}`,'_blank')
      } else {
        window.open(`https://wa.me/${encodeURIComponent(phone)}?text=${encodeURIComponent(budget)}`,'_blank')
      }
    }
  }
}
</script>

<style>
@media (max-width: 599px) {
  #app-bar {
    font-size: 10pt;
  }
}
</style>
