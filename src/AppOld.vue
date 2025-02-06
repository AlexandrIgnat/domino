<script setup>
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
import { ref, reactive, computed } from 'vue';
import draggable from 'vuedraggable';
import { cards } from './cards/cards'

// Инициализация реактивного состояния
const state = reactive({
  user_cards: [],
  deck: [...cards]
})

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
}

// Инициализация игры
function initGame() {
  shuffleArray(state.deck);
  state.user_cards.push(...state.deck.splice(0, 7));
}



// Обработчик выбора карты
function pickCard(card) {
  const index = state.deck.findIndex(c => c.value === card.value);
  if (index > -1) {
    state.deck.splice(index, 1);
    state.user_cards.push(card);
  }
}


initGame();
</script>

<template>
  <!-- <header>
    <div class="wrapper">
      <HelloWorld msg="You did it!"/>

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
        <RouterLink to="/contacts">Contacts</RouterLink>
      </nav>
    </div>
  </header> -->

  <div class="game-area">
    <div class="game-area__cards">
      <div v-for="(card, index) in state.deck" @click="pickCard(card)" class="card">
        <span>*</span>
        <span class="line"></span>
        <span>*</span>
      </div>
    </div>

    <div class="game-area__user-cards">
      <div v-for="card in state.user_cards" class="user_card">
        <span>{{ card.left }}</span>
        <span class="line"></span>
        <span>{{ card.right }}</span>
      </div>
    </div>
  </div>


  <!-- <RouterView /> -->
</template>

<style scoped>
.game-area {
  display: flex;
  gap: 50px;
  justify-content: center;
  flex-direction: column;
  flex-wrap: wrap;
}

.game-area__cards {
  display: grid;
  grid-template-columns: auto auto auto;  
}

.game-area__user-cards {
  display: grid;
  grid-template-columns:repeat(7, 1fr);
  grid-template-rows: 1fr 1fr 1fr;
}

.card, .user_card {
  display: grid;
  width: 100px;
  grid-template-columns: 1fr auto 1fr;
  grid-template-rows: 1fr;
  margin: 5px;
  padding: 5px;
  border: 1px solid black;
  border-radius: 7px;
  justify-items: center;
}

.user_card {
  display: grid;
  width: initial;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr auto 1fr;
  margin: 5px;
  padding: 5px;
  border: 1px solid black;
  border-radius: 7px;
  justify-items: center;
}

.line {
  width: 1px;
  height: 100%;
  background-color: black;
}

.user_card .line {
  width: 100%;
  height: 1px;
  background-color: black;
}
</style>
