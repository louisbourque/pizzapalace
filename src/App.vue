<template>
  <div id="app">
    <div class="input__div">
      <label class="input__label"
        >Input:
        <textarea class="input__textarea" v-model="input" />
      </label>
      <div v-if="errors.length" class="error">
        <div v-for="(error, index) in errors" :key="index">
          {{ error }}
        </div>
      </div>
    </div>
    <Receipt v-if="order.length" :order="order" />
  </div>
</template>

<script>
import Receipt from './components/Receipt.vue'

const orderContainsItem = function(order, item) {
  return order.find(
    orderItem =>
      orderItem.sliceSize === item.sliceSize &&
      orderItem.sliceIngredients.length === item.sliceIngredients.length &&
      !orderItem.sliceIngredients.find(
        order_ingredient =>
          !item.sliceIngredients.find(
            ingredient => order_ingredient == ingredient
          )
      )
  )
}

const sliceSizePriceMap = {
  Small: 12,
  Medium: 14,
  Large: 16,
}

const toppingTypePriceMap = {
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

const toppingTypeMap = {
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
      input: `Large - Pepperoni, Cheese,
Medium - Pepperoni, Cheese
Small - Pepperoni, Cheese`,
      errors: [],
    }
  },
  components: {
    Receipt,
  },
  watch: {
    input: function() {
      this.errors = []
    },
  },
  computed: {
    order: function() {
      if (!this.input) {
        return []
      }
      return this.input
        .split('\n')
        .map(line => line.trim())
        .map(line => {
          const [sliceSize, sliceIngredientsString] = line
            .split('-')
            .map(line => line.trim())
          const sliceIngredients = sliceIngredientsString
            ? sliceIngredientsString
                .split(',')
                .map(line => line.trim())
                .filter(ingredient => {
                  // Assumption: on invalid ingredient, warn the user, continue calculation without ingredient
                  const isValid = toppingTypeMap[ingredient]
                  if (!isValid && ingredient !== '') {
                    this.errors.push('Unknown topping: ' + ingredient)
                  }
                  return isValid
                })
            : []
          return { sliceSize, sliceIngredients }
        })
        .reduce((order, item) => {
          // Assumption: combine slices if they have the same size and ingredients
          const existingItem = orderContainsItem(order, item)
          if (existingItem) {
            existingItem.count += 1
          } else {
            order.push({ count: 1, ...item })
          }
          return order
        }, [])
        .filter(item => {
          // Assumption: on invalid size, warn the user, continue calculation without pizza slice
          const isValid = !!sliceSizePriceMap[item.sliceSize]
          if (!isValid) {
            this.errors.push('Unknown size: ' + item.sliceSize)
          }
          return isValid
        })
        .map(item => {
          const price =
            item.count * sliceSizePriceMap[item.sliceSize] +
            item.sliceIngredients
              .map(
                ingredient =>
                  toppingTypePriceMap[toppingTypeMap[ingredient]][
                    item.sliceSize
                  ]
              )
              .reduce((accumulator, value) => accumulator + value, 0)
          return { ...item, price }
        })
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
  display: flex;
  flex-wrap: wrap;
}

.input__div {
  min-width: 400px;
}

.input__label {
  font-weight: bold;
  font-size: 120%;
  padding-right: 2rem;
}

.input__textarea {
  display: block;
  width: 100%;
  height: 6rem;
}

.center {
  text-align: center;
}
.error {
  color: red;
}
</style>
