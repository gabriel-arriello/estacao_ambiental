<template>
  <div class="sensor-detail">
    <Card>
      <template #title>{{ nomeSensor }}</template>
      <template #content>
        <div class="flex align-items-center gap-4 mb-4">
          <span class="text-6xl font-bold">{{ valorAtual }}</span>
          <span class="text-2xl">{{ unidade }}</span>
        </div>
        <Chart
          ref="chart"
          type="line"
          :data="chartData"
          :options="chartOptions"
          class="h-20rem"
        />
      </template>
    </Card>
    <Button
      label="Voltar"
      icon="pi pi-arrow-left"
      @click="$router.go(-1)"
      class="mb-4"
    />
  </div>
</template>

<script>
import axios from "axios";
import Card from "primevue/card";
import Chart from "primevue/chart";
import Button from "primevue/button";

export default {
  components: { Card, Chart, Button },
  data() {
    console.log("id: " + this.$route.params.id);
    return {
      sensorId: this.$route.params.id,
      historico: [],
      pollingInterval: null,
    };
  },
  computed: {
    nomeSensor() {
      const nomes = {
        pm1: "Material Particulado 1.0 (PM1.0)",
        pm10: "Material Particulado 10.0 (PM10.0)",
        pm25: "Material Particulado 2.5 (PM2.5)",
        temperatura: "Temperatura",
        umidade: "Umidade",
        uv: "Luz Ultravioleta (UV)",
        ruido: "Ruído Sonoro",
        co: "Monóxido de Carbono (CO)",
        no2: "Dióxido de Nitrogênio (NO2)",
        voc: "Compostos Orgânicos Voláteis (VOCs) ",
        o3: "Trioxigênio (O3)",
        co2: "Dióxido de Carbono (CO2)",
      };
      return nomes[this.sensorId] || this.sensorId;
    },
    unidade() {
      const unidades = {
        pm1: " pm1.0/10^-4 m³",
        pm10: " pm10.0/10^-4 m³",
        pm25: " pm2.5/10^-4 m³",
        temperatura: " °C",
        umidade: " %",
        uv: " UV index",
        ruido: " dB",
        co: " ppm",
        no2: " ppm",
        voc: " ppb",
        o3: " ppb",
        co2: " ppm",
      };
      return unidades[this.sensorId] || "";
    },
    valorAtual() {
      return this.historico[-1]?.[this.sensorId] || "--";
    },
    chartData() {
      return {
        labels: this.historico.map((d) =>
          new Date(d.timestamp).toLocaleTimeString()
        ),
        datasets: [
          {
            label: this.nomeSensor,
            data: this.historico.map((d) => d[this.sensorId]),
            borderColor: "#4BC0C0",
            tension: 0.4,
            fill: false,
          },
        ],
      };
    },
    chartOptions() {
      return {
        plugins: { legend: { display: false } },
        scales: {
          y: { title: { display: true, text: this.unidade } },
          x: { title: { display: true, text: "Tempo (hh-mm-ss)" } },
        },
        animation: false, // Remove animação
      };
    },
  },
  methods: {
    async fetchData() {
      try {
        const response = await axios.get("http://localhost:5000/historico");
        const novosDados = response.data;

        // Adiciona os novos dados ao histórico (preservando os dados anteriores)
        this.historico = novosDados;

        this.$nextTick(() => {
          // Aqui, você deve acessar o gráfico e adicionar os novos dados de maneira incremental
          this.$refs.chart.chart.update(); // Atualiza o gráfico sem animação
        });
      } catch (error) {
        console.error("Erro ao obter histórico:", error);
      }
    },
  },
  mounted() {
    this.fetchData();
    this.pollingInterval = setInterval(this.fetchData, 5000); // Atualiza a cada 5s
  },
  beforeUnmount() {
    clearInterval(this.pollingInterval);
  },
};
</script>

<style scoped>
.sensor-detail {
  padding: 1rem;
  max-width: 1200px;
  margin: 0 auto;
  min-height: 100vh;
}
</style>
