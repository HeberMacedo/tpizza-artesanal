<template>
  <div>
    <h1>Cardapio</h1>
    <div id="scroll-horizontal">
      <div id="card-content" v-for="pizza in listaMenuPizzas" :key="pizza.id">
        <div id="card-linha">
          <div class="foto-pizza">
            <img :src="pizza.foto" />
            <div class="card-coluna">
              <p id="nome-content">{{ pizza.nome }}</p>
              <p id="preco-content">R$ {{ pizza.valor }},00</p>
              <p id="descricao-content">{{ pizza.descricao }}</p>
              <button @click="selecionarPizza(pizza)">Selecionar</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MenuView",
  data() {
    return {
      listaMenuPizzas: [],
    };
  },
  methods: {
    async consultarMenu() {
      const response = await fetch(`${this.$apiUrl}/menu`);
      const dados = await response.json();
      this.listaMenuPizzas = dados.pizzas;
    },
    selecionarPizza(pizzaSelecionada) {
      const param = JSON.stringify(pizzaSelecionada);
      const pizzaJson = encodeURIComponent(param);
      this.$router.push({
        path: "/config-pedido",
        query: { pizza: pizzaJson },
      });
    },
  },
  mounted() {
    this.consultarMenu();
  },
};
</script>

<style scoped>
#card-content {
  display: inline-block;
  width: 280px;
  min-height: 500px;
  margin: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 8px #444;
  position: relative;
}

#scroll-horizontal {
  flex: 1;
  overflow-x: auto;
  white-space: nowrap;
  width: min(900px, 100%);
  margin: 0 auto;
  box-shadow: inset -10px 0px 15px -20px grey;
}

#nome-content {
  font-size: 30px;
  font-weight: bold;
  text-align: center;
  width: 100%;
  margin: 12px;
}

.foto-pizza {
  flex-shrink: 0;
}

.foto-pizza img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

#preco-content {
  font-size: 35px;
  font-weight: bold;
  text-align: center;
  color: #15803d;
}

#descricao-content {
  font-size: 16px;
  text-align: left;
  color: #555;
  margin: 16px;
  white-space: pre-line;
  text-overflow: ellipsis;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 6;
  -webkit-box-orient: vertical;
}

.card-linha {
  display: flex;
  flex-direction: row;
  height: 100%;
}

.card-coluna {
  flex-grow: 1;
  padding: 15px;
  height: 100%;
}

.card-coluna button {
  padding: 10px;
  width: 100%;
  border-radius: 8px;
  border: none;
  color: white;
  background-color: #15803d;
  cursor: pointer;
  transition: 0.5s;
}

.card-coluna button:hover {
  color: rgb(202, 246, 202);
  background-color: #14532d;
}
</style>
