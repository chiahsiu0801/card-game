<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  resetToggle: {
    type: Boolean,
    required: true
  },
  cards: {
    type: Array,
    default: () => []
  }
});

const cards = ref(props.cards || []);
const inputDisabled = ref(false);
const bidInput = ref(0);
const emit = defineEmits(['updateBid']);

const handleBidConfirm = () => {
  inputDisabled.value = true;
  emit('updateBid', Number(bidInput.value));
};

// Method to reset the bid state
const resetBidState = () => {
  inputDisabled.value = false;
  bidInput.value = 0;
};

watch(() => props.resetToggle, () => {
  resetBidState();
});

// Add a new watcher for cards prop
watch(() => props.cards, (newCards) => {
  // Update the cards ref with new cards
  cards.value = newCards;
  // Reset bid state when cards are reset (all null)
  if (newCards.every(card => card === null)) {
    resetBidState();
  }
}, { deep: true });

// Add the getCardImagePath function
const getCardImagePath = (card) => {
  if (!card) return '';
  const baseUrl = import.meta.env.BASE_URL || '/card-game/';
  return new URL(`${baseUrl}assets/${card.suit}_${card.value}.png`, import.meta.url).href;
};
</script>

<template>
  <div class="player-container">
    <div class="player-info">
      <img src="../assets/player.png" alt="player" class="player-image" />
      <div class="card-container">
        <div v-for="(card, index) in cards" :key="index" class="card-placeholder">
          <img v-if="card"
               :src="getCardImagePath(card)"
               :alt="`${card.value} of ${card.suit}`"
               class="player-card" />
        </div>
      </div>
    </div>
    <div class="bid-container">
        <span class="bid-label">競標價格 : </span>
        <div class="bid-input-container">
            <input
              type="number"
              v-model="bidInput"
              :disabled="inputDisabled"
            />
            <span>USDT</span>
        </div>
        <button
          @click="handleBidConfirm"
          :disabled="inputDisabled"
        >確認價格</button>
    </div>
  </div>
</template>

<style scoped>
.player-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.player-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.player-image {
  width: 80px;
}

.card-container {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.card-placeholder {
  width: 60px;
  height: 84px;
  border: 2px dashed #ccc;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.player-card {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.bid-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.bid-input-container {
  display: flex;
  align-items: center;
  gap: 4px;
}

.bid-label {
  font-size: 16px;
  font-weight: bold;
}

input[type="number"] {
  width: 80px;
  padding: 5px;
  border: 2px solid #666;
  border-radius: 4px;
  font-size: 16px;
  text-align: center;
  outline: none;
}

input[type="number"]:focus {
  border-color: #4a90e2;
  box-shadow: 0 0 5px rgba(74, 144, 226, 0.5);
}

/* Remove spinner buttons from number input */
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}

input[type="number"]:disabled {
  cursor: not-allowed;
  pointer-events: none;
  opacity: 0.7;
}

button {
  padding: 6px 12px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

button:hover {
  background-color: #357abd;
}

button:active {
  background-color: #2a5f96;
}

button:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.4);
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
</style>
