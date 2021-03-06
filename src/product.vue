<style lang="sass">
  @import './../_variables.scss';

  .product__detail {
    display: flex;
    & > * {
      flex: 1;
    }
  }

  .product__detail--left {
    order: 2;
  }

  .product__detail--right {
    order: 1;
    max-width: $sidebar-width * 2;
    .price {
      font-size: 24px;
    }
  }

  .product__gallery__preview {
    img {
      max-width: 100%;
      height: auto;
    }
  }

  .product__gallery__wrapper {
    display: flex;
    flex-wrap: wrap;
  }

  .product__gallery__item {
    cursor: pointer;
    flex: 1;
    max-width: 25%;
    /* border: 2px solid $link-color; */
    text-align: center;
    &:hover {
      border-color: $hover-color;
    }
  }

  .availableVariations {
    list-style-type: none;
    padding-left: 5px;
    padding-bottom: 5px;
    clear: both;
    overflow: hidden;
    li {
      cursor: pointer;
      display: block;
      padding: 5px 10px;
      margin-bottom: 5px;
      border: 1px solid #ccc;
      &.active, &:hover {
        border: 1px solid #444;
      }
      &.attribute_pa_velikost {
        float: left;
        display: block;
        margin: 10px 10px -5px -5px;
        background-color: #999;
        border: 1px solid #999;
        color: #fff;
        .variation__attribute__label {
          display: none;
        }
        &.active, &:hover {
          border: 1px solid #444;
          background-color: #444;
        }
      }
    }
  }

  .availableVariations--table {
    .variation__image {
      width: 80px;
      img {
        width: 80px;
        height: 80px;
      }
    }
    .variation__description {
      padding-left: 20px;
    }
    .variation__price {
      width: 100px;
    }
    .variation__buy {
      width: 150px;
    }
    .variation__buy__btn {
      padding: 4px 8px;
      font-size: 12px;
    }
  }

  .table-attributes {
    margin-top: 20px;
    td, th {
      vertical-align: top;
      padding: 5px 10px;
    }
    th {
      padding-left: 0;
    }
    ul {
      margin: 0;
      padding: 0;
      list-style-type: none;
    }
  }

  .product__gallery__item--placeholder {
    img {
      max-width: 100%;
      height: auto;
    }
  }
</style>

<template>
  <div class="page content" :class="[product.slug, product.template]">

    <div class="container cols" v-if="product.id">

      <sidebar></sidebar>

      <div class="main">

        <div class="product__detail">

          <div class="product__detail--left">

            <div class="product__gallery__preview" v-if="product.images.large !== false">
              <img :src="product_gallery_preview"
                    :width="product.images.large[1]"
                    :height="product.images.large[2]"
                    :alt="product.title">
            </div>
            <div class="product__gallery__preview product__gallery__preview--placeholder" v-else>
                <img  :src="wp.wc_placeholder"
                      :alt="product.title">
            </div>

            <div class="product__gallery__wrapper">
              <div  v-if="product.images.full !== false" @click="showGallery(product.images.full[0])" class="product__gallery__item">
                <img  :src="product.images.thumbnail[0]"
                      :width="product.images.thumbnail[1]"
                      :height="product.images.thumbnail[2]"
                      :alt="product.title">
              </div>
              <div v-else  class="product__gallery__item product__gallery__item--placeholder">
                <img  :src="wp.wc_placeholder"
                      :alt="product.title">
              </div>

              <div v-for="(image, index) in product.gallery.thumbnail" @click="showGallery(product.gallery.full[index][0])" class="product__gallery__item">
                <img  :src="image[0]"
                      :width="image[1]"
                      :hight="image[2]"
                      :alt="product.title">
              </div>
            </div>

          </div>

          <div class="product__detail--right">

            <h1 class="entry-title">{{ decode(product.title) }}</h1>

            <span v-html="product.formatted_prices.regular" class="price price--regular"></span>

            <ladda @click="addToCart(product)"
                   :stop="added"
                   color="green"
                   laddastyle="contract"
                   autostart="false">
              {{ lang.add_to_cart }}
            </ladda>

            <!-- Variation selection: List -->
            <ol class="availableVariations" v-if="!settings.wyvern_theme_options_woocommerce.variations_table">
              <li v-for="variation in product.availableVariations" v-if="variation.variation_is_active && variation.variation_is_visible" @click="selectVariation(variation)" :class="[{ 'active' : variation.variation_id == selectedVariation }, getAttributeClass(variation.attributes) ]">
                <span v-for="(value, key) in variation.attributes" :class="[ key + '__item' ]">
                  <span class="variation__attribute__label">{{ getAttributeByKey(key) }}</span>
                  <span class="variation__attribute__value">{{ value }}</span>
                </span>
              </li>
            </ol>

            <!-- Attributes table -->
            <table class="table table-attributes">
              <tbody>
                <tr v-for="(attribute, key) in product.attributes" v-if="attribute.length > 0 && isNotVariationAttribute(key, attribute, product.availableVariations)">
                  <th class="text-left">{{ getAttributeByKey(key) }}</th>
                  <td>
                    <ul>
                      <li v-for="value in attribute">
                        {{ value.name }}
                      </li>
                    </ul>
                  </td>
                </tr>
              </tbody>
            </table>

          </div>

        </div>

        <div class="entry-content" v-html="product.content">
        </div>

        <!-- Variation selection: Table -->
        <table class="table availableVariations--table" v-if="settings.wyvern_theme_options_woocommerce.variations_table">
          <tbody>
            <tr v-for="variation in product.availableVariations" v-if="variation.variation_is_active && variation.variation_is_visible" :class="[{ 'active' : variation.variation_id == selectedVariation }, getAttributeClass(variation.attributes) ]">
              <td class="variation__image">
                <img v-if="variation.image_src != ''" :src="variation.image_src">
                <img  v-else
                      :src="product.images.thumbnail[0]"
                      :width="product.images.thumbnail[1]"
                      :height="product.images.thumbnail[2]"
                      :alt="product.title">
              </td>
              <td class="variation__description">
                <span v-for="(value, key) in variation.attributes" :class="[ key + '__item' ]">
                  <span class="variation__attribute__label">{{ getAttributeByKey(key) }}</span>
                  <span class="variation__attribute__value">{{ value }}</span>
                </span>
              </td>
              <td class="variation__price">
                {{ price(variation.display_price) }}
              </td>
              <td class="variation__buy">
                <ladda @click="addVariationToCart(product, variation)"
                       :stop="added"
                       color="green"
                       laddastyle="contract"
                       autostart="false"
                      class="variation__buy__btn">
                  {{ lang.add_to_cart }}
                </ladda>
              </td>
            </tr>
          </tbody>
        </table>

        <component is="levels" :object="product"></component>

      </div>

    </div>

  </div>
</template>

<script>
  const querystring = require('querystring');

  export default {
    mounted() {
      const vm = this;
      window.wyvern.http.get(`${vm.wp.root}api/products/${vm.$route.meta.postId}`).then((response) => {
        vm.product = response.data;
        vm.product_gallery_preview = vm.product.images.large[0];

        if (typeof vm.product.availableVariations !== 'undefined') {
          vm.product.availableVariations.forEach((variation) => {
            Object.keys(variation.attributes).forEach((key) => {
              const attribute = variation.attributes[key];
              const variated = vm.product.defaultAttributesVariation[vm.getAttributeSlugByKey(key)];
              if (attribute === variated) {
                vm.selectVariation(variation);
              }
            });
          });
        }
      });
    },

    data() {
      return {
        product: {
          id: 0,
        },
        product_gallery_preview: null,
        quantity: 1,
        added: false,
        wp: window.wp,
        lang: window.lang,
        selectedVariation: null,
        variation: null,
        settings: window.settings,
      };
    },

    methods: {
      showGallery(url) {
        this.product_gallery_preview = url;
      },
      addToCart(product) {
        const vm = this;

        this.added = false;

        const query = querystring.stringify({
          variation_id: this.selectedVariation,
          quantity: this.quantity,
          variation: JSON.stringify(this.variation),
        });

        window.wyvern.http.post(`${vm.wp.root}api/cart/${product.id}/?${query}`).then(() => {
          window.eventHub.$emit('cart-add', vm.quantity);
          window.eventHub.$emit('message', {
            type: 'success',
            message: 'Zboží bylo úspěšně přidáno do košíku',  // @todo: make lang
          });
          this.added = true;
        });
      },
      addVariationToCart(product, variation) {
        this.selectVariation(variation);
        this.addToCart(product);
      },
      selectVariation(selectedVariation) {
        this.selectedVariation = selectedVariation.variation_id;

        const variation = {};

        Object.keys(selectedVariation.attributes).forEach((key) => {
          const item = selectedVariation.attributes[key];
          variation[this.getAttributeByKey(key)] = item;
        });

        if (typeof selectedVariation.image_src !== 'undefined' && selectedVariation.image_src !== '') {
          this.showGallery(selectedVariation.image_src);
        }

        if (typeof selectedVariation.display_price !== 'undefined') {
          this.product.formatted_prices.regular = this.price(selectedVariation.display_price);
        }

        this.variation = variation;
      },
      getAttributeByKey(key) {
        const attributeKey = key.replace('attribute_pa_', '');
        return this.wp.attributes[attributeKey].label;
      },
      getAttributeSlugByKey(key) {
        return key.replace('attribute_pa_', '');
      },
      getAttributeClass(attributes) {
        return Object.keys(attributes).join(' variations__');
      },
      isNotVariationAttribute(mainAttributeKey, attribute, availableVariations) {
        const variatedAttributes = [];
        const attributeWooPrefix = 'attribute_pa_';

        if (typeof availableVariations !== 'undefined') {
          Object.keys(availableVariations).forEach((key) => {
            const keys = Object.keys(availableVariations[key].attributes);
            Object.keys(keys).forEach((i) => {
              const attributeKey = keys[i];
              variatedAttributes[attributeKey] = attributeKey;
            });
          });
        }

        return typeof variatedAttributes[attributeWooPrefix + mainAttributeKey] === 'undefined';
      },
    },

    route: {
      canReuse() {
        return false;
      },
    },
  };
</script>
