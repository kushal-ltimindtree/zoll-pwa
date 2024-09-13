<template>
  <div class="hero">
    <img src="../assets/rice-dish.png"/>
    <img src="../assets/orange-dish.png"/>
    <img src="../assets/egg-dish.png"/>
    <div class="hero-content wrapper">
      <div>
        <h3>Welcome</h3>
        <h1>Your Culinary Journey Begins Here</h1>
      </div>
      <SearchForm v-on:submitEvent="(searchvalue) => $router.push(`/search?search=${searchvalue}`)" />
      <div class="download-link">
        <template v-if="!isStandalone">
          <h3>To download as an app, Please click on below download Icon</h3>
          <button  @click="pwaInstall"><PhDownload size="25" /></button>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import SearchForm from './SearchForm.vue';
import {onBeforeMount, computed} from "vue"
import { PhDownload } from '@phosphor-icons/vue';

let pwaInstallEvent;

onBeforeMount(() => {
  window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  pwaInstallEvent = e;
  })
})

const isStandalone = computed(() => getPWADisplayMode() === 'standalone');

const pwaInstall = async () => {
  pwaInstallEvent.prompt()
  const { outcome } = await pwaInstallEvent.userChoice;
  if (outcome === 'accepted') {
    // Take extra actions if the user opted for installation.
  } else {
    // Take extra actions if the user cancels.
  }
}

const getPWADisplayMode = () =>{
  const isStandalone = window.matchMedia('(display-mode: standalone)').matches;
  if (document.referrer.startsWith('android-app://')) {
    return 'twa';
  } else if (navigator.standalone || isStandalone) {
    return 'standalone';
  }
  return 'browser';
}
</script>

<style scoped>
.hero {
  width: 100%;
  height: 70vw;
  background-color: white;
  min-height:400px;
  max-height: 600px;
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content:center;
}

.hero img {
  position: absolute;
}

.hero img:nth-child(1) {
  top: -150px;
  left: -200px;
}

.hero img:nth-child(2) {
  width: 320px;
  top: -200px;
  right: -200px;
}

.hero img:nth-child(3) {
  bottom: -127px;
  width: 300px;
  height: 300px;
  right: -100px;
}

.hero-content {
  z-index: 100;
}

.hero-content h1 {
  font-size: 42px;
  margin-bottom: 30px;
}

.hero-content h3 {
  color: rgb(181, 186, 189);
  font-weight: 500;
  margin-bottom: 10px;
}

.hero-content .download-link {
  margin-top: 30px;
}

@media (min-width: 425px) {
  .hero img:nth-child(2) {
    width: 350px;
  }
}

@media (min-width: 768px) {
  .hero img:nth-child(1) {
    top: -243px;
    width: 500px;
    object-fit: contain;
  }

  .hero img:nth-child(2) {
    width: initial;
  }
}
button {
  background-color: #2c3e50;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 100%;
  box-shadow: none;
  border: none;
  margin-right: 10px;
  cursor: pointer;
}
</style>
