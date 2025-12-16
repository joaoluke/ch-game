<template>
  <q-page class="kids-page q-pa-md">
    <div class="full-width row items-center q-mb-md">
      <q-btn flat round icon="arrow_back" @click="goBack" />
      <div class="text-h5 q-ml-sm">Jogo da Memória</div>
      <q-space />
      <div class="text-h6">Acertos: {{ matchedPairs }}/{{ totalPairs }}</div>
    </div>

    <div class="row q-gutter-md justify-center">
      <div
        v-for="(card, index) in cards"
        :key="index"
        class="memory-card"
        :class="{ flipped: card.isFlipped || card.isMatched }"
        @click="flipCard(index)"
      >
        <div class="memory-card-inner">
          <div class="memory-card-front">
            <q-icon name="help" size="lg" />
          </div>
          <div class="memory-card-back">
            <q-icon :name="card.icon" size="lg" />
            <div class="text-caption">{{ card.word }}</div>
          </div>
        </div>
      </div>
    </div>

    <q-dialog v-model="showSuccessDialog" persistent>
      <q-card class="kids-success-card q-pa-md">
        <q-card-section class="column items-center">
          <q-icon name="celebration" size="48px" class="q-mb-sm text-amber-7" />
          <div class="kids-title text-h5 q-mb-xs">Parabéns!</div>
          <div class="kids-subtitle text-subtitle1 text-center">
            Você completou o jogo da memória com {{ moves }} jogadas!
          </div>
        </q-card-section>
        <q-card-actions align="center">
          <q-btn color="primary" unelevated label="Jogar novamente" @click="resetGame" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

interface MemoryGameStats {
  gamesPlayed: number;
  gamesWon: number;
  bestScore: number;
}
import { useRouter } from 'vue-router';
import { wordBuilderWords } from 'src/data/wordBuilderWords';

interface MemoryCard {
  id: string;
  word: string;
  icon: string;
  isFlipped: boolean;
  isMatched: boolean;
}

const router = useRouter();
const cards = ref<MemoryCard[]>([]);
const flippedCards = ref<number[]>([]);
const moves = ref(0);
const gameComplete = ref(false);

const totalPairs = computed(() => Math.min(6, Math.floor(wordBuilderWords.length / 2)));
const matchedPairs = computed(() => cards.value.filter((card) => card.isMatched).length / 2);
const showSuccessDialog = computed({
  get: () => gameComplete.value,
  set: (val) => {
    gameComplete.value = val;
  },
});

function getRandomWords(count: number) {
  const shuffled = [...wordBuilderWords].sort(() => 0.5 - Math.random());
  return shuffled.slice(0, count);
}

function initializeGame() {
  const selectedWords = getRandomWords(totalPairs.value);
  const gameCards = [
    ...selectedWords.map((word) => ({
      id: `${word.id}-1`,
      word: word.word,
      icon: word.icon,
      isFlipped: false,
      isMatched: false,
    })),
    ...selectedWords.map((word) => ({
      id: `${word.id}-2`,
      word: word.word,
      icon: word.icon,
      isFlipped: false,
      isMatched: false,
    })),
  ];

  cards.value = gameCards.sort(() => Math.random() - 0.5);
  flippedCards.value = [];
  moves.value = 0;
  gameComplete.value = false;
}

function flipCard(index: number) {
  const card = cards.value[index];
  if (!card) return;

  if (card.isFlipped || card.isMatched || flippedCards.value.length >= 2) {
    return;
  }

  card.isFlipped = true;

  if (flippedCards.value.length === 0) {
    flippedCards.value = [index];
    return;
  }

  moves.value++;
  const firstCardIndex = flippedCards.value[0];
  const firstCard = firstCardIndex !== undefined ? cards.value[firstCardIndex] : null;

  if (!firstCard) {
    return;
  }

  if (firstCard.word === card.word) {
    firstCard.isMatched = true;
    card.isMatched = true;
    flippedCards.value = [];

    if (matchedPairs.value === totalPairs.value) {
      setTimeout(() => {
        gameComplete.value = true;
        saveGameResult(moves.value);
      }, 500);
    }
  } else {
    flippedCards.value.push(index);
    setTimeout(() => {
      firstCard.isFlipped = false;
      card.isFlipped = false;
      flippedCards.value = [];
    }, 1000);
  }
}

function resetGame() {
  initializeGame();
  showSuccessDialog.value = false;
}

async function goBack() {
  await router.push('/');
}

function saveGameResult(moves: number) {
  try {
    const stats = JSON.parse(localStorage.getItem('memory-game-stats') || '{}');
    const currentStats: MemoryGameStats = {
      gamesPlayed: (stats.gamesPlayed || 0) + 1,
      gamesWon: (stats.gamesWon || 0) + 1,
      bestScore: Math.min(stats.bestScore || Infinity, moves),
    };
    localStorage.setItem('memory-game-stats', JSON.stringify(currentStats));
  } catch (error) {
    console.error('Erro ao salvar estatísticas do jogo:', error);
  }
}

onMounted(() => {
  initializeGame();
});
</script>

<style scoped>
.memory-card {
  width: 100px;
  height: 120px;
  perspective: 1000px;
  cursor: pointer;
  margin: 8px;
}

.memory-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  text-align: center;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.memory-card.flipped .memory-card-inner {
  transform: rotateY(180deg);
}

.memory-card-front,
.memory-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.memory-card-front {
  background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
  color: white;
}

.memory-card-back {
  background: white;
  color: #333;
  transform: rotateY(180deg);
  border: 2px solid #ff9a9e;
}
</style>
