<template>
  <div class="container">
    <h1 class="title mt-20">Actualizar productos de MercadoLibre ({{ meliTotal }})</h1>
    <h2
      class="subtitle"
    >Aplicá las configuraciones de publicaciones que te gustaría que tus productos cumplan</h2>
    <hr />
    <button
      @click="updateEach"
      id="boton"
      class="button is-success is-fullwidth"
    >MODIFICAR PRODUCTOS A MERCADOLIBRE</button>
    <hr />
    Resultado ({{ total }})
    <div class="row">
      <div class="col-md-12">
        <table class="table is-narrow is-hoverable is-fullwidth">
          <thead>
            <th>Estado</th>
            <th>Producto</th>
            <th>Mensaje</th>
          </thead>
          <tbody>
            <tr v-for="response_ in response">
              <td>
                <div
                  v-bind:class="!response_.error ? 'button is-success ' : 'button is-danger ' "
                >
                  <span class v-if="!response_.error">APROBADO</span>
                  <span v-else>NO APROBADO</span>
                </div>
              </td>
              <td>{{ response_.title }}</td>
              <td>
                <div v-if="!response_.error">
                  <b>Tipo de publicación:</b>
                  {{ response_.type }}
                  <br />
                  <b>Código MercadoLibre:</b>
                  {{ response_.code }}
                  <br />
                  <b>Precio MercadoLibre:</b>
                  {{ response_.price }}
                  <br />
                  <b>Porcentaje de recargo aplicado:</b>
                  {{ response_.percent }}
                </div>
                <div v-else><b>ERROR:</b> {{ response_.error.message }}</div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      products: [],
      response: [],
      total: 0,
      meliTotal: 0
    };
  },
  async mounted() {
    const products = await axios.get(
      process.env.apiUrl + "/product",
      this.$cookies.get("header-token")
    );
    this.products = products.data;
    await this.$data.products.forEach(async product => {
      await product.mercadolibre.forEach(async meli => {
        this.meliTotal += 1;
      });
    });
  },
  methods: { 
    updateEach: async function() {

      document.getElementById("boton").classList.add("is-loading");

      var checkToken = await axios.get(
        process.env.apiUrl + "/mercadolibre/refresh-token"
      );

      let configMeli = await axios.get(
        process.env.apiUrl + "/mercadolibre/config"
      );

      let percentGoldEspecial = configMeli.data[0].gold_especial_percent;
      let percentGoldPro = configMeli.data[0].gold_pro_percent;
      let shipping = configMeli.data[0].shipping;

      if(this.products.lenght === 0) {
        return;
      } else {
        await this.products[0].mercadolibre.map(async meli => {
            let percent =
              meli.type == "gold_especial" ? percentGoldEspecial : percentGoldPro;
              await axios
              .put(
                process.env.apiUrl + "/mercadolibre/item/" + meli.code,
                {
                  item: this.products[0],
                  shipping: shipping,
                  percent: percent,
                  type: meli.type
                },
                this.$cookies.get("header-token")
              )
              .then(r => {
                console.log(r);
                if (r.data.status === 200) {
                  this.total += 1;
                }
                this.response.push(r.data);
              });
          });

          this.products.shift();
          this.updateEach();
        }
    } 
  }
};
</script> 