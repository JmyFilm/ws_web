<script setup>
import { reactive, computed } from 'vue'
import {  RouterView } from 'vue-router'
import { zhCN, dateZhCN, useOsTheme, darkTheme, NConfigProvider, NMessageProvider } from 'naive-ui'

const s = reactive({
  theme: computed(() => (useOsTheme().value === 'dark' ? darkTheme : null)),
})
</script>

<template>
  <n-config-provider   :theme="s.theme" :locale="zhCN" :date-locale="dateZhCN">
    <n-message-provider :max="1">
      <RouterView class="r" />
    </n-message-provider>
  </n-config-provider>
</template>

<style>
  body {
    padding: 0;
    margin: 0;
  }
  .r {
    /* min-height: 100vh; */
    /* border: 1px solid #000; */
    /* background-color: #054948; */
  }
</style>
