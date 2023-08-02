<template>
  <div>
    <Message :msg="msg" v-show="msg" />
    <form id="burger-form" @submit="createBurger">
      <div class="input_container">
        <label for="nome">Nome do cliente</label>
        <input type="text" id="nome" name="name" v-model="nome" placeholder="Digite o seu nome">
      </div>
      <div class="input_container">
        <label for="pao">Escolha o pão</label>
        <select name="pao" id="pao" v-model="pao">
          <option value="">Selecione o seu pão</option>
          <option v-for="pao in paes" :key="pao.id" :value="pao.tipo">{{ pao.tipo }}</option>
        </select>
      </div>
      <div class="input_container">
        <label for="carne">Escolha a carne do seu Burger:</label>
        <select name="carne" id="carne" v-model="carne">
          <option value="">Selecione a sua carne</option>
          <option v-for="carne in carnes" :key="carne.id" :value="carne.tipo">{{ carne.tipo }}</option>
        </select>
      </div>
      <div id="opcionais-container" class="input_container">
        <label id="opcionais-title" for="opcionais">Selecione os opcionais:</label>
        <div class="checkbox-container" v-for="opcional in opcionaisdata" :key="opcional.id">
          <input type="checkbox" name="opcionais" v-model="opcionais" :value="opcional.tipo">
          <span>{{ opcional.tipo }}</span>
        </div>
      </div>
      <div class="input_container">
        <input type="submit" class="submit-btn" value="Criar meu Burger!">
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios';
import Message from './Message.vue';

export default {
  name: "BurgerForm",
  data() {
    return {
      paes: null,
      carnes: null,
      opcionaisdata: null,
      nome: null,
      pao: null,
      carne: null,
      opcionais: [],
      status: "Solicitado",
      msg: null,
      nextPedidoNumero: 1
    };
  },
  methods: {
    async getIngredientes() {
      try {
        const response = await axios.get('http://localhost:3000/ingredientes');
        const data = response.data;
        this.paes = data[0].paes;
        this.carnes = data[0].carnes;
        this.opcionaisdata = data[0].opcionais;

        console.log(data);
      } catch (error) {
        console.error('Erro ao obter ingredientes:', error);
      }
    },
    async createBurger(e) {
      e.preventDefault();

      const carneId = this.carnes.find(carne => carne.tipo === this.carne)?.id;
      const paoId = this.paes.find(pao => pao.tipo === this.pao)?.id;
      const opcionaisIds = this.opcionais.map(opcional => {
        const opcionalObj = this.opcionaisdata.find(item => item.tipo === opcional);
        return opcionalObj ? opcionalObj.id : null;
      });

      const statusObj = this.opcionaisdata.find(item => item.tipo === this.status);
      const statusId = statusObj ? statusObj.id : '61a311fa-2d68-11ee-be56-0242ac120002';

      const data = {
        nome: this.nome,
        carne: carneId,
        pao: paoId,
        opcionais: opcionaisIds.filter(id => id !== null),
        status: statusId
      };

      try {
        const response = await axios.post("http://localhost:3000/burgers", data);
        const res = response.data;

        this.msg = `Pedido realizado com sucesso`;

        setTimeout(() => {
          this.msg = "";
        }, 5000);

        this.nome = "";
        this.carne = "";
        this.pao = "";
        this.opcionais = [];
      } catch (error) {
        console.error('Erro ao criar pedido:', error);
        this.msg = "Erro ao criar pedido. Por favor, tente novamente mais tarde.";
      }
    }
  },
  mounted() {
    this.getIngredientes();
  },
  components: { Message }
}
</script>

<style scoped>
#burger-form {
  max-width: 400px;
  margin: 0 auto;
}

.input_container {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

label {
  font-weight: bold;
  margin-bottom: 15px;
  color: #222;
  ;
  padding: 5px 10px;
  border-left: 4px solid #fcba03;
}

input,
select {
  padding: 5px 10px;
  width: 300px;
}

#opcionais-container {
  flex-direction: row;
  flex-wrap: wrap;
}

#opcionais-title {
  width: 100%;
}

.checkbox-container {
  display: flex;
  align-items: flex-start;
  width: 50%;
  margin-bottom: 20px;
}

.checkbox-container span,
.checkbox-container input {
  width: auto;
}

.checkbox-container span {
  margin-left: 6px;
  font-weight: bold;
}

.submit-btn {
  background-color: #222;
  color: #fcba03;
  font-weight: bold;
  border: 2px solid #222;
  padding: 10px;
  font-size: 16px;
  margin: 0 auto;
  cursor: pointer;
  transition: .5s;
}

.submit-btn:hover {
  background-color: transparent;
  color: #222;
}
</style>