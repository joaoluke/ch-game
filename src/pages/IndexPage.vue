<template>
  <q-page class="kids-page q-pa-md column items-center">
    <div class="kids-title text-h4 q-mb-lg">Jogos de Palavras</div>

    <div class="row q-gutter-md justify-center full-width">
      <q-card v-for="game in games" :key="game.id" class="kids-card q-pa-md" style="width: 280px">
        <q-card-section class="text-center">
          <q-icon :name="game.icon" size="xl" class="q-mb-sm" />
          <div class="text-h6">{{ game.title }}</div>
          <div class="kids-subtitle text-subtitle2 q-mt-xs">
            {{ game.description }}
          </div>

          <div v-if="game.stats && gameStats[game.id]" class="q-mt-sm text-caption">
            <div class="row items-center">
              <q-icon :name="gameStats[game.id].icon" size="sm" class="q-mr-xs" />
              {{ gameStats[game.id].value }} {{ gameStats[game.id].description }}
            </div>
          </div>
        </q-card-section>

        <q-card-actions align="center">
          <q-btn color="primary" unelevated label="Jogar" @click="goToGame(game.routeName)" />
        </q-card-actions>
      </q-card>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';

interface GameDefinition {
  id: 'game-word-builder' | 'game-memory' | 'game-hangman';
  title: string;
  description: string;
  icon: string;
  routeName: string;
  stats: boolean;
}

interface GameStat {
  title: string;
  value: number;
  icon: string;
  description: string;
}

interface WordBuilderStats {
  correctById: Record<string, number>;
}

interface MemoryGameStats {
  gamesPlayed: number;
  gamesWon: number;
  bestScore: number;
}

interface HangmanStats {
  gamesPlayed: number;
  gamesWon: number;
  currentStreak: number;
  maxStreak: number;
}

const games = ref<GameDefinition[]>([
  {
    id: 'game-word-builder',
    title: 'Montar a palavra',
    description: 'Olhe a figura e monte a palavra letra por letra.',
    icon: 'spellcheck',
    routeName: 'game-word-builder',
    stats: true,
  },
  {
    id: 'game-memory',
    title: 'Jogo da Memória',
    description: 'Encontre os pares de palavras e figuras iguais.',
    icon: 'memory',
    routeName: 'game-memory',
    stats: true,
  },
  {
    id: 'game-hangman',
    title: 'Jogo da Forca',
    description: 'Descubra a palavra antes que o boneco seja enforcado!',
    icon: 'sentiment_very_satisfied',
    routeName: 'game-hangman',
    stats: true,
  },
]);

const wordBuilderStats = ref<WordBuilderStats | null>(null);
const memoryGameStats = ref<MemoryGameStats | null>(null);
const hangmanStats = ref<HangmanStats | null>(null);

function loadWordBuilderStats(): WordBuilderStats {
  if (typeof window === 'undefined') {
    return { correctById: {} };
  }

  const raw = window.localStorage.getItem('word-builder-stats');
  if (!raw) {
    return { correctById: {} };
  }

  try {
    return JSON.parse(raw) as WordBuilderStats;
  } catch {
    return { correctById: {} };
  }
}

function loadMemoryGameStats(): MemoryGameStats {
  if (typeof window === 'undefined') {
    return { gamesPlayed: 0, gamesWon: 0, bestScore: 0 };
  }

  const raw = window.localStorage.getItem('memory-game-stats');
  if (!raw) {
    return { gamesPlayed: 0, gamesWon: 0, bestScore: 0 };
  }

  try {
    return JSON.parse(raw) as MemoryGameStats;
  } catch {
    return { gamesPlayed: 0, gamesWon: 0, bestScore: 0 };
  }
}

function loadHangmanStats(): HangmanStats {
  if (typeof window === 'undefined') {
    return { gamesPlayed: 0, gamesWon: 0, currentStreak: 0, maxStreak: 0 };
  }

  const raw = window.localStorage.getItem('hangman-stats');
  if (!raw) {
    return { gamesPlayed: 0, gamesWon: 0, currentStreak: 0, maxStreak: 0 };
  }

  try {
    return JSON.parse(raw) as HangmanStats;
  } catch {
    return { gamesPlayed: 0, gamesWon: 0, currentStreak: 0, maxStreak: 0 };
  }
}

onMounted(() => {
  wordBuilderStats.value = loadWordBuilderStats();
  memoryGameStats.value = loadMemoryGameStats();
  hangmanStats.value = loadHangmanStats();
});

const gameStats = computed<Record<GameDefinition['id'], GameStat>>(() => ({
  'game-word-builder': {
    title: 'Palavras montadas',
    value: wordBuilderStats.value ? Object.keys(wordBuilderStats.value.correctById).length : 0,
    icon: 'spellcheck',
    description: 'Palavras diferentes já montadas',
  },
  'game-memory': {
    title: 'Jogos ganhos',
    value: memoryGameStats.value?.gamesWon || 0,
    icon: 'emoji_events',
    description: 'Partidas vencidas no jogo da memória',
  },
  'game-hangman': {
    title: 'Sequência atual',
    value: hangmanStats.value?.currentStreak || 0,
    icon: 'local_fire_department',
    description: 'vitórias seguidas',
  },
}));

const router = useRouter();

function goToGame(routeName: string) {
  void router.push({ name: routeName });
}
</script>
