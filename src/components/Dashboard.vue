<template>
    <div id="burger-table" v-if="burgers && burgers.length > 0">
        <div>
            <Message :msg="msg" v-show="msg" />
            <div id="burger-table-heading">
                <div class="order-id">#</div>
                <div>Clientes:</div>
                <div>Pão:</div>
                <div>Carne:</div>
                <div>Opcionais:</div>
                <div>Ações</div>
            </div>
        </div>
        <div id="burger-table-rows">
            <div class="burger-table-row" v-for="burger in burgers" :key="burger.id">
                <div class="order-number">{{ burger.numeroPedido }}</div>
                <div>{{ burger.nome }}</div>
                <div>{{ burger.pao.tipo }}</div>
                <div>{{ burger.carne.tipo }}</div>
                <div>
                    <ul>
                        <li v-for="(opcional, index) in burger.opcionais" :key="index">
                            {{ opcional.tipo }}
                        </li>
                    </ul>
                </div>
                <div>
                    <select name="status" class="status" @change="updateBurger($event, burger.id)">
                        <option value="">Selecione</option>
                        <option v-for="s in status" :key="s.id" :value="s.id" :selected="burger.status.id == s.id">
                            {{ s.tipo }}
                        </option>
                    </select>
                    <button class="delete-btn" @click="deleteBurger(burger.id)">Cancelar</button>
                </div>
            </div>
        </div>
    </div>
    <div v-else>
        <h2>Não há pedidos registrados</h2>
    </div>
</template>

<script>
import axios from 'axios';
import Message from './Message.vue';

export default {
    name: "Dashboard",
    data() {
        return {
            burgers: null,
            burger_id: null,
            status: "",
            msg: null,
        };
    },
    methods: {
        async getPedidos() {
            try {
                const response = await axios.get("http://localhost:3000/burgers");
                this.burgers = response.data.map((burger, index) => ({
                    ...burger,
                    numeroPedido: index + 1,
                }));
                this.getStatus();
            } catch (error) {
                console.error("Erro ao buscar pedidos:", error);
            }
        },
        async getStatus() {
            try {
                const response = await axios.get("http://localhost:3000/status");
                this.status = response.data;
            } catch (error) {
                console.error("Erro ao buscar status:", error);
            }
        },
        async deleteBurger(id) {
            try {
                const response = await axios.delete(`http://localhost:3000/burgers/${id}`);
                this.msg = `Pedido Removido com Sucesso!`;
                setTimeout(() => {
                    this.msg = "";
                }, 5000);
                this.getPedidos();
            } catch (error) {
                console.error("Erro ao deletar pedido:", error);
            }
        },
        async updateBurger(event, id) {
            const option = event.target.value;

            const updatedStatus = {
                status: option
            };

            try {
                const response = await axios.patch(`http://localhost:3000/burgers/${id}`, updatedStatus);
                const res = response.data;
                const burger = this.burgers.find(b => b.id === id);
                this.msg = `O Pedido Nº ${burger.numeroPedido} foi atualizado para Status ${res.status.tipo}!`;
                setTimeout(() => {
                    this.msg = "";
                }, 5000);
            } catch (error) {
                console.error("Erro ao atualizar pedido:", error);
                console.log("Response data:", error.response.data);
            }
        }

    },
    mounted() {
        this.getPedidos();
    },
    components: { Message },
};
</script>


<style scoped>
#burger-table {
    max-width: 1200px;
    margin: 0 auto;
}

#burger-table-heading,
#burger-table-rows,
.burger-table-row {
    display: flex;
    flex-wrap: wrap;
}

#burger-table-heading {
    font-weight: bold;
    padding: 12px;
    border-bottom: 3px solid #333;
}

h2 {
    padding: 50px;
    text-align: center;
}

.burger-table-row {
    width: 100%;
    padding: 12px;
    border-bottom: 1px solid #CCC;
}

#burger-table-heading div,
.burger-table-row div {
    width: 19%;
}

#burger-table-heading .order-id,
.burger-table-row .order-number {
    width: 5%;
}

select {
    padding: 12px 6px;
    margin-right: 12px;
}

.delete-btn {
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

.delete-btn:hover {
    background-color: transparent;
    color: #222;
}
</style>