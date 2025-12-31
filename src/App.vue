<script setup lang="ts">
import { NConfigProvider, NMessageProvider } from 'naive-ui'
import HangToLaRanker from './components/HangToLaRanker.vue'
</script>

<template>
  <n-config-provider>
    <n-message-provider>
      <HangToLaRanker />
    </n-message-provider>
  </n-config-provider>
</template>

<style>
body {
  margin: 0;
  background-color: #f5f5f5;
}
</style>
