<template>
  <v-app>
    <v-app-bar color="primary" app>
      <v-spacer></v-spacer>
      <v-btn icon @click="switchTheme">
        <v-icon>
          {{ theme.global.current.value.dark ? 'mdi-white-balance-sunny' : 'mdi-weather-night' }}
        </v-icon>
      </v-btn>
    </v-app-bar>
    <default-view />
  </v-app>
</template>

<script setup>
import { useTheme } from 'vuetify';
import DefaultView from './View.vue'

const theme = useTheme()

const switchTheme = () => theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'
</script>
