<template>
  <div class="receipt">
    <h1 class="center">Pizza Place #902</h1>
    <div class="center"><strong>All sales final</strong></div>
    <ul class="receipt__itemlist">
      <li v-for="(item, index) in order" :key="index">
        {{ item.count }} {{ item.sliceSize }},
        {{ getTextNumber(item.sliceIngredients.length) }} Topping Pizza{{
          formatIngredientList(item.sliceIngredients)
        }}: ${{ item.price.toFixed(2) }}
      </li>
    </ul>
    Subtotal: ${{ subtotal.toFixed(2) }}<br />
    GST: ${{ gst.toFixed(2) }}<br />
    Total: ${{ total.toFixed(2) }}

    <br />
    <br />
    GST Reg.# A1231456248

    <br /><br />
    <div class="center">&lt;&lt;&lt;&lt; customer copy &gt;&gt;&gt;&gt;</div>
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
      // Ensure we always round up
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
          return ' - ' + ingredient
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

<style scoped>
.receipt {
  font-family: monospace;
  border: 1px solid black;
  padding: 0.5rem;
  box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  margin: 0 auto;
}
.receipt__itemlist {
  list-style-type: none;
  padding-left: 0;
}
</style>
