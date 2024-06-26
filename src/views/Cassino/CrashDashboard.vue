<script setup>
import { ref, onMounted, watch } from "vue";
import { useRoute } from 'vue-router'

// Sections components
import BaseLayout from "../../layouts/sections/components/BaseLayout.vue";

import MediaVelas from "./components/crash/MediaVelas/MediaVelas.vue"
import AlarmePadroes from "./components/crash/AlarmePadroes/AlarmePadroes.vue"
import PadraoEstrategiasMinutagem from "./components/crash/padroes/minutagem/PadraoEstrategiasMinutagem.vue"
import Padroes from "./components/crash/padroes/Padroes.vue"
import ContagemCores from "./components/crash/ContagemCores.vue"
import Caixa from "./components/crash/Caixa.vue"
import GraficoLinha from "./components/crash/GraficoLinha.vue"
import TabelaCrashes from "./components/crash/TabelaCrashes/TabelaCrashes.vue"
import { startCase } from 'lodash';
//nav-pills
import setNavPills from "@/assets/js/nav-pills";
import { Collapse } from 'vue-collapsed'
import Tools from "@/components/commons/index.vue";
import MaterialProgress from "@/components/MaterialProgress.vue";

const estrategias = ref()
const contagem_cores = ref({})
const media_velas = ref({})
const velas = ref([])
const qtd_velas_total = ref(0)
const galho = ref(2)
const targetVela = ref(2)
const balance = ref(0)
const qtd_velas = ref("200")
const route = useRoute()
const platform = route.params.platform
const apiHost = import.meta.env.DEV ? '' : 'https://cassino-online-api-production.up.railway.app'
const loading = ref(false)
const loadedDashboard = ref(false)
const minProbabilidade = ref(50)
const maxProbabilidade = ref(100)
const isExpanded = ref(true)
const padroesSelecionados = ref([])

const urlDashboard = () => {
  return `${apiHost}/api/${platform}/crash/dashboard?
          qtdVelas=${qtd_velas.value}`.replace(/ /g, '')
}

const urlEstrategias = () => {
  return `${apiHost}/api/${platform}/crash/estrategias?
          minProbabilidade=${minProbabilidade.value}&
          maxProbabilidade=${maxProbabilidade.value}&
          qtdVelas=${qtd_velas.value}&
          qtdGalho=${galho.value}&
          targetVela=${targetVela.value}`.replace(/ /g, '')
}

const loadDashboard = () => {
  const url = urlDashboard()
  fetch(url)
    .then(response => response.json())
    .then(data => {
      contagem_cores.value = data['contagem_cores']
      media_velas.value = data['media_velas']
      velas.value = data['velas']
      qtd_velas_total.value = data['qtd_velas_total']
      balance.value = data['balance']
      loadedDashboard.value = true
    })
}

const loadEstrategias = () => {
  estrategias.value = {}

  loading.value = true
  const url = urlEstrategias()
  fetch(url)
    .then(response => response.json())
    .then(data => {
      estrategias.value = data
      loading.value = false
    })
}

const onPadraoClicked = (padrao) => {
  console.log('onPadraoClicked ', padrao)
  padroesSelecionados.value.push(padrao)
}

const evtSource = new EventSource(`https://cassino-database-manager-production.up.railway.app/stream/${platform}/crash`);
evtSource.onmessage = (event) => {
  loadDashboard()
};

//hook
onMounted(() => {
  document.title = `Djabet | ${startCase(platform)} | Crash`
  setNavPills();
  loadDashboard()
});

</script>
<template>
  <BaseLayout :title="`${startCase(platform)} - Crash`">
    <div class="container" v-if="loadedDashboard">
      <div class="row">
        <div class="col-3">
          <Tools />
        </div>
        <div class="col-3">
          <Caixa :balance="balance" />
        </div>
      </div>

      <div class="row" style="justify-content: space-between;">
        <div class="col-sm">
          <MediaVelas :media_velas="media_velas" />
        </div>
        <div class="col-sm">
          <ContagemCores :contagem_cores="contagem_cores" />
        </div>
      </div>
      <div class="row">
        <div class="col">
          <AlarmePadroes :velas="velas" :padroes="padroesSelecionados" :platform="startCase(platform)"/>
        </div>
      </div>
      <div class="row">
        <h4 style="margin-top: 40px; margin-bottom: 40px; text-decoration: underline;" @click="isExpanded = !isExpanded">Estrategias</h4>
      </div>
      <Collapse :when="isExpanded">
        <div class="row">
          <div style="display: flex; flex-direction: column; gap: 20px;">
            <div>
              <span style="margin-right: 12px; font-size: large; width: fit-content;">Velas:</span>
              <input name="qtdVelas" label="Quantidade de Velas" type="number" v-model="qtd_velas" style="width: 60px;" />
            </div>
            <div style="display: flex;">
              <span style="margin-right: 12px; font-size: large; width: fit-content;">Galho:</span>
              <input type="number" v-model="galho" style="width: 60px;" />
            </div>
            <div style="display: flex;">
              <span style="margin-right: 22px; font-size: large; width: fit-content;">Vela:</span>
              <input type="number" v-model="targetVela" style="width: 60px;" />
            </div>
            <div style="display: flex;">
              <span style="margin-right: 10px; font-size: large; width: fit-content;">Min %:</span>
              <input type="number" v-model="minProbabilidade" style="width: 60px;" />
            </div>
            <div style="display: flex;">
              <span style="margin-right: 10px; font-size: large; width: fit-content;">Max %:</span>
              <input type="number" v-model="maxProbabilidade" style="width: 60px;" />
            </div>
            <button @click="loadEstrategias" style="width: fit-content;">Carregar</button>
          </div>
        </div>
        <div class="row">
          <MaterialProgress v-if="loading" color="success" :value="100" />
          <Padroes v-if="estrategias?.padroes" :padroes="estrategias?.padroes" :on-padrao-clicked="onPadraoClicked" />
        </div>
        <div class="row">
          <PadraoEstrategiasMinutagem v-if="estrategias?.minutagem" :minutagem="estrategias?.minutagem" />
        </div>
      </Collapse>
    </div>
    <div>
      <TabelaCrashes :velas="velas" />
    </div>

  </BaseLayout>
</template>
