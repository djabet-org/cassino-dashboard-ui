<script setup>
import { ref, onMounted, watch } from "vue";
import { useRoute } from 'vue-router'

// Sections components
import BaseLayout from "../../layouts/sections/components/BaseLayout.vue";
import View from "../../layouts/sections/components/View.vue";

import MediaVelas from "./components/MediaVelas.vue"
import PadraoEstrategiasMinutagem from "./components/PadraoEstrategiasMinutagem.vue"
import PadraoEstrategiasAposPadrao from "./components/PadraoEstrategiasAposPadrao.vue"
import PadraoEstrategiasSomaDigitos from "./components/PadraoEstrategiasSomaDigitos.vue"
import ContagemCores from "./components/ContagemCores.vue"
import GraficoLinha from "./components/GraficoLinha.vue"
import Clock from "./components/Clock.vue"
import TabelaCrashes from "./components/TabelaCrashes/TabelaCrashes.vue"
import { startCase } from 'lodash';
//nav-pills
import setNavPills from "@/assets/js/nav-pills";
import PadraoEstrategiasAposXx from "./components/PadraoEstrategiasAposXx.vue";

const estrategias = ref({})
const contagem_cores = ref({})
const media_intervalos = ref({})
const velas = ref([])
const qtd_velas_total = ref(0)
const qtd_velas = ref("200")
const route = useRoute()
const platform = route.params.platform
const audio = ref()
const apiHost = import.meta.env.DEV ? '' : 'https://cassino-online-api-production.up.railway.app'

const load = () => {
  fetch(`${apiHost}/api/${platform}/crash/dashboard/${qtd_velas.value}`)
    .then(response => response.json())
    .then(data => {
      estrategias.value = data['estrategias']
      contagem_cores.value = data['contagem_cores']
      media_intervalos.value = data['media_intervalos']
      velas.value = data['velas']
      qtd_velas_total.value = data['qtd_velas_total']
    })
}

const evtSource = new EventSource("https://cassino-database-manager-production.up.railway.app/ingested");
evtSource.onmessage = (event) => {
  console.log(`message: ${event.data}`);
  load()
};

//hook
onMounted(() => {
  setNavPills();
  load()
});

const alertIfVelasAcima50 = (value) => {
  const percentageVerde = value?.percentageVerde
  const isPercentageHigherThan = parseInt(percentageVerde.slice(0, 2)) >= 52
  console.log(parseInt(percentageVerde.slice(0, 2)))

  if (isPercentageHigherThan) {
    audio.value.play();
  }
}

watch(() => contagem_cores.value, (contagemCores, prevContagemCores) => {
  alertIfVelasAcima50(contagemCores)
})

</script>
<template>
  <BaseLayout :title="`${startCase(platform)} - Crash`">
    <div class="container">
      <div class="row">
        <div class="col-sm-4">
          <div style="justify-content: space-between;">
            <label for="qtdVelas">
              Quantidade de velas:
              <input name="qtdVelas" class="input-group-static" label="Quantidade de Velas" type="number"
                v-model="qtd_velas" />
            </label>
            <button @click="load">Load</button>
            <audio ref="audio" controls>
              <source src="https://www.myinstants.com/media/sounds/111-pokemon-recovery.mp3" type="audio/mpeg">
              Your browser does not support the audio element.
            </audio>
          </div>
        </div>
        <div class="col-sm-8">
          <Clock />
        </div>
      </div>

      <div class="row" style="justify-content: space-between;">
        <div class="col-sm">
          <MediaVelas :media_intervalos="media_intervalos" />
        </div>
        <div class="col-sm">
          <ContagemCores :contagem_cores="contagem_cores" />
        </div>        
      </div>

     
      <div class="row" style="margin: 4% 2%;">
          <h4>Estrategias - Probabilidades</h4>
      </div>
      <div class="row" style="margin: 2%;">
        <h5 class="mb-1">* Duplo</h5>
      </div>
      <div class="row">
        <PadraoEstrategiasAposPadrao :estrategias="estrategias" />
      </div>
      <div class="row" style="margin: 2%;">
        <h5 class="mb-1">* Minutagem</h5>
      </div>
      <div class="row">
        <PadraoEstrategiasMinutagem :estrategias="estrategias" />
      </div>
      <div class="row">        
        <div class="col">
          <PadraoEstrategiasSomaDigitos :estrategias="estrategias" />
        </div>
        <div class="col">
          <PadraoEstrategiasAposXx :estrategias="estrategias" />
        </div>
        <div class="col"></div>
      </div>
    </div>
    <div>
      <TabelaCrashes :velas="velas" />
    </div>

  </BaseLayout>
</template>