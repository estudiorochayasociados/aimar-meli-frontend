<template>
  <div class="container">
    <h1 class="title mt-20">Productos ({{ products.length }})</h1>
    <h2 class="subtitle">Listado de productos en tu desarrollo web</h2>
    <hr/>
    <button @click="updateProducts" id="boton" class="button is-success is-fullwidth" >ACTUALIZAR PRODUCTOS DESDE LA WEB</button>
    <hr/>
    <table class="table is-fullwidth mt-20">
      <thead>
        <th>Producto </th>
      </thead>
      <tbody>
        <tr v-for="product in products">
          <td>{{ product.title }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      products: [],
      textButton:"ACTUALIZAR PRODUCTOS DESDE LA WEB"
    };
  },
  async mounted() {
    const products = await axios.get(process.env.apiUrl + "/product", this.$cookies.get("header-token"));
    this.products = products.data;    
  },
  methods: {
    viewDiv: function(id) {
      var element = document.getElementById(id);
      var element = document.getElementById(id);
      element.classList.add("d-block");
    },
    updateProducts : async function() {
      await document.getElementById('boton').classList.add('is-loading');
      await axios.get(process.env.apiUrl + "/product/update-products-with-web", this.$cookies.get("header-token"));
      const products = await axios.get(process.env.apiUrl + "/product", this.$cookies.get("header-token"));
      this.products = products.data;
      await document.getElementById('boton').classList.remove('is-loading');
      await document.getElementById('boton').classList.replace('is-loading','');      
    }
  }
};
</script> 