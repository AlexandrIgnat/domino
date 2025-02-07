<script setup>
import { useTemplateRef, reactive, ref } from 'vue'
import { cards } from './cards/cards'

const suggLeft = useTemplateRef('suggLeft')
const suggRight = useTemplateRef('suggRight')

console.log(suggLeft, suggRight)

const state = reactive({
  sourceDeck: [],
  userCards: [],
  droppedCards: [],
  pickedCards: {},
})

let left = ref(0)
let right = ref(0)

// Инициализация игры
function initGame() {
  const shuffled = shuffleArray([...cards])
  state.sourceDeck = shuffled
  state.userCards = state.sourceDeck.splice(0, 7)
  state.droppedCards = []
}

// Перемешивание карт
function shuffleArray(array) {
  return array.sort(() => Math.random() - 0.5)
}

function move(element) {
  state.userCards.forEach((item) => {
    item.class = null
  })

  element.class = 'active'

  if (Object.keys(state.droppedCards).length === 0) {
    element.class = null
    if (element.identical) {
      element.class = 'user-card_rotated'
    }
    state.droppedCards.push(element)
    left.value = element.left
    right.value = element.right
    state.userCards = state.userCards.filter((item) => item.id !== element.id)
    state.pickedCards = {}

    return
  } else if (Object.keys(state.pickedCards).length === 0) {
    state.pickedCards = element

    if (element.values.includes(left.value) && element.values.includes(right.value)) {
      suggLeft.value.classList.add('show')
      suggRight.value.classList.add('show')
    } else {
      suggLeft.value.classList.remove('show')
      suggRight.value.classList.remove('show')
    }

    return
  } else if (Object.keys(state.pickedCards).length !== 0) {
    if (!isEqualShallow(state.pickedCards, element)) {
      state.pickedCards = element
      suggLeft.value.classList.remove('show')
      suggRight.value.classList.remove('show')
      return
    }

    let match = 0
    let leftOld = left.value
    let rightOld = right.value

    if (left.value == element.left) {
      left.value = state.pickedCards.right
      match = 1
    } else if (left.value == element.right) {
      left.value = state.pickedCards.left
      match = 1
    } else if (right.value == element.left) {
      right.value = state.pickedCards.right
      match = 1
    } else if (right.value == element.right) {
      right.value = state.pickedCards.left
      match = 1
    }

    if (match == 1) {
      element.class = null
      if (state.pickedCards.left == leftOld) {
        console.log(1)
        element.class = 'rotated'
        state.droppedCards.unshift(element)
      } else if (state.pickedCards.left == rightOld) {
        console.log(2)
        state.droppedCards.push(element)
      } else if (state.pickedCards.right == leftOld) {
        console.log(3)
        state.droppedCards.unshift(element)
      } else if (state.pickedCards.right == rightOld) {
        console.log(4)
        element.class = 'rotated'
        state.droppedCards.push(element)
      }

      state.userCards = state.userCards.filter((item) => item.id !== element.id)
      state.pickedCards = {}
      suggLeft.value.classList.remove('show')
      suggRight.value.classList.remove('show')
      return
    }
  }
  state.pickedCards = element
}

function add(element) {
  state.sourceDeck = state.sourceDeck.filter((item) => item.id !== element.id)
  state.userCards.push(element)
}

function isEqualShallow(obj1, obj2) {
  const keys1 = Object.keys(obj1)
  const keys2 = Object.keys(obj2)

  if (keys1.length !== keys2.length) return false

  for (let key of keys1) {
    if (obj1[key] !== obj2[key]) return false
  }

  return true
}

initGame()
</script>

<template>
  <div class="game-area">
    <!-- Основная колода -->
    <div class="game-area__cards" data-id="source-deck">
      <div v-for="element in state.sourceDeck" class="card" :key="element.id" @click="add(element)">
        <span>{{ element.icon }}</span>
        <span class="line"></span>
        <span>{{ element.icon }}</span>
      </div>
    </div>

    <div class="drop-zone">
      <div ref="suggLeft" class="user-card__suggest-left hiden"></div>
      <div
        v-for="element in state.droppedCards"
        class="user-card"
        :key="element.id"
        :class="['user-card', element.class ? element.class : '', element.img ? element.img : '']"
      >
        <span>{{ element.left }}</span>
        <span class="line"></span>
        <span>{{ element.right }}</span>
      </div>
      <div ref="suggRight" class="user-card__suggest-right hiden"></div>
    </div>

    <!-- Карты игрока -->
    <div class="game-area__user-cards">
      <div
        v-for="element in state.userCards"
        @click="move(element)"
        :class="['user-card', element.class ? element.class : '', element.img ? element.img : '']"
        :key="element.id"
      >
        <span>{{ element.left }}</span>
        <span class="line"></span>
        <span>{{ element.right }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-area {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
}

.game-area__cards,
.game-area__user-cards {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  min-height: 100px;
  padding: 15px;
  border: 2px dashed #ccc;
}

.drop-zone {
  min-height: 200px;
  border: 3px dashed #666;
  transition: background 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card,
.user-card {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: center;
  justify-content: center;
  min-width: 120px;
  padding: 15px;
  border: 2px solid #333;
  border-radius: 8px;
  background: white;
  z-index: 2;
  cursor: move;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
}

.user-card_rotated {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  grid-template-rows: repeat(3, 1fr);
  align-items: center;
  justify-content: center;
  min-height: 100px;
  min-width: auto;
  padding: 15px;
  border: 2px solid #333;
  border-radius: 8px;
  background: white;
  z-index: 2;
  cursor: move;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
}

.user-card_rotated span {
  transform: rotate(90deg);
}

.line {
  display: inline-block;
  width: 1px;
  height: 20px;
  background: #000;
  margin: 0 10px;
}

.active {
  transform: translateY(-5px);
}

.icons8-domino-32png {
  background-image: url('./assets/cards/icons8-domino-32.png');
  background-size: 100% 100%;
}

.rotated {
  transform: rotate(180deg);
}

.user-card__suggest-left,
.user-card__suggest-right {
  width: 35px;
  height: 51px;
  border: 1px solid;
  border-radius: 5px;
}

.user-card__suggest-left {
  transform: translateX(10%);
  background-color: #d321218c;
}

.user-card__suggest-right {
  transform: translateX(-10%);
  background-color: #3321d38c;
}

.hiden {
  visibility: hidden;
}

.show {
  visibility: visible;
}

/* .grid-icon-1 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(1, 1fr);
  grid-template-rows: repeat(1, 1fr);
}
.grid-icon-2 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(1, 1fr);
  grid-template-rows: repeat(2, 1fr);
}
.grid-icon-3 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(1, 1fr);
}
.grid-icon-4 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
}
.grid-icon-5 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
}
.grid-icon-6 {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
} */
</style>
