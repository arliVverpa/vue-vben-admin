<script setup lang="ts">
import type { AnyFunction } from '@vben-core/typings';

import type { Component } from 'vue';
import { computed, ref } from 'vue';

import {
  IcRoundLock,
  IcRoundLogout,
  IcRoundSettingsSuggest,
} from '@vben-core/iconify';
import { $t } from '@vben-core/locales';
import { preferences, usePreferences } from '@vben-core/preferences';
import {
  Badge,
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuShortcut,
  DropdownMenuTrigger,
  VbenAlertDialog,
  VbenAvatar,
  VbenIcon,
} from '@vben-core/shadcn-ui';
import { isWindowsOs } from '@vben-core/toolkit';

import { useMagicKeys, whenever } from '@vueuse/core';

import { LockScreenModal } from '../lock-screen';
import { useOpenPreferences } from '../preferences';

interface Props {
  /**
   * 头像
   */
  avatar?: string;
  /**
   * @zh_CN 描述
   */
  description?: string;
  /**
   * 是否启用快捷键
   */
  enableShortcutKey?: boolean;
  /**
   * 菜单数组
   */
  menus?: Array<{ handler: AnyFunction; icon?: Component; text: string }>;

  /**
   * 标签文本
   */
  tagText?: string;
  /**
   * 文本
   */
  text?: string;
}

defineOptions({
  name: 'UserDropdown',
});

const props = withDefaults(defineProps<Props>(), {
  avatar: '',
  description: '',
  enableShortcutKey: true,
  menus: () => [],
  showShortcutKey: true,
  tagText: '',
  text: '',
});

const emit = defineEmits<{ lockScreen: [string]; logout: [] }>();
const openPopover = ref(false);
const openDialog = ref(false);
const openLock = ref(false);

const {
  globalLockScreenShortcutKey,
  globalLogoutShortcutKey,
  globalPreferencesShortcutKey,
} = usePreferences();
const { handleOpenPreference } = useOpenPreferences();

const altView = computed(() => (isWindowsOs() ? 'Alt' : '⌥'));

const enableLogoutShortcutKey = computed(() => {
  return props.enableShortcutKey && globalLogoutShortcutKey.value;
});

const enableLockScreenShortcutKey = computed(() => {
  return props.enableShortcutKey && globalLockScreenShortcutKey.value;
});

const enableShortcutKey = computed(() => {
  return props.enableShortcutKey && preferences.shortcutKeys.enable;
});

const enablePreferencesShortcutKey = computed(() => {
  return props.enableShortcutKey && globalPreferencesShortcutKey.value;
});

function handleOpenLock() {
  openLock.value = true;
}

function handleSubmitLock({
  lockScreenPassword,
}: {
  lockScreenPassword: string;
}) {
  openLock.value = false;
  emit('lockScreen', lockScreenPassword);
}
function handleLogout() {
  // emit
  openDialog.value = true;
  openPopover.value = false;
}

function handleSubmitLogout() {
  emit('logout');
  openDialog.value = false;
}

if (enableShortcutKey.value) {
  const keys = useMagicKeys();
  whenever(keys['Alt+KeyQ'], () => {
    if (enableLogoutShortcutKey.value) {
      handleLogout();
    }
  });

  whenever(keys['Alt+Comma'], () => {
    if (enablePreferencesShortcutKey.value) {
      handleOpenPreference();
    }
  });

  whenever(keys['Alt+KeyL'], () => {
    if (enableLockScreenShortcutKey.value) {
      handleOpenLock();
    }
  });
}
</script>

<template>
  <LockScreenModal
    v-if="preferences.widget.lockScreen"
    v-model:open="openLock"
    :avatar="avatar"
    :text="text"
    @submit="handleSubmitLock"
  />
  <VbenAlertDialog
    v-model:open="openDialog"
    :cancel-text="$t('common.cancel')"
    :content="$t('widgets.logoutTip')"
    :submit-text="$t('common.confirm')"
    :title="$t('common.prompt')"
    @submit="handleSubmitLogout"
  />

  <DropdownMenu>
    <DropdownMenuTrigger>
      <div class="hover:bg-accent ml-1 mr-2 cursor-pointer rounded-full p-1.5">
        <div class="hover:text-accent-foreground flex-center">
          <VbenAvatar :alt="text" :src="avatar" class="size-8" dot />
        </div>
      </div>
    </DropdownMenuTrigger>
    <DropdownMenuContent class="mr-2 min-w-[240px] p-0 pb-1">
      <DropdownMenuLabel class="flex items-center p-3">
        <VbenAvatar
          :alt="text"
          :src="avatar"
          class="size-12"
          dot
          dot-class="bottom-0 right-1 border-2 size-4 bg-green-500"
        />
        <div class="ml-2 w-full">
          <div
            class="text-foreground mb-1 flex items-center text-sm font-medium"
          >
            {{ text }}
            <Badge class="ml-2 text-green-400">
              {{ tagText }}
            </Badge>
          </div>
          <div class="text-muted-foreground text-xs font-normal">
            {{ description }}
          </div>
        </div>
      </DropdownMenuLabel>
      <DropdownMenuSeparator />
      <DropdownMenuItem
        v-for="menu in menus"
        :key="menu.text"
        class="mx-1 flex cursor-pointer items-center rounded-sm py-1 leading-8"
        @click="menu.handler"
      >
        <VbenIcon :icon="menu.icon" class="mr-2 size-5" />
        {{ menu.text }}
      </DropdownMenuItem>
      <DropdownMenuSeparator />
      <DropdownMenuItem
        class="mx-1 flex cursor-pointer items-center rounded-sm py-1 leading-8"
        @click="handleOpenPreference"
      >
        <IcRoundSettingsSuggest class="mr-2 size-5" />
        {{ $t('preferences.title') }}
        <DropdownMenuShortcut v-if="enablePreferencesShortcutKey">
          {{ altView }} ,
        </DropdownMenuShortcut>
      </DropdownMenuItem>
      <DropdownMenuItem
        v-if="preferences.widget.lockScreen"
        class="mx-1 flex cursor-pointer items-center rounded-sm py-1 leading-8"
        @click="handleOpenLock"
      >
        <IcRoundLock class="mr-2 size-5" />
        {{ $t('widgets.lockScreen.title') }}
        <DropdownMenuShortcut v-if="enableLockScreenShortcutKey">
          {{ altView }} L
        </DropdownMenuShortcut>
      </DropdownMenuItem>
      <DropdownMenuSeparator />
      <DropdownMenuItem
        class="mx-1 flex cursor-pointer items-center rounded-sm py-1 leading-8"
        @click="handleLogout"
      >
        <IcRoundLogout class="mr-2 size-5" />
        {{ $t('common.logout') }}
        <DropdownMenuShortcut v-if="enableLogoutShortcutKey">
          {{ altView }} Q
        </DropdownMenuShortcut>
      </DropdownMenuItem>
    </DropdownMenuContent>
  </DropdownMenu>
</template>