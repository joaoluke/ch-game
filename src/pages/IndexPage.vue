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

          <div v-if="game.id === 'game-word-builder'" class="text-caption q-mt-sm">
            Já montou
            {{ wordBuilderSummary.distinctWords }} palavra(s) em
            {{ wordBuilderSummary.totalCorrect }} acerto(s).
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
  id: string;
  title: string;
  description: string;
  icon: string;
  routeName: string;
}

interface WordBuilderStats {
  correctById: Record<string, number>;
}

const games = ref<GameDefinition[]>([
  {
    id: 'game-word-builder',
    title: 'Montar a palavra',
    description: 'Olhe a figura e monte a palavra letra por letra.',
    icon: 'spellcheck',
    routeName: 'game-word-builder',
  },
]);

const wordBuilderStats = ref<WordBuilderStats | null>(null);

function loadWordBuilderStats(): WordBuilderStats {
  if (typeof window === 'undefined') {
    return { correctById: {} };
  }

  const raw = window.localStorage.getItem('word-builder-stats');
  if (!raw) {
    return { correctById: {} };
  }

  try {
    const parsed = JSON.parse(raw) as WordBuilderStats;
    if (!parsed.correctById || typeof parsed.correctById !== 'object') {
      return { correctById: {} };
    }
    return { correctById: { ...parsed.correctById } };
  } catch {
    return { correctById: {} };
  }
}

onMounted(() => {
  wordBuilderStats.value = loadWordBuilderStats();
});

const wordBuilderSummary = computed(() => {
  const stats = wordBuilderStats.value;
  if (!stats) {
    return { distinctWords: 0, totalCorrect: 0 };
  }

  const counts = Object.values(stats.correctById);
  const distinctWords = counts.length;
  const totalCorrect = counts.reduce((sum, value) => sum + value, 0);

  return { distinctWords, totalCorrect };
});

const router = useRouter();

function goToGame(routeName: string) {
  void router.push({ name: routeName });
}
</script>
