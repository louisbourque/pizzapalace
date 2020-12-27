<template>
  <div id="app">
    <label class="input__label"
      >Input:
      <textarea class="input__textarea" v-model="input" />
    </label>
    <Receipt v-if="order.length" :order="order" />
  </div>
</template>

<script>
import Receipt from './components/Receipt.vue'

const orderContainsItem = function(order, item) {
  return order.find(
    orderItem =>
      orderItem.slice_size === item.slice_size &&
      orderItem.slice_ingredients.length === item.slice_ingredients.length &&
      !orderItem.slice_ingredients.find(
        order_ingredient =>
          !item.slice_ingredients.find(
            ingredient => order_ingredient == ingredient
          )
      )
  )
}

const sizePrices = {
  Small: 12,
  Medium: 14,
  Large: 16,
}

const toppingPrice = {
  Basic: {
    Small: 0.5,
    Medium: 0.75,
    Large: 1.0,
  },
  Deluxe: {
    Small: 2,
    Medium: 3,
    Large: 4,
  },
}

const toppingType = {
  Cheese: 'Basic',
  Pepperoni: 'Basic',
  Ham: 'Basic',
  Pineapple: 'Basic',
  Sausage: 'Deluxe',
  'Feta Cheese': 'Deluxe',
  Tomatoes: 'Deluxe',
  Olives: 'Deluxe',
}

export default {
  name: 'App',
  data: function() {
    return {
      input: '',
    }
  },
  components: {
    Receipt,
  },
  computed: {
    order: function() {
      return this.input
        ? this.input
            .split('\n')
            .map(line => line.trim())
            .map(line => {
              const [slice_size, slice_ingredients_string] = line
                .split('-')
                .map(line => line.trim())
              const slice_ingredients = slice_ingredients_string
                ? slice_ingredients_string
                    .split(',')
                    .map(line => line.trim())
                    .filter(ingredient => toppingType[ingredient])
                : []
              return { slice_size, slice_ingredients }
            })
            .reduce((order, item) => {
              const existingItem = orderContainsItem(order, item)
              if (existingItem) {
                existingItem.count += 1
              } else {
                order.push({ count: 1, ...item })
              }
              return order
            }, [])
            .filter(item => !!sizePrices[item.slice_size])
            .map(item => {
              let price =
                item.count * sizePrices[item.slice_size] +
                item.slice_ingredients
                  .map(
                    ingredient =>
                      toppingPrice[toppingType[ingredient]][item.slice_size]
                  )
                  .reduce((accumulator, value) => accumulator + value, 0)
              return { ...item, price }
            })
        : []
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.input__label {
  font-weight: bold;
  font-size: 120%;
}

.input__textarea {
  display: block;
  width: 40%;
  height: 6rem;
}
</style>
