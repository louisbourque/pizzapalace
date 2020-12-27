<template>
  <div class="receipt">
    <h1>Receipt</h1>
    <ul>
      <li v-for="(item, index) in order" :key="index">
        {{ item.count }} {{ item.slice_size }},
        {{ getTextNumber(item.slice_ingredients.length) }} Topping Pizza -
        {{ formatIngredientList(item.slice_ingredients) }}: ${{
          item.price.toFixed(2)
        }}
      </li>
    </ul>
    Subtotal: ${{ subtotal.toFixed(2) }}<br />
    GST: ${{ gst.toFixed(2) }}<br />
    Total: ${{ total.toFixed(2) }}
  </div>
</template>

<script>
const numberMapping = [
  'Zero',
  'One',
  'Two',
  'Three',
  'Four',
  'Five',
  'Six',
  'Seven',
  'Eight',
  'Nine',
  'Ten',
]

export default {
  name: 'Receipt',
  props: {
    order: { type: Array, required: true },
  },
  computed: {
    subtotal: function() {
      return this.order.reduce(
        (accumulator, item) => accumulator + item.price,
        0
      )
    },
    gst: function() {
      return Math.ceil(this.subtotal * 100 * 0.05) / 100
    },
    total: function() {
      return this.subtotal + this.gst
    },
  },
  methods: {
    getTextNumber: function(number) {
      return numberMapping[number] || number
    },
    formatIngredientList: function(ingredients) {
      return ingredients.reduce((string, ingredient, index) => {
        if (index === 0) {
          return ingredient
        } else if (index === ingredients.length - 1) {
          return string + ' and ' + ingredient
        } else {
          return string + ', ' + ingredient
        }
      }, '')
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
