<template>
  <q-page class="kids-page q-pa-md column items-center">
    <q-btn
      flat
      round
      color="primary"
      icon="arrow_back"
      class="q-mb-md self-start"
      @click="goBack"
    />

    <div class="text-h4 q-mb-md">Jogo da Forca</div>

    <div v-if="!gameStarted" class="text-center q-pa-lg">
      <q-btn color="primary" label="Começar Jogo" size="lg" @click="startGame" />
    </div>

    <div v-else class="column items-center">
      <!-- Desenho da forca -->
      <div class="q-mb-lg" style="height: 200px; position: relative">
        <svg width="200" height="200" viewBox="0 0 200 200">
          <!-- Base -->
          <line x1="20" y1="180" x2="100" y2="180" stroke="brown" stroke-width="10" />
          <line x1="40" y1="180" x2="40" y2="20" stroke="brown" stroke-width="8" />
          <line x1="40" y1="20" x2="120" y2="20" stroke="brown" stroke-width="6" />
          <line x1="120" y1="20" x2="120" y2="50" stroke="brown" stroke-width="4" />

          <!-- Cabeça -->
          <circle
            v-if="wrongGuesses >= 1"
            cx="120"
            cy="70"
            r="20"
            fill="none"
            stroke="black"
            stroke-width="3"
          />

          <!-- Corpo -->
          <line
            v-if="wrongGuesses >= 2"
            x1="120"
            y1="90"
            x2="120"
            y2="140"
            stroke="black"
            stroke-width="3"
          />

          <!-- Braço esquerdo -->
          <line
            v-if="wrongGuesses >= 3"
            x1="120"
            y1="100"
            x2="90"
            y2="120"
            stroke="black"
            stroke-width="3"
          />

          <!-- Braço direito -->
          <line
            v-if="wrongGuesses >= 4"
            x1="120"
            y1="100"
            x2="150"
            y2="120"
            stroke="black"
            stroke-width="3"
          />

          <!-- Perna esquerda -->
          <line
            v-if="wrongGuesses >= 5"
            x1="120"
            y1="140"
            x2="100"
            y2="170"
            stroke="black"
            stroke-width="3"
          />

          <!-- Perna direita -->
          <line
            v-if="wrongGuesses >= 6"
            x1="120"
            y1="140"
            x2="140"
            y2="170"
            stroke="black"
            stroke-width="3"
          />
        </svg>
      </div>

      <!-- Dica visual -->
      <div class="q-mb-md text-center">
        <q-icon :name="currentWord.icon" size="xl" class="q-mb-sm" />
      </div>

      <!-- Palavra oculta -->
      <div class="row q-gutter-sm q-mb-lg">
        <div v-for="(letter, index) in currentWord.word" :key="index" class="letter-box">
          {{ guessedLetters.includes(letter) ? letter : '_' }}
        </div>
      </div>

      <!-- Teclado virtual -->
      <div class="q-mb-lg">
        <div v-for="(row, rowIndex) in keyboard" :key="rowIndex" class="row justify-center q-mb-sm">
          <q-btn
            v-for="letter in row"
            :key="letter"
            :label="letter"
            :color="getButtonColor(letter)"
            :disable="!gameStarted || guessedLetters.includes(letter) || gameOver"
            class="q-mx-xs"
            @click="guessLetter(letter)"
          />
        </div>
      </div>

      <!-- Mensagens de jogo -->
      <q-banner v-if="message" :class="messageClass" class="q-mb-md">
        {{ message }}
      </q-banner>

      <!-- Botão de reiniciar -->
      <q-btn v-if="gameOver" color="primary" label="Jogar Novamente" @click="startGame" />
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { wordBuilderWords } from 'src/data/wordBuilderWords';

const router = useRouter();

// Estado do jogo
const gameStarted = ref(false);
const gameOver = ref(false);
const gameWon = ref(false);
const currentWord = ref<{ word: string; icon: string }>({ word: '', icon: '' });
const guessedLetters = ref<string[]>([]);
const wrongGuesses = ref(0);
const message = ref('');
const messageClass = ref('');

// Teclado virtual
const keyboard = [
  ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
  ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', 'Ç'],
  ['Z', 'X', 'C', 'V', 'B', 'N', 'M'],
];

// Escolhe uma palavra aleatória
function getRandomWord() {
  const randomIndex = Math.floor(Math.random() * wordBuilderWords.length);
  const word = wordBuilderWords[randomIndex];
  if (!word) {
    // Retorna uma palavra padrão caso algo dê errado
    return { word: 'PALAVRA', icon: 'help' };
  }
  return word;
}

// Inicia um novo jogo
function startGame() {
  gameStarted.value = true;
  gameOver.value = false;
  gameWon.value = false;
  guessedLetters.value = [];
  wrongGuesses.value = 0;
  message.value = '';

  const wordObj = getRandomWord();
  currentWord.value = {
    word: wordObj.word,
    icon: wordObj.icon,
  };

  // Adiciona algumas dicas iniciais (vogais)
  const vowels = ['A', 'E', 'I', 'O', 'U'];
  const wordVowels = [
    ...new Set(currentWord.value.word.split('').filter((l: string) => vowels.includes(l))),
  ];
  if (wordVowels.length > 0) {
    const hintVowel = wordVowels[Math.floor(Math.random() * wordVowels.length)];
    if (hintVowel) {
      guessedLetters.value.push(hintVowel);
    }
  }

  // Adiciona a primeira letra como dica
  const firstLetter = currentWord.value.word[0];
  if (firstLetter && !guessedLetters.value.includes(firstLetter)) {
    guessedLetters.value.push(firstLetter);
  }
}

// Tenta adivinhar uma letra
function guessLetter(letter: string) {
  if (guessedLetters.value.includes(letter) || gameOver.value) return;

  guessedLetters.value.push(letter);

  if (!currentWord.value.word.includes(letter)) {
    wrongGuesses.value++;
    showMessage('Letra incorreta!', 'negative');

    if (wrongGuesses.value >= 6) {
      endGame(false);
    }
  } else {
    showMessage('Boa!', 'positive');

    // Verifica se o jogador ganhou
    const wordLetters = [...new Set(currentWord.value.word.split(''))];
    const allGuessed = wordLetters.every((l) => guessedLetters.value.includes(l));

    if (allGuessed) {
      endGame(true);
    }
  }
}

// Finaliza o jogo
function endGame(won: boolean) {
  gameOver.value = true;
  gameWon.value = won;

  if (won) {
    showMessage('Parabéns! Você venceu!', 'positive');
    saveGameResult(true);
  } else {
    showMessage(`Fim de jogo! A palavra era ${currentWord.value.word}`, 'negative');
    saveGameResult(false);
  }
}

// Mostra uma mensagem temporária
function showMessage(msg: string, type: 'positive' | 'negative' | 'info' = 'info') {
  message.value = msg;
  messageClass.value = `bg-${type} text-white`;

  setTimeout(() => {
    message.value = '';
    messageClass.value = '';
  }, 2000);
}

// Salva o resultado do jogo
function saveGameResult(won: boolean) {
  try {
    const stats = JSON.parse(localStorage.getItem('hangman-stats') || '{}') as {
      gamesPlayed: number;
      gamesWon: number;
      currentStreak: number;
      maxStreak: number;
    };

    const currentStats = {
      gamesPlayed: (stats.gamesPlayed || 0) + 1,
      gamesWon: (stats.gamesWon || 0) + (won ? 1 : 0),
      currentStreak: won ? (stats.currentStreak || 0) + 1 : 0,
      maxStreak: Math.max(stats.maxStreak || 0, won ? (stats.currentStreak || 0) + 1 : 0),
    };

    localStorage.setItem('hangman-stats', JSON.stringify(currentStats));
  } catch (error) {
    console.error('Erro ao salvar estatísticas do jogo da forca:', error);
  }
}

// Obtém a cor do botão do teclado
function getButtonColor(letter: string) {
  if (!guessedLetters.value.includes(letter)) return 'primary';
  return currentWord.value.word.includes(letter) ? 'positive' : 'negative';
}

// Navega de volta
async function goBack() {
  await router.push('/');
}

// Inicia o jogo quando o componente é montado
onMounted(() => {
  startGame();
});
</script>

<style scoped>
.letter-box {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-bottom: 3px solid #1976d2;
  font-size: 24px;
  font-weight: bold;
  margin: 0 2px;
}

.q-banner {
  border-radius: 4px;
  padding: 8px 16px;
  margin-top: 10px;
}
</style>
