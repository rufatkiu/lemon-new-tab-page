<script setup lang="ts">
import { ColorLensOutlined } from '@vicons/material'
import { computed, ref, watch } from 'vue'
import { useColorMode, useDark, useTimeoutFn, usePreferredDark } from '@vueuse/core'

import changeTheme from '@/newtab/scripts/use-element-plus-theme'
import { i18n } from '@/.wxt/i18n'
import { useSettingsStore } from '@/newtab/scripts/store/settingsStore'

const isGoogleChrome = import.meta.env.CHROME && !import.meta.env.EDGE
const settingsStore = useSettingsStore()
const { store } = useColorMode()

const predefineColors = ref([
  '#ff4500',
  '#ff8c00',
  '#ffd700',
  '#90ee90',
  '#00ced1',
  '#1e90ff',
  '#c71585',
  '#1677ff',
  '#FFBB00'
])

const isDark = useDark()
const isDarkLocal = ref(isDark.value)
const preferredDark = usePreferredDark()

const isAuto = computed(() => store.value === 'auto')
const isAutoLocal = ref(isAuto.value)

function changeByPreferred(htmlElement: HTMLHtmlElement) {
  if (preferredDark.value) {
    htmlElement.classList.add('dark')
    htmlElement.classList.remove('light')
    isDarkLocal.value = true
  } else {
    htmlElement.classList.add('light')
    htmlElement.classList.remove('dark')
    isDarkLocal.value = false
  }
}

function changeByUser(htmlElement: HTMLHtmlElement) {
  if (isDarkLocal.value) {
    htmlElement.classList.add('dark')
    htmlElement.classList.remove('light')
    isAutoLocal.value = false
  } else {
    htmlElement.classList.add('light')
    htmlElement.classList.remove('dark')
    isAutoLocal.value = false
  }
}

function toggleDark() {
  const htmlElement: HTMLHtmlElement | null = document.querySelector('html')
  if (!htmlElement) return
  if (isDark.value) {
    // 先切换CSS，等待动画结束后，再切换store
    changeByUser(htmlElement)
    useTimeoutFn(() => {
      store.value = 'light'
    }, 300)
  } else {
    changeByUser(htmlElement)
    useTimeoutFn(() => {
      store.value = 'dark'
    }, 300)
  }
}

function toggleAuto() {
  const htmlElement: HTMLHtmlElement | null = document.querySelector('html')
  if (!htmlElement) return
  if (!isAutoLocal.value) {
    store.value = isDarkLocal.value ? 'dark' : 'light'
    return
  }

  if (isDarkLocal.value !== preferredDark.value) {
    // 先切换CSS，等待动画结束后，再切换store
    changeByPreferred(htmlElement)
    useTimeoutFn(() => {
      store.value = 'auto'
    }, 300)
  } else {
    store.value = 'auto'
  }
}

watch(preferredDark, () => {
  const doc: HTMLHtmlElement | null = document.querySelector('html')
  if (!doc) return

  if (isAuto.value) {
    if (isDarkLocal.value !== preferredDark.value) {
      changeByPreferred(doc)
    }
  }
})
</script>

<template>
  <div class="settings-title">
    <el-icon><color-lens-outlined /></el-icon>
    <span>{{ i18n.t('newtab.settings.theme.title') }}</span>
  </div>
  <div class="setting-items-container">
    <div class="settings-item horizontal">
      <div class="settings-label">{{ i18n.t('newtab.settings.theme.darkMode') }}</div>
      <el-switch v-model="isDarkLocal" @change="toggleDark" />
    </div>
    <div class="settings-item horizontal">
      <div class="settings-label">{{ i18n.t('newtab.settings.theme.systemMode') }}</div>
      <el-switch v-model="isAutoLocal" @change="toggleAuto" />
    </div>
    <p
      v-if="isGoogleChrome"
      style="
        color: var(--el-text-color-regular);
        line-height: 1.5em;
        font-size: 12px;
        margin-top: 0;
      "
    >
      {{ i18n.t('newtab.settings.theme.chromeTip') }}
    </p>
    <div class="settings-item horizontal">
      <div class="settings-label">{{ i18n.t('newtab.settings.theme.primaryColor') }}</div>
      <div class="color-mode">
        <el-color-picker
          v-model="settingsStore.primaryColor"
          :predefine="predefineColors"
          @change="
            () => {
              if (settingsStore.primaryColor === null) {
                settingsStore.primaryColor = '#1677ff'
              }
              changeTheme(settingsStore.primaryColor)
            }
          "
        />
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.theme-mode {
  display: flex;
  column-gap: 8px;

  .theme-item {
    position: relative;
    cursor: pointer;

    & .selected {
      display: none;
    }
    &.active .selected {
      display: block;
      position: absolute;
      right: 8px;
      bottom: 8px;
      color: var(--el-color-primary);
    }
  }
}

:deep().el-color-picker__trigger {
  padding: 0;
  border: none;
  overflow: hidden;
  border-radius: 8px;

  .el-color-picker__color {
    border: none;
    border-radius: none;
  }
}
</style>
