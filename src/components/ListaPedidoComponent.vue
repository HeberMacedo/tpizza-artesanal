<template>
  <div>
    <alerta-component-vue :tipo="alerta.tipo" :mensagem="alerta.mensagem" />

    <div id="pedidos-tabela">
      <div>
        <div id="pedidos-tabela-cabecalho">
          <div id="ordem-id">#ID</div>
          <div>Cliente</div>
          <div>Pizza</div>
          <div>Tamanho</div>
          <div>Sabores e itens</div>
          <div>Status</div>
          <div id="div-acoes">Acoes</div>
        </div>
      </div>
    </div>

    <div
      class="pedidos-tabela-linha"
      v-for="pedido in listaPedidosRealizados"
      :key="pedido.id"
    >
      <div id="ordem-numero">{{ pedido.id }}</div>
      <div>{{ pedido.nome }}</div>
      <div>{{ pedido.pizza.nome }}</div>
      <div>{{ pedido.tamanho.descricao }}</div>
      <div>
        <strong>Sabores</strong>
        <ul>
          <li v-for="(sabor, index) in pedido.sabores" :key="index">
            {{ sabor.nome }}
          </li>
        </ul>
        <div v-if="pedido.borda">
          <strong>Borda</strong>
          <p>{{ pedido.borda.nome }}</p>
        </div>
        <div class="divider"></div>
        <strong>Bebidas</strong>
        <ul>
          <li v-for="(bebida, index) in pedido.bebidas" :key="index">
            {{ bebida.nome }}
          </li>
        </ul>
      </div>
      <div>
        <select
          @change="atualizarStatusPedido($event, pedido.id)"
          name="status"
          class="status"
        >
          <option value="">Selecione</option>
          <option
            v-for="status in listaStatusPedido"
            :key="status.id"
            :value="status.id"
            :selected="status.id == pedido.statusId"
          >
            {{ status.descricao }}
          </option>
        </select>
      </div>
      <div id="div-acoes">
        <button class="botao-excluir" @click="deletarPedido(pedido.id)">
          X
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import AlertaComponentVue from "@/components/AlertaComponent.vue";

export default {
  name: "ListaPedidoComponent",
  components: {
    AlertaComponentVue,
  },
  data() {
    return {
      listaPedidosRealizados: [],
      listaStatusPedido: [],
      alerta: {
        tipo: "info",
        mensagem: "Acompanhe os pedidos cadastrados e atualize o status da producao.",
      },
    };
  },
  methods: {
    exibirAlerta(tipo, mensagem) {
      this.alerta = { tipo, mensagem };
    },
    async consultarPedidos() {
      const response = await fetch(`${this.$apiUrl}/pedidos`);
      this.listaPedidosRealizados = await response.json();
    },
    async consultarStatusPedido() {
      const response = await fetch(`${this.$apiUrl}/status_pedido`);
      this.listaStatusPedido = await response.json();
    },
    async atualizarStatusPedido(event, idPedido) {
      const idPedidoAtualizado = event.target.value;

      if (!idPedidoAtualizado) {
        this.exibirAlerta("aviso", "Selecione um status valido para atualizar o pedido.");
        return;
      }

      const atualizacaoJson = JSON.stringify({ statusId: Number(idPedidoAtualizado) });
      const response = await fetch(`${this.$apiUrl}/pedidos/${idPedido}`, {
        method: "PATCH",
        headers: { "Content-type": "application/json" },
        body: atualizacaoJson,
      });

      if (response.ok) {
        this.exibirAlerta("sucesso", "Status do pedido atualizado com sucesso.");
        await this.consultarPedidos();
      } else {
        this.exibirAlerta("erro", "Nao foi possivel atualizar o status do pedido.");
      }
    },
    async deletarPedido(idPedido) {
      const response = await fetch(`${this.$apiUrl}/pedidos/${idPedido}`, {
        method: "DELETE",
      });

      if (response.ok) {
        this.listaPedidosRealizados = this.listaPedidosRealizados.filter(
          (pedido) => pedido.id !== idPedido
        );
        this.exibirAlerta("sucesso", "Pedido excluido com sucesso.");
      } else {
        this.exibirAlerta("erro", "Nao foi possivel excluir o pedido.");
      }
    },
  },
  mounted() {
    this.consultarPedidos();
    this.consultarStatusPedido();
  },
};
</script>

<style scoped>
#pedidos-tabela {
  width: 100%;
  margin: 0 auto;
}

#pedidos-tabela-cabecalho,
#pedidos-tabela-linhas,
.pedidos-tabela-linha {
  display: flex;
  flex-wrap: wrap;
}

#pedidos-tabela-cabecalho {
  font-weight: bold;
  padding: 12px;
  border-bottom: 2px solid #222;
}

#pedidos-tabela-cabecalho div,
.pedidos-tabela-linha div {
  width: 18%;
}

.pedidos-tabela-linha {
  width: 100%;
  padding: 12px;
  border-bottom: 1px dotted #222;
  text-align: left;
}

.pedidos-tabela-linha ul {
  padding-left: 18px;
}

.pedidos-tabela-linha p {
  margin: 6px 0;
}

#pedidos-tabela-cabecalho #ordem-id,
.pedidos-tabela-linha #ordem-numero,
.pedidos-tabela-linha #div-acoes,
#pedidos-tabela-cabecalho #div-acoes {
  width: 5%;
}

.status {
  width: 150px;
  min-height: 40px;
}

.botao-excluir {
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 8px;
  background: #dc2626;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.botao-excluir:hover {
  background: #7f1d1d;
}
</style>
