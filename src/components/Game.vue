<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { getRandomElement, getRandomNum, setFocusOnInput } from '../utils/utils';
import ChampList from '../components/ChampList.vue';
import { Champions } from '../typescript/ChampionsType';

const props = defineProps<{ champions: Champions[] }>();

const currentChampion = ref<Champions>({ id: '', name: '', skins: [], value: undefined });
const filteredChampions = ref<Champions[]>([]);
const currentSkin  = ref('');
const userInput = ref('');
const gameInProgress = ref(true);
const availableChampions = ref<Champions[]>(props.champions);
const guessedChampions = ref<Champions[]>([]);
const inputRef = ref<HTMLInputElement | null>(null);
const splashartXPos = ref(getRandomNum(-190, 18));
const splashartYPos = ref(getRandomNum(-16, 16));
const splashartScale = ref(6);
const loading = ref(true)
const surrended = ref(false)

function filterAvailableChampions() {
  filteredChampions.value = availableChampions.value.filter(champ => 
    champ.name.toLowerCase().includes(userInput.value.toLowerCase())
  );
}

function resetGame() {
  loading.value = true
  splashartXPos.value = getRandomNum(-190, 18);
  splashartYPos.value = getRandomNum(-16, 16);
  splashartScale.value = 6;
  surrended.value = false;
  const randomChampion = getRandomElement(props.champions);
  if (randomChampion) {
    currentChampion.value = randomChampion;
    const randomSkin = getRandomElement(randomChampion.skins);
    if (randomSkin) {
      currentSkin .value = randomSkin;
    }
  }
  
  availableChampions.value = props.champions;
  guessedChampions.value = [];
  gameInProgress.value = true;
  setFocusOnInput(inputRef.value);
  setTimeout(() => loading.value = false, 1000)
}

function addChampionToGuessedList(name: string) {
  const champ = availableChampions.value.find(champ => champ.name.toLowerCase() === name.toLowerCase());
  if (champ) guessedChampions.value.unshift(champ);
}

function removeChampionFromAvailableList(name: string) {
  availableChampions.value = availableChampions.value.filter(champ => champ.name.toLowerCase() !== name.toLowerCase());
}

function verifyChampionGuess(selectedChampionName: string) {
  if (selectedChampionName.toLowerCase() === currentChampion.value.name.toLowerCase()) {
    addChampionToGuessedList(selectedChampionName);
    gameInProgress.value = false;
    splashartScale.value = 1;
    splashartXPos.value = 0;
    splashartYPos.value = 0;
  } else {
    addChampionToGuessedList(selectedChampionName);
    removeChampionFromAvailableList(selectedChampionName);
    splashartScale.value = splashartScale.value > 1.2 ? splashartScale.value - splashartScale.value * 0.1 : splashartScale.value;
    setFocusOnInput(inputRef.value);
  }
  userInput.value = '';
}

function checkFirstFilteredChampion() {
  if (userInput.value.length > 0) {
    if (filteredChampions.value.length > 0) {
      const firstChampionName = filteredChampions.value[0].name;
      verifyChampionGuess(firstChampionName);
    }
  }
}


function surrender() {
  surrended.value = true;
  gameInProgress.value = false;
  splashartScale.value = 1;
  splashartXPos.value = 0;
  splashartYPos.value = 0;
  addChampionToGuessedList(currentChampion.value.name);
}

onMounted(() => {
  resetGame();
  setFocusOnInput(inputRef.value);
});
</script>

<template>
  <div class="game">
    <h2>Guess who?</h2>
    <div class="reload">
      <button v-if="gameInProgress" @click="surrender">Surrender</button>
      <img @click="resetGame" src="../assets/reload.svg" alt="reload" width="32">
    </div>
    <div class="zoomed-img">
    <h2 v-if="loading">Chargement...</h2>
      <img v-else
        :src="'https://ddragon.leagueoflegends.com/cdn/img/champion/splash/'+ currentChampion.id + '_' + currentSkin  +'.jpg'"
        :alt="currentSkin +'_id'"
        :class="gameInProgress ? 'splashart': 'revealed'"
        :style="{
          'object-position': splashartXPos + 'px ' + splashartYPos + 'px',
          'transform': 'scale(' + splashartScale + ')'
        }"
      />
    </div>
    <div class="sub" v-if="gameInProgress">
      <input v-model="userInput" ref="inputRef" @input="filterAvailableChampions" placeholder="Search for a champion" @keydown.enter="checkFirstFilteredChampion" />
      <button @click="checkFirstFilteredChampion">Confirm</button>
    </div>
    <div v-else-if="!gameInProgress && !surrended" class="sub">
      <h3>GG WP!</h3>
      <button @click="resetGame">Play again</button>
      <h3>GG WP!</h3>
    </div>
    <div v-else class="sub">
      <h3>FF 15</h3>
      <button @click="resetGame">Try again</button>
      <h3>FF 15</h3>
    </div>

    <div class="selectable" v-if="userInput.length > 0">
      <ChampList :champions="filteredChampions" :isProposed="false" @selectChampion="verifyChampionGuess"/>
    </div>
  </div>
  <div class="proposed">
    <ChampList :champions="guessedChampions" :isProposed="true" :champion="currentChampion"/>
  </div>
</template>

<style scoped>
.game {
  display: flex;
  flex-direction: column;
  gap: 16px;
  position: relative;
}

.zoomed-img {
  margin: 0 auto;
  min-width: 252px;
  min-height: 252px;
  max-width: 252px;
  max-height: 252px;
  width: fit-content;
  height: fit-content;
  overflow: hidden;
  border-radius: 8px;
  border: #1a1a1a 5px solid;
  display: flex;
  align-items: center;
  justify-content: center;
}

.zoomed-img:has(.revealed) {
  border: 0;
  border-radius: 0;
  max-width: 415px;
  overflow:visible;
}

.splashart {
  width: 252px;
  height: 252px;
  object-fit: cover;
}

.revealed{
  width: 98%;
  height: 100%;
  border-radius: 8px;
  border: #1a1a1a 5px solid;
}

.sub {
  display: flex;
  justify-content: space-between;
}

.reload {
  display: flex;
  justify-content: end;
  gap: 16px;
}

.selectable {
  position: absolute;
  width: 100%;
  top: 102%;
  left: 0;
  z-index: 100;
}

.reload img {
  border-radius: 150%;
  border: 1px solid transparent;
  padding: 8px;
  background-color: #1a1a1a;
  cursor: pointer;
  transition: border-color 0.25s;
  }

.reload img:hover {
  border-color: #646cff;
}

input {
  height: 45px;
  width: 280px;
  font-size: 1.5rem;
  padding:0 8px;
}
</style>
