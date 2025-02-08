<script setup>
import { ref, onMounted } from 'vue';
import BaseTimer from "./components/Timer.vue";
import Player from "./components/Player.vue";

// Initialize deck
const suits = ['clubs', 'diamonds', 'hearts', 'spades'];
const values = Array.from({ length: 13 }, (_, i) => i + 1);
const deck = ref([]);
const currentCard = ref(null);
const resetToggle = ref(false);
const showWinnerModal = ref(false);
const winnerMessage = ref('');

// Add these refs after other ref declarations
const players = ref([
  { cards: [null, null, null], bid: 0 },
  { cards: [null, null, null], bid: 0 },
  { cards: [null, null, null], bid: 0 },
  { cards: [null, null, null], bid: 0 }
]);

// Add these constants for hand rankings
const HAND_RANKINGS = {
  STRAIGHT_FLUSH: 6,
  THREE_OF_KIND: 5,
  STRAIGHT: 4,
  FLUSH: 3,
  PAIR: 2,
  HIGH_CARD: 1
};

// Create full deck
const initializeDeck = () => {
  deck.value = [];
  for (const suit of suits) {
    for (const value of values) {
      deck.value.push({ suit, value });
    }
  }
};

// Draw a random card
const drawCard = () => {
  if (deck.value.length === 0) {
    alert('No cards left in the deck!');
    return;
  }

  const randomIndex = Math.floor(Math.random() * deck.value.length);
  const drawnCard = deck.value.splice(randomIndex, 1)[0];
  currentCard.value = drawnCard;
};

// Add helper functions for hand evaluation
const isStraight = (values) => {
  values.sort((a, b) => a - b);
  // Check for A,2,3
  if (values[0] === 1 && values[1] === 2 && values[2] === 3) return true;
  // Check for Q,K,A
  if (values[0] === 12 && values[1] === 13 && values[2] === 1) return true;
  // Check normal straight
  return values[2] - values[1] === 1 && values[1] - values[0] === 1;
};

const isFlush = (cards) => {
  return cards.every(card => card.suit === cards[0].suit);
};

const getHighestValue = (values) => {
  // Convert 1 (Ace) to 14 for comparison
  return Math.max(...values.map(v => v === 1 ? 14 : v));
};

const getHandRank = (cards) => {
  if (!cards.every(card => card !== null)) return { rank: 0, highCard: 0 };

  const values = cards.map(card => card.value);
  const sortedValues = [...values].sort((a, b) => b - a);

  // Straight Flush
  if (isFlush(cards) && isStraight(values)) {
    return { rank: HAND_RANKINGS.STRAIGHT_FLUSH, highCard: getHighestValue(values) };
  }

  // Three of a Kind
  if (values[0] === values[1] && values[1] === values[2]) {
    return { rank: HAND_RANKINGS.THREE_OF_KIND, highCard: getHighestValue(values) };
  }

  // Straight
  if (isStraight(values)) {
    return { rank: HAND_RANKINGS.STRAIGHT, highCard: getHighestValue(values) };
  }

  // Flush
  if (isFlush(cards)) {
    return { rank: HAND_RANKINGS.FLUSH, highCard: getHighestValue(values) };
  }

  // Pair
  for (let i = 0; i < values.length - 1; i++) {
    if (values[i] === values[i + 1]) {
      return { rank: HAND_RANKINGS.PAIR, highCard: values[i] === 1 ? 14 : values[i] };
    }
  }

  // High Card
  return { rank: HAND_RANKINGS.HIGH_CARD, highCard: getHighestValue(values) };
};

// Update handleTimeUp function
const handleTimeUp = () => {
  // Find player with highest bid
  const highestBidder = players.value.reduce((highest, current, index) => {
    return current.bid > highest.bid ? { ...current, index } : highest;
  }, { bid: -1, index: -1 });

  // Only proceed if someone has placed a bid
  if (highestBidder.bid > 0) {
    // Find the first empty card slot for the highest bidder
    const emptySlotIndex = players.value[highestBidder.index].cards.findIndex(card => card === null);

    if (emptySlotIndex !== -1 && currentCard.value) {
      players.value[highestBidder.index].cards[emptySlotIndex] = currentCard.value;
      // Reset the bid after card is assigned
      players.value[highestBidder.index].bid = 0;
    }
  }

  // Check if all players have full hands
  const allHandsFull = players.value.every(player =>
    player.cards.every(card => card !== null)
  );

  if (allHandsFull) {
    // Calculate results
    const results = players.value.map((player, index) => ({
      playerIndex: index,
      ...getHandRank(player.cards)
    }));

    // Sort by rank, then by high card
    results.sort((a, b) => {
      if (b.rank !== a.rank) return b.rank - a.rank;
      return b.highCard - a.highCard;
    });

    // Announce winner
    const winner = results[0];
    const handTypes = {
      6: 'Straight Flush',
      5: 'Three of a Kind',
      4: 'Straight',
      3: 'Flush',
      2: 'Pair',
      1: 'High Card'
    };

    winnerMessage.value = `Player ${winner.playerIndex + 1} wins with ${handTypes[winner.rank]}!`;
    showWinnerModal.value = true;
    return; // Stop the game
  }

  drawCard();
  resetToggle.value = !resetToggle.value;
};

// Get card image path
const getCardImagePath = (card) => {
  if (!card) return '';
  return new URL(`./assets/${card.suit}_${card.value}.png`, import.meta.url).href;
};

// Add this new function after handleTimeUp
const restartGame = () => {
  // Reset all player cards and bids
  players.value.forEach(player => {
    player.cards = [null, null, null];
    player.bid = 0;
  });

  // Reset the game state
  initializeDeck();
  drawCard();
  showWinnerModal.value = false;
};

onMounted(() => {
  initializeDeck();
  drawCard();
});
</script>

<template>
  <div class="wrapper">
    <BaseTimer @time-up="handleTimeUp" />
    <div class="table-container">
      <div class="table-center">
        <img src="./assets/poker-table.png" alt="poker-table" class="table-image" />
        <div class="deck-container">
          <img src="./assets/poker-deck.png" alt="poker-deck" class="deck-image" />
          <img v-if="currentCard"
             :src="getCardImagePath(currentCard)"
             :alt="currentCard ? `${currentCard.value} of ${currentCard.suit}` : ''"
             class="current-card" />
        </div>
      </div>
      <Player
        class="player top"
        :reset-toggle="resetToggle"
        :cards="players[0].cards"
        @update-bid="(bid) => players[0].bid = bid"
      />
      <Player
        class="player right"
        :reset-toggle="resetToggle"
        :cards="players[1].cards"
        @update-bid="(bid) => players[1].bid = bid"
      />
      <Player
        class="player bottom"
        :reset-toggle="resetToggle"
        :cards="players[2].cards"
        @update-bid="(bid) => players[2].bid = bid"
      />
      <Player
        class="player left"
        :reset-toggle="resetToggle"
        :cards="players[3].cards"
        @update-bid="(bid) => players[3].bid = bid"
      />
      <div class="cards-remaining">剩餘牌數: {{ deck.length }}</div>
    </div>

    <!-- Add this modal markup just before closing wrapper div -->
    <div v-if="showWinnerModal" class="modal-overlay">
      <div class="modal">
        <h2>遊戲結束!</h2>
        <p>{{ winnerMessage }}</p>
        <button @click="restartGame" class="restart-button">重新開始</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 0px;
}

.table-container {
  position: relative;
  max-width: 1280px;
  width: 100%;
  display: flex;
  height: calc(100vh - 244px);
  justify-content: center;
  align-items: center;
}

.table-container .table-image {
  width: 100%;
  height: 60%;
  object-fit: contain;
}

.table-container .deck-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.table-container .deck-image {
  width: 100px;
  height: 100px;
  object-fit: contain;
}

.player {
  position: absolute;
}

.player.top {
  top: 0;
  left: 50%;
  transform: translateX(-50%);
}

.player.right {
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

.player.bottom {
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
}

.player.left {
  left: 0;
  top: 50%;
  transform: translateY(-50%);
}

.table-center {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.current-card {
  width: 80px;
  object-fit: contain;
}

.cards-remaining {
  position: absolute;
  bottom: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
}

.deck-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

/* Add these new styles at the end */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  text-align: center;
  max-width: 400px;
  width: 90%;
}

.restart-button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.restart-button:hover {
  background: #45a049;
}
</style>
