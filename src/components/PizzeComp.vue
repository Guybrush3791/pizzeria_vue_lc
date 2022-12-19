<template>
  <div class="hello">
    <h1>Pizze</h1>
    <div
      v-if="!pizza_create_form"
    >
      <button @click="pizza_create_form = true">CREATE NEW PIZZA</button>
    </div>
    <div
      v-else
    >
      <form
        @submit="createPizza"
      >
        <label for="name">Name</label>
        <input type="text" name="name" v-model="pizza_create.name">
        <br> 
        <label for="description">Description</label>
        <input type="text" name="description" v-model="pizza_create.description">
        <br>
        <label for="price">Price</label>
        <input type="number" name="price" v-model="pizza_create.price">
        <br><br>
        <input type="submit" value="CREATE">
        <button @click="editPizza(PIZZE_EDIT_ID_DEFAULT)">CANCEL</button>
      </form>
    </div>
    <ul>
      <li
        v-for="pizza in pizze"
        :key="pizza.id"
      >
        <div
          v-if="pizze_edit_id != pizza.id"
        >
          {{ pizza.name }}
          <br>
          <div
            v-if="!pizza.ingredienti"
          >
            <button @click="getIngredienti(pizza.id)">GET INGREDIENTI</button>
          </div>
          <div
            v-else
          >
            <ul
              v-if="pizza.ingredienti.length > 0"
            >
              <li
                v-for="ingrediente in pizza.ingredienti"
                :key="ingrediente.id"
              >
                {{ ingrediente.name }}
              </li>
            </ul>
          </div>
          <br>
          <button @click="editPizza(pizza.id)">EDIT</button>
          <br>
          <button @click="deletePizza(pizza.id)">DELETE</button>
        </div>
        <div
          v-else
        >
          <form
            @submit="updatePizza"
          >
            <label for="name">Name</label>
            <input type="text" name="name" v-model="pizza.name">
            <br> 
            <label for="description">Description</label>
            <input type="text" name="description" v-model="pizza.description">
            <br>
            <label for="price">Price</label>
            <input type="number" name="price" v-model="pizza.price">
            <br><br>
            <input type="submit" value="UPDATE">
            <button @click="editPizza(PIZZE_EDIT_ID_DEFAULT)">CANCEL</button>
          </form>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>

import axios from 'axios';

const API_URL = "http://localhost:8080/api/1";
const PIZZE_EDIT_ID_DEFAULT = -1;

export default {
  name: 'PizzeComp',
  data() {

    return {

      pizze: [],
      pizze_edit_id: PIZZE_EDIT_ID_DEFAULT,
      pizza_create: { },
      pizza_create_form: false, 
    };
  },
  methods: {

    getPizzaIndexById(id) {

      for (let x=0;x<this.pizze.length;x++) {

        const pizza = this.pizze[x];
        if (pizza.id == id)
          return x;
      }

      return -1;
    },
    getPizzaById(id) {

      return this.pizze[this.getPizzaIndexById(id)];
    },
    createPizza(e) {

      e.preventDefault();

      axios.post(API_URL + "/pizza/create", this.pizza_create)
           .then(res => {
      
            const pizza = res.data;

            if (pizza == null) return;

            this.pizze.push(pizza);
      });
    },
    editPizza(id) {

      this.pizze_edit_id = id;
    },
    updatePizza(e) {

      e.preventDefault();

      const id = this.pizze_edit_id;
      const pizza = this.getPizzaById(id);

      this.editPizza(PIZZE_EDIT_ID_DEFAULT);

      axios.post(API_URL + '/pizza/update/' + id, pizza)
           .then(res => {

            const index = this.getPizzaIndexById(id);
            const oldPizza = this.pizze[index];

            const pizza = res.data;
            pizza.ingredienti = oldPizza.ingredienti;

            this.pizze[index] = pizza;
           });
    },
    getIngredienti(id) {

      axios.get(API_URL + '/ingredienti/by/pizza/' + id)
           .then(res => {
      
            const ingredienti = res.data;
            
            if (ingredienti == null) return;

            const index = this.getPizzaIndexById(id);
            this.pizze[index].ingredienti = ingredienti;
      });
    },
    deletePizza(id) {

      console.log(id);

      axios.get(API_URL + '/pizza/delete/' + id)
           .then(res => {
            
            const deleted = res.data;

            if (!deleted) return;

            const index = this.getPizzaIndexById(id);
            this.pizze.splice(index, 1);
      });
    }
  },
  mounted() {

    axios.get(API_URL + '/pizza/all')
         .then(res => {
     
        const pizze = res.data;
        if (pizze == null) return;

        this.pizze = pizze;
    });
   
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
