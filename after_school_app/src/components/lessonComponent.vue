<template>
  <div>
      <h2 v-text="product.subject"></h2>
      <figure>
          <img width="100" height="100" v-bind:src="product.image">
      </figure>
      <p v-html="product.description"></p>
      <p>Subject: {{product.subject}}</p>
      <p>Location: {{product.location}}</p>
      <p>Price: <strong>£</strong>{{product.price}}</p>
      <p>Space: {{product.spaces}}</p>

      <button v-bind:disabled="disableAddToCart" v-on:click='addToCart'>Add to cart</button>
  </div>
</template>

<script>
export default {
  name: 'lessonComponent',
  props:{
    product: Object
  },
  computed:{
    disableAddToCart(){
        if (this.product.spaces >= 1){
            return false
        }
        return true
    },
  },
  methods:{
      addToCart(){
          this.$emit('add-to-cart', this.product)
      }
  },
}
</script>
