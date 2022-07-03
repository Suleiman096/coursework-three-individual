<template>
  <div id="app">
    <div class="search-wrapper">
        <label>Search</label>
        <input class="form-control" v-on:input="getfilteredLessons" type="text" v-model="search"
            placeholder="Search title, location.."/><br><br>
            <select v-model="sort_attribute" v-on:change="sortProducts">
            <option value="">-- Sort by ---</option>
            <option value="subject">Subject</option>
            <option value="location">Location</option>
            <option value="price">Price</option>
            <option value="spaces">Availability</option>
        </select>
        <select v-model="sort_order" v-on:change="sortProducts">
            <option value="">-- Sort order ---</option>
            <option value="ascending">Ascending</option>
            <option value="descending">Descending</option>
        </select>
    </div>

    <header>
        <h1 v-text="sitename"></h1>
        <button v-on:click='showCheckout'>
            {{cartItemCount}}
            <span class="fas fa-cart-plus"></span> Checkout
        </button>
    </header>

    <main>
        <div v-if='showProduct'>
            <div v-for="product in products" :key="product.id">
                <lesson-component :product="product" v-on:add-to-cart="addToCart($event)"></lesson-component>
            </div>
        </div>
        <div v-else>
            <checkout-component v-on:checkout="submitForm($event)" v-on:remove-from-cart="removeFromCart($event)" :cartItems="displayCartItems()"></checkout-component>
        </div>
    </main>
  </div>
</template>

<script>
import lessonComponent from './components/lessonComponent.vue'
import checkoutComponent from './components/checkoutComponent.vue'
export default {
  name: 'App',
  components: {
    lessonComponent,
    checkoutComponent
  },
  data() {
    return {
        search: '',
        sitename: 'Lesson Activities',
        products: [],
        sort_order: "",
        sort_attribute: "",
        cart: [],
        showProduct: true,
    }
  },
  created() {
    this.getLessons()
  },
  computed: {
        cartItemCount() {
            return this.cart.length;
        }
    },
  methods: {
     addToCart(product) {
        // this.cart.push(product.id);
        if (product.spaces >= 1) {
            let lessonInCart = false
            if (this.cartItemCount >= 1){
                for (let i = 0; i < this.cart.length; i++) {
                    if (this.cart[i].id == product.id){
                        this.cart[i].spaces += 1
                        lessonInCart = true
                        break
                    }
                }
                if (lessonInCart == false) {
                    let item = {}
                    item.id = product.id
                    item.spaces = 1
                    this.cart.push(item)
                }
            }
            else{
                let item = {}
                item.id = product.id
                item.spaces = 1
                this.cart.push(item)
            }
            product.spaces -= 1
        }
        else{
            product.spaces = 0
        }
    },
    removeFromCart: function(cart_obj){
        for (let i = 0; i < this.products.length; i++) {
            if (this.products[i].id == cart_obj.id) {
                this.products[i].spaces += cart_obj.spaces
            }
        }
        for (let i = 0; i < this.cart.length; i++) {
            if (this.cart[i].id == cart_obj.id) {
                this.cart.splice(i, 1)
            }
        }
    },
    displayCartItems(){
        let cart_items_displayed = []
        for (let i = 0; i < this.cart.length; i++) {
            for (let j = 0; j < this.products.length; j++) {
                if (this.products[j].id == this.cart[i].id) {
                    let cart_item = {}
                    cart_item.id = this.cart[i].id
                    cart_item.subject = this.products[j].subject
                    cart_item.location = this.products[j].location
                    cart_item.image = this.products[j].image
                    cart_item.spaces = this.cart[i].spaces
                    cart_items_displayed.push(cart_item)
                }   
            }
        }
        return cart_items_displayed
    },
    showCheckout() {
        this.showProduct = this.showProduct ? false : true;
    },
    submitForm(userDetails) {
        let order = {
            name: userDetails.name,
            phone: userDetails.phone,
            products: this.cart
        }
        let order_string = (JSON.stringify(order))
        fetch('https://coursework-two-backend.herokuapp.com/collection/orders', {
            method: "POST",
            body: order_string,
            headers: {
                "Content-type": "application/json; charset=UTF-8"
            }
        })
        .then(response => response.json())
        .then(json_response => {
            console.log(json_response)
            this.updateSpacesInDb()
        })
        .catch(err => console.log(err))
    },
    updateSpacesInDb(){
        let updated_spaces = []
        for (let i = 0; i < this.cart.length; i++) {
            for (let j = 0; j < this.products.length; j++) {
                if (this.cart[i].id == this.products[j].id) {
                    let item = {
                        id: this.cart[i].id,
                        spaces: this.products[j].spaces
                    }
                    updated_spaces.push(item)
                }
            }    
        }
        let updated_spaces_string = (JSON.stringify(updated_spaces))
        fetch('https://coursework-two-backend.herokuapp.com/collection/lessons', {
            method: "PUT",
            body: updated_spaces_string,
            headers: {
                "Content-type": "application/json; charset=UTF-8"
            }
        })
        .then(response => response.json())
        .then(response => {
            console.log(response)
            this.cart = []
            alert('Order has been submitted!')
        })
        .catch(err => console.log(err))
    },
    canAddToCart(product) {
        return product.spaces > this.cartCount(product.id);
    },
    cartCount(id) {
        let count = 0;
        for (let i = 0; i < this.cart.length; i++) {
            if (this.cart[i] === id) {
                count++;
            }
        }
        return count;
    },
    getLessons(){
        fetch(`https://coursework-two-backend.herokuapp.com/collection/lessons`)
        .then(res => {
            return res.json()
        })
        .then(data => {
            this.products = data
        })
        .catch(err => {
            this.products = []
            console.log(`unable to get lessons: ${err}`)
        })
    },  
    getfilteredLessons(){
        fetch(`https://coursework-two-backend.herokuapp.com/collection/lessons/filter?search=${this.search}`)
        .then(res => {
            return res.json()
        })
        .then(data => {
            this.products = data
        })
        .catch(err => {
            this.products = []
            console.log(`unable to get lessons: ${err}`)
        })
    },
    sortProducts(){
        if (this.sort_attribute == "location") {
            this.products.sort(this.locationSort)
        }
        else if (this.sort_attribute == "price"){
            this.products.sort(this.priceSort)
        }
        else{
            this.products.sort(this.subjectSort)
        }

        if (this.sort_order == "descending"){
            this.products.reverse()
        }
    },
    subjectSort(a, b){
        if ( a.subject.toLowerCase() < b.subject.toLowerCase()){
            return -1;
        }
        if ( a.subject.toLowerCase() > b.subject.toLowerCase()){
            return 1;
        }
        return 0;
    },
    locationSort(a, b){
        if ( a.location.toLowerCase() < b.location.toLowerCase()){
            return -1;
        }
        if ( a.location.toLowerCase() > b.location.toLowerCase()){
            return 1;
        }
        return 0;
    },
    priceSort(a, b){
        return a.price-b.price;
    }
  }
}
</script>