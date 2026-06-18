<template>
  <div>
    <alerta-component-vue :tipo="alerta.tipo" :mensagem="alerta.mensagem" />
    <form id="pedido-form" @submit="criarPedido($event)">
      <div>
        <p id="nome-pizza-content">
          {{ pizza && pizza.nome ? pizza.nome : "--" }}
        </p>
        <img
          id="foto-content"
          :src="pizza && pizza.foto ? pizza.foto : ''"
        />
      </div>
      <div class="inputs" id="form-pedido">
        <label>Nome do Cliente</label>
        <input
          v-model="nomeCliente"
          type="text"
          placeholder="Digite o nome do cliente"
          id="nome-cliente"
        />
      </div>
      <div class="inputs">
        <label>Tamanho da pizza</label>
        <select
          v-model="tamanhoSelecionado"
          name="tamanho-pizza"
          id="tamanho-pizza"
        >
          <option value="" selected>Selecione o tamanho</option>
          <option
            v-for="tamanho in listaTamanhos"
            :key="tamanho.id"
            :value="tamanho"
          >
            {{ tamanho.descricao }}
          </option>
        </select>
      </div>
      <div class="inputs">
        <label id="opcionais-titulo">Monte sua pizza</label>
        <label id="opcionais-subtitulo">Escolha ate 2 sabores</label>

        <div
          v-for="sabor in listaSabores"
          :key="sabor.id"
          class="checkbox-container"
        >
          <input
            type="checkbox"
            :name="sabor.nome"
            :value="sabor"
            v-model="listaSaboresSelecionados"
          />
          <span>{{ sabor.nome }}</span>
        </div>

        <label>Escolha a borda</label>

        <select v-model="bordaSelecionada" name="borda-pizza" id="borda-pizza">
          <option value="">Sem borda recheada</option>
          <option v-for="borda in listaBordas" :key="borda.id" :value="borda">
            {{ borda.nome }}
          </option>
        </select>

        <label>Adicione uma bebida</label>

        <div
          v-for="bebida in listaBebidas"
          :key="bebida.id"
          class="checkbox-container"
        >
          <input
            type="checkbox"
            :name="bebida.nome"
            :value="bebida"
            v-model="listaBebidasSelecionadas"
          />
          <span>{{ bebida.nome }}</span>
        </div>

        <div class="inputs">
          <input type="submit" class="submit-btn" value="Confirmar Pedido" />
        </div>
      </div>
    </form>
  </div>
</template>
<script>
import AlertaComponentVue from "@/components/AlertaComponent.vue";

export default {
  name: "PedidoComponent",
  components: {
    AlertaComponentVue,
  },
  props: {
    pizza: null,
  },
  data() {
    return {
      listaTamanhos: [],
      listaSabores: [],
      listaBordas: [],
      listaBebidas: [],
      nomeCliente: "",
      tamanhoSelecionado: "",
      bordaSelecionada: "",
      listaSaboresSelecionados: [],
      listaBebidasSelecionadas: [],
      alerta: {
        tipo: "info",
        mensagem: "Revise os dados do pedido antes de confirmar.",
      },
    };
  },
  methods: {
    exibirAlerta(tipo, mensagem) {
      this.alerta = { tipo, mensagem };
    },
    async getTamanhos() {
      const response = await fetch(`${this.$apiUrl}/tamanhos`);
      const dados = await response.json();
      this.listaTamanhos = dados;
    },
    async getOpcionais() {
      const response = await fetch(`${this.$apiUrl}/opcionais`);
      const dados = await response.json();
      this.listaSabores = dados.sabores;
      this.listaBordas = dados.bordas;
      this.listaBebidas = dados.bebidas;
    },
    validarPedido() {
      if (!this.pizza || !this.pizza.id) {
        this.exibirAlerta("erro", "Selecione uma pizza no cardapio antes de confirmar o pedido.");
        return false;
      }

      if (!this.nomeCliente.trim()) {
        this.exibirAlerta("erro", "Informe o nome do cliente para continuar.");
        return false;
      }

      if (!this.tamanhoSelecionado) {
        this.exibirAlerta("erro", "Escolha o tamanho da pizza.");
        return false;
      }

      if (this.listaSaboresSelecionados.length === 0) {
        this.exibirAlerta("erro", "Escolha pelo menos um sabor para a pizza.");
        return false;
      }

      if (this.listaSaboresSelecionados.length > 2) {
        this.exibirAlerta("aviso", "A pizza meio-a-meio permite no maximo 2 sabores.");
        return false;
      }

      return true;
    },
    async criarPedido(e) {
      e.preventDefault();

      if (!this.validarPedido()) {
        return;
      }

      const dadosPedido = {
        nome: this.nomeCliente.trim(),
        tamanho: this.tamanhoSelecionado,
        sabores: Array.from(this.listaSaboresSelecionados),
        borda: this.bordaSelecionada,
        bebidas: Array.from(this.listaBebidasSelecionadas),
        pizza: this.pizza,
        statusId: 5,
      };

      const dadosJson = JSON.stringify(dadosPedido);

      const req = await fetch(`${this.$apiUrl}/pedidos`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: dadosJson,
      });

      if (!req.ok) {
        this.exibirAlerta("erro", "Nao foi possivel cadastrar o pedido. Tente novamente.");
        return;
      }

      this.exibirAlerta("sucesso", "Pedido cadastrado com sucesso! Abrindo a tela de monitoramento...");

      setTimeout(() => {
        this.$router.push("/pedidos");
      }, 1200);
    },
  },
  mounted() {
    this.getTamanhos();
    this.getOpcionais();
  },
};
</script>

<style scoped>
#foto-content {
  margin-bottom: 16px;
  border-radius: 16px;
  position: relative;
  z-index: -1;
  justify-content: center;
  width: 100%;
  height: 180px;
  object-fit: cover;
}

#nome-pizza-content {
  font-size: 43px;
  font-weight: bold;
  text-align: start;
  margin-bottom: -90px;
  margin-left: 40px;
  color: antiquewhite;
  padding: 16px;
}

#form-pedido {
  max-width: 750px;
  margin: 0 auto;
}

.inputs {
  display: flex;
  flex-direction: column;
  margin-bottom: 16px;
}

label {
  font-weight: bold;
  margin-bottom: 16px;
  color: #222;
  padding: 5px 12px;
  flex-direction: start;
  display: flex;
  border-left: 4px solid #c2410c;
}

input,
select {
  padding: 12px;
  width: 300px;
  border: solid #222 1px;
  border-radius: 8px;
  height: 20px;
  font-size: 12px;
}

select {
  height: 45px;
}

#opcionais-titulo {
  width: 100%;
}

#opcionais-subtitulo {
  display: flex;
  align-items: flex-start;
  align-content: center;
  width: 100%;
  margin-bottom: 12px;
}

.checkbox-container span {
  margin-left: 6px;
  font-weight: bold;
}

.checkbox-container span,
.checkbox-container input {
  width: auto;
  height: 20px;
}

.submit-btn {
  background-color: #7f1d1d;
  color: white;
  font-weight: bold;
  border: none;
  font-size: 18px;
  border-radius: 12px;
  padding: 16px;
  margin: 0 auto;
  cursor: pointer;
  width: 100%;
  height: auto;
  transition: 0.5s;
}

.submit-btn:hover {
  background-color: #f97316;
  color: #222;
}
</style>
