<template>
  <q-page class="q-pa-md column items-center">
    <div class="full-width row items-center q-mb-md">
      <q-btn flat round icon="arrow_back" @click="goBack" />
      <div class="text-h5 q-ml-sm">Montar a palavra</div>
    </div>

    <div class="q-mt-lg q-mb-lg flex flex-center" style="width: 260px; height: 180px">
      <q-card class="full-width full-height column flex flex-center bg-amber-1">
        <q-card-section class="column items-center justify-center">
          <q-icon :name="currentWord.icon" size="64px" class="q-mb-sm text-amber-8" />
          <div class="text-subtitle1">Que palavra é essa?</div>
        </q-card-section>
      </q-card>
    </div>

    <div class="q-mb-md text-subtitle1">Toque nas letras para formar a palavra:</div>

    <div class="row justify-center q-gutter-sm q-mb-lg">
      <q-chip
        v-for="(letter, index) in selectedLetters"
        :key="`selected-${index}-${letter}`"
        color="primary"
        text-color="white"
        size="lg"
      >
        {{ letter }}
      </q-chip>
    </div>

    <div class="row justify-center q-gutter-sm q-mb-md">
      <q-btn
        v-for="(letter, index) in availableLetters"
        :key="`available-${index}-${letter}`"
        color="secondary"
        round
        size="lg"
        @click="selectLetter(index)"
      >
        {{ letter }}
      </q-btn>
    </div>

    <div class="row justify-center q-gutter-sm q-mt-md">
      <q-btn flat color="negative" label="Limpar" @click="resetSelection" />
    </div>

    <q-dialog
      v-model="showSuccessDialog"
      persistent
      transition-show="scale"
      transition-hide="scale"
    >
      <q-card class="kids-success-card q-pa-md">
        <q-card-section class="column items-center">
          <q-icon name="star" size="48px" class="q-mb-sm text-amber-7" />
          <div class="kids-title text-h5 q-mb-xs">Muito bem!</div>
          <div class="kids-subtitle text-subtitle1 text-center">
            Você montou a palavra corretamente: {{ currentWord.word }}.
          </div>
        </q-card-section>
        <q-card-actions align="center">
          <q-btn color="primary" unelevated label="Próxima palavra" @click="handlePlayAgain" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { useRouter } from 'vue-router';
import { type WordDefinition, wordBuilderWords } from 'src/data/wordBuilderWords';

interface WordBuilderStats {
  correctById: Record<string, number>;
}

const router = useRouter();

const STORAGE_KEY = 'word-builder-stats';

function loadStats(): WordBuilderStats {
  if (typeof window === 'undefined') {
    return { correctById: {} };
  }

  const raw = window.localStorage.getItem(STORAGE_KEY);
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

const fallbackWord: WordDefinition = {
  id: 'fallback',
  word: '',
  icon: 'help',
};

const stats = ref<WordBuilderStats>(loadStats());

const words = ref<WordDefinition[]>(
  wordBuilderWords.filter((word) => (stats.value.correctById[word.id] ?? 0) === 0),
);

const currentIndex = ref(0);

const currentWord = computed<WordDefinition>(() => words.value[currentIndex.value] ?? fallbackWord);

const selectedLetters = ref<string[]>([]);
const availableLetters = ref<string[]>([]);

function saveStats() {
  if (typeof window === 'undefined') {
    return;
  }

  window.localStorage.setItem(STORAGE_KEY, JSON.stringify(stats.value));
}

function registerSuccessForCurrentWord() {
  const id = currentWord.value.id;
  const current = stats.value.correctById[id] ?? 0;
  stats.value.correctById[id] = current + 1;
  saveStats();
}

function getAttemptsFor(wordId: string): number {
  return stats.value.correctById[wordId] ?? 0;
}

function getNextIndexPreferringUnseen(): number {
  const len = words.value.length;
  if (len === 0) {
    return 0;
  }

  const availableWords = words.value
    .map((word, idx) => ({ word, idx }))
    .filter(({ word }) => getAttemptsFor(word.id) === 0);

  if (availableWords.length === 0) {
    stats.value.correctById = {};
    saveStats();
    return 0;
  }

  const availableIndexes = availableWords.map(({ idx }) => idx);

  const nextAfterCurrent = availableIndexes.find((idx) => idx > currentIndex.value);
  return nextAfterCurrent ?? availableIndexes[0]!;
}

function shuffle(array: string[]): string[] {
  const result = array.slice();
  for (let i = result.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1));
    const temp = result[i]!;
    result[i] = result[j]!;
    result[j] = temp;
  }
  return result;
}

function setupLetters() {
  const letters = currentWord.value.word.split('');
  selectedLetters.value = [];
  availableLetters.value = shuffle(letters);
}

function selectLetter(index: number) {
  const letter = availableLetters.value[index];
  if (!letter) {
    return;
  }
  selectedLetters.value.push(letter);
  availableLetters.value.splice(index, 1);
}

function resetSelection() {
  setupLetters();
}

const isCompleted = computed(() => selectedLetters.value.length === currentWord.value.word.length);

const isCorrect = computed(() => selectedLetters.value.join('') === currentWord.value.word);

const showSuccessDialog = ref(false);

watch(
  () => ({ completed: isCompleted.value, correct: isCorrect.value }),
  ({ completed, correct }) => {
    if (completed && correct) {
      registerSuccessForCurrentWord();
      showSuccessDialog.value = true;
    }
  },
);

function handlePlayAgain() {
  showSuccessDialog.value = false;

  if (words.value.length > 0) {
    currentIndex.value = getNextIndexPreferringUnseen();
  }

  setupLetters();
}

function goBack() {
  void router.push({ path: '/' });
}

setupLetters();
</script>
