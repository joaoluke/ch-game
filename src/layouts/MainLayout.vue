<template>
  <q-layout view="lHh Lpr lFf" class="kids-layout">
    <q-header elevated class="kids-header">
      <q-toolbar>
        <q-btn flat dense round icon="menu" aria-label="Menu" @click="toggleLeftDrawer" />

        <q-toolbar-title>Jogos do Charles</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header>Jogos</q-item-label>

        <q-item
          v-for="game in games"
          :key="game.id"
          clickable
          v-ripple
          @click="goToGame(game.routeName)"
        >
          <q-item-section avatar>
            <q-icon :name="game.icon" />
          </q-item-section>

          <q-item-section>
            <q-item-label>{{ game.title }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';

interface GameLink {
  id: string;
  title: string;
  icon: string;
  routeName: string;
}

const games = ref<GameLink[]>([
  {
    id: 'game-word-builder',
    title: 'Montar a palavra',
    icon: 'spellcheck',
    routeName: 'game-word-builder',
  },
  {
    id: 'game-memory',
    title: 'Jogo da Memória',
    icon: 'memory',
    routeName: 'game-memory',
  },
  {
    id: 'game-hangman',
    title: 'Jogo da Forca',
    icon: 'sentiment_very_satisfied',
    routeName: 'game-hangman',
  },
]);

const leftDrawerOpen = ref(false);
const router = useRouter();

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}

function goToGame(routeName: string) {
  leftDrawerOpen.value = false;
  void router.push({ name: routeName });
}
</script>
