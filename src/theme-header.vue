<style lang="sass">
  @import '../_variables.scss';

  .theme-header {
    background-color: $header-background;
    line-height: $header-height;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 3;
  }

  .page.content {

  }

  .sidebar {
    position: fixed;
    padding-top: 0;
    height: 100%;
    top: 0;
  }

  .filters {
    display: flex;
    flex-wrap: wrap;
    padding: 20px 0;
  }

  .filter {
    flex: 1;
    min-width: 25%;
    max-width: 25%;
    padding: 0 0 20px 2px;
  }

  .site-title {
    height: $header-height;
    a {
      text-decoration: none;
      font-size: 36px;
      font-weight: 700;
      color: $brand-primary;
    }
  }

  .site-header {
    display: flex;
    background-color: $header-background;
    position: relative;
    z-index: 2;

    & > * {
      flex: 1;
    }
    .site-title {
      max-width: $sidebar-width;
    }
  }

  .nav {
    list-style-type: none;
    float: right;
    text-align: right;
    padding-left: 0;
    margin: 0;
    li {
      display: inline-block;
      a {
        line-height: $header-height;
        text-decoration: none;
        padding: 0 25px;
        cursor: pointer;
        user-select: none;
      }
    }
  }

  .nav, .nav__link {
    font-size: 12px;
    letter-spacing: 0.6px;
    text-transform: uppercase;
  }

  .nav__item__search {
    position: relative;
    &:after {
     content: "";
     display: block;
     width: 0;
     height: 0;
     border-left: 5px solid transparent;
     border-right: 5px solid transparent;
     border-top: 5px solid $link-color;
     position: absolute;
     top: ($header-height - 5px)/2;
     right: 5px;
    }
  }
</style>

<template>
  <div class="theme-header">
    <div class="container site-header">
      <div class="site-title" @click="clearFilters()">
        <router-link :to="{ path: wp.base_path }" v-show="!wp.display.show_logo">{{ wp.site_name }}</router-link>
        <router-link :to="{ path: wp.base_path }" v-show="wp.display.show_logo" class="block">
          <img v-bind:src="wp.assets_path + '/images/logo.svg'"
               v-bind:alt="wp.site_name"
               height="40">
        </router-link>
      </div>
      <div>
        <router-link :to="{ path: wp.base_path }" v-if="$route.path != wp.base_path" class="nav__link">&lt; Zpět na katalog</router-link>
      </div>
      <ul class="nav">
        <li class="nav__item__search">
          <a @click="toggleFilters()">
            Hledat
          </a>
        </li>
        <li v-for="item in menu" class="nav__item">
          <router-link :to="{ path: url2Slug(item.url) }">{{ filterTitle(item.title) }}</router-link>
        </li>
      </ul>
    </div>
    <transition name="slide-up">
      <div class="filters container animated animated--all" v-show="show_filters">
        <div v-for="attribute in wp.attributes" class="filter">
          <select-component :attribute="attribute" v-model="filters[attribute.name]"></select-component>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  let Headroom = require('headroom.js')

  export default {
    mounted() {
      let vm = this
      this.getMenuLocation('primary', (response) => {
        vm.menu = response
      })
      vm.addClass(document.querySelector('body'), 'pinned')

      // Set default filters
      let filters = {}
      for ( let key in this.wp.attributes ) {
        filters[key] = null
      }
      vm.$set(vm, 'filters', filters)

      var headerElement = document.querySelector('.theme-header')
      var headroom  = new Headroom(headerElement, {
        offset: 205,
        tolerance: 5,
        classes: {
          initial: "animated",
          pinned: "slideDown",
          unpinned: "slideUp"
        },
        onPin : function() {
          vm.addClass(document.querySelector('body'), 'pinned')
        },
        onUnpin : function() {
          vm.removeClass(document.querySelector('body'), 'pinned')
        }
      })
      headroom.init()
    },

    data() {
      return {
        wp: window.wp,
        menu: [],
        show_filters: false,
        filters: {}
      }
    },

    methods: {
      toggleFilters(state) {
        if ( typeof state === 'undefined' )
          state = !this.show_filters

        this.show_filters = state
      },
      filterTitle(title) {
        if ( title.indexOf('Nákupní košík') > -1 ) {
          return '(' + this.wp.cart_total + ') Nákupní košík'
        }
        return title
      },
      cartAdd(number) {
        this.wp.cart_total = parseInt(this.wp.cart_total) + parseInt(number)
      },
      cartEmpty() {
        this.wp.cart_total = 0
      },
      cartUpdate(cart, cart_total) {
        this.wp.cart_total = cart_total
      },
      clearFilters() {
        window.eventHub.$emit('filters-changed', {})
      }
    },

    watch: {
      'show_filters' (value) {
        if ( value ) {
          this.addClass(document.querySelector('body'), 'filtering')
        } else {
          this.removeClass(document.querySelector('body'), 'filtering')
        }
      },
      'filters': {
        handler(value) {
          window.eventHub.$emit('filters-changed', value)
        },
        deep: true
      }
    },

    // Create listeners
    created() {
      window.eventHub.$on('cart-add', this.cartAdd)
      window.eventHub.$on('empty-cart', this.cartEmpty)
      window.eventHub.$on('update-cart', this.cartUpdate)
    },

    // It's good to clean up event listeners before
    // a component is destroyed.
    beforeDestroy() {
      window.eventHub.$off('cart-add')
      window.eventHub.$off('empty-cart')
      window.eventHub.$off('update-cart')
    },
  }
</script>