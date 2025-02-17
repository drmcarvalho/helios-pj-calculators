<template>
  <div class="container">
    <h1 class="text-left pb-2">
      Calculadora de IRPF para MEI
    </h1>
    <div class="grid">
      <div class="card">
        <h2 class="text-left pb-2">Passo 1:</h2>
        <div>
          <div class="form__control">
            <label class="text-left" for="receitaBrutaAnual"
              >Receita Bruta Anual (salário mensal * 12)</label
            >
            <input
              type="tel"
              name="receitaBrutaAnual"
              v-model.lazy="receitaBrutaAnual"
              v-money="money"
            />
          </div>

          <div class="form__control">
            <label class="text-left" for="despesasComprovadasDoMei"
              >Despesas comprovadas do MEI (gastos no ano)</label
            >
            <input
              type="tel"
              name="despesasComprovadasDoMei"
              v-model.lazy="despesasComprovadasDoMei"
              v-money="money"
            />
          </div>

          <div class="form__control">
            <label class="text-left" for="lucroEvidenciado"
              >Lucro Evidenciado (Receita Bruta Anual - Despesas comprovadas do
              MEI)</label
            >
            <input
              type="tel"
              name="lucroEvidenciado"
              v-money="money"
              :value="Math.ceil(lucroEvidenciado)"
              disabled
            />
          </div>

          <div class="form__control">
            <label class="text-left" for="parcelaIsenta"
              >Parcela Isenta de tributação (32% da Receita Bruta Anual)</label
            >
            <input
              type="tel"
              name="parcelaIsenta"
              v-money="money"
              :value="Math.ceil(parcelaIsenta)"
              disabled
            />
          </div>

          <div class="form__control">
            <label class="text-left" for="parcelaIsenta"
              >Lucro tributavel (Lucro evidenciado - Parcela Isenta)</label
            >
            <input
              type="tel"
              name="lucroTributavel"
              v-money="money"
              :value="Math.ceil(lucroTributavel)"
              disabled
            />
          </div>
        </div>
      </div>
      <div class="card">
        <div>
          <h2 class="text-left pb-2">Passo 2:</h2>
          <h3 class="text-left">
            Olhar tabela anual da Receita Federal (2021)
          </h3>

          <table class="pb-2">
            <tr class="border-bottom">
              <th>Base de cálculo</th>
              <th>Alíquota</th>
              <th>Parcela a deduzir do IR</th>
            </tr>
            <tr
              v-for="aliquota in aliquotas"
              :key="aliquota.id"
              :id="aliquota.id"
              :class="{ 'row--selected': aliquota.id === valorDaAliquota.id }"
            >
              <td class="text-left">{{ aliquota.text }}</td>
              <td>{{ aliquota.percentage }}%</td>
              <td>{{ convert2Real(aliquota.deduzir) }}</td>
            </tr>
          </table>

          <h2 class="text-left pb-2">Passo 3:</h2>
          <div class="form__control">
            <label class="text-left" for="parcelaIsenta"
              >Parcela a deduzir (Lucro tributavel * porcentagem da aliquota da
              tabela)</label
            >
            <input
              type="tel"
              name="parcelaDeduzida"
              v-money="money"
              :value="parcelaDeduzida"
              disabled
            />
          </div>

          <h2 class="text-left pb-2">Passo 4:</h2>
          <div class="form__control">
            <label class="text-left" for="parcelaIsenta"
              >Total para pagar de IR (Parcela deduzida - Parcela deduzida da
              tabela)</label
            >
            <input
              type="tel"
              name="totalImpostoDeRenda"
              v-money="money"
              :value="totalImpostoDeRenda * 100"
              disabled
            />
          </div>

          <div class="featured text-left">
            Você pode guardar por mês o valor de
            <b>{{ convert2Real(totalImpostoDeRenda / 12) }}</b> para pagar o
            imposto de renda de pessoa física no outro ano
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { convertMoneyMaskToNumber } from "@/utils/utils.js";
import { VMoney } from "v-money";
export default {
  data() {
    return {
      receitaBrutaAnual: 0,
      despesasComprovadasDoMei: 0,
      money: {
        decimal: ",",
        thousands: ".",
        prefix: "R$ ",
        precision: 2,
      },
      aliquotas: [],
    };
  },
  mounted() {
    this.getTabelaAliquotas();
  },
  directives: { money: VMoney },
  methods: {
    track() {
      this.$ga.page("/calculator");
    },
    async getTabelaAliquotas() {
      await axios.get("/tabela-aliquota-2021.json").then((response) => {
        this.aliquotas = response.data.aliquotas;
      });
    },
    convert2Real(valor) {
      return valor.toLocaleString("pt-br", {
        style: "currency",
        currency: "BRL",
      });
    },
  },
  computed: {
    lucroEvidenciado() {
      const receitaBruta = convertMoneyMaskToNumber(this.receitaBrutaAnual);
      const despesasCompravadas = convertMoneyMaskToNumber(
        this.despesasComprovadasDoMei
      );
      // To v-money work properly
      return (receitaBruta - despesasCompravadas) * 100;
    },
    parcelaIsenta() {
      const porcentagem = 32 / 100;
      const receitaBruta = convertMoneyMaskToNumber(this.receitaBrutaAnual);
      return porcentagem * receitaBruta * 100;
    },
    lucroTributavel() {
      const lucroEvidenciado = this.lucroEvidenciado;
      const parcelaIsenta = this.parcelaIsenta;
      return lucroEvidenciado - parcelaIsenta;
    },
    valorDaAliquota() {
      let faixaAliquota = {};

      const lucroTributavel = this.lucroTributavel / 100;

      for (const iterator of this.aliquotas) {
        if (
          lucroTributavel >= iterator.start &&
          lucroTributavel <= iterator.end
        ) {
          faixaAliquota = iterator;
        }
      }

      return faixaAliquota;
    },
    parcelaDeduzida() {
      const lucroTributavel = this.lucroTributavel / 100;
      const porcentagemAliquota = this.valorDaAliquota.percentage / 100;

      return Math.ceil(lucroTributavel * porcentagemAliquota * 100);
    },
    totalImpostoDeRenda() {
      const parcelaDeduzida = this.parcelaDeduzida / 100;
      const parcelaDeduzidaPorAliquota = this.valorDaAliquota.deduzir;
      const IR = Math.ceil(parcelaDeduzida - parcelaDeduzidaPorAliquota);

      return IR;
    },
  },
};
</script>
