<template>
    <div>
        <h2>Checkout</h2>
            <p>
                <strong>Name:</strong>
                <input v-model="order.name" />
            </p>

            <p>
                <strong>Phone:</strong>
                <input v-model.number="order.phone" type="number" />
            </p>

            <h2>Order Information</h2>
            <p>Name: {{order.name}}</p>
            <p>Phone: {{order.phone}}</p>
            <button v-on:click="submitForm">Checkout</button>

            <h2>Cart Items</h2>
            <div v-for="item in cartItems" :key="item.id">
                <h3 v-text="item.subject"></h3>
                <figure>
                    <img width="100" height="100" v-bind:src="item.image">
                </figure>
                <p>Location: {{item.location}}</p>
                <p>Space: {{item.spaces}}</p>
                <button v-on:click="removeFromCart(item.id, item.spaces)">Remove from cart</button>
            </div>
    </div>
</template>

<script>
export default {
    name: 'checkoutComponent',
    props:{
        cartItems: Array
    },
    data() {
        return {
            order: {
                name: '',
                phone: ''
            },
        }
    },
    methods:{
        removeFromCart(id, spaces){
            this.$emit('remove-from-cart', {id: id, spaces: spaces})
        },
        submitForm(){
            this.$emit('checkout', this.order)
            this.clearCheckoutForm()
        },
        clearCheckoutForm: function(){
            this.order.name = ""
            this.order.phone = ""
        },
    }
}
</script>