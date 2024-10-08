<script setup lang="ts">
import {
  Sheet,
  SheetContent,
  SheetDescription,
  SheetHeader,
  SheetTitle,
  SheetTrigger
} from '@/components/ui/sheet'
import { RouterLink, useRoute, useRouter } from 'vue-router'
import { Menu } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'
import GameLoader from '@/components/GameLoader.vue'

import { GAME_TEST_ID, useGameStore } from '@/stores/game'
import { ref } from 'vue'

const router = useRouter()
const route = useRoute()
const gameStore = useGameStore()
const open = ref(false)

function viewGame() {
  open.value = false

  if (route.name !== 'map') {
    router.push({ name: 'map' })
  }
}

function loadTestGame(id: string) {
  gameStore.loadGame(id)
  viewGame()
}
</script>

<template>
  <Sheet v-model:open="open">
    <SheetTrigger as-child>
      <Button
        class="w-12 h-12 p-2 ml-2 menu"
        size="icon"
        variant="ghost"
      >
        <Menu :size="48" />
      </Button>
    </SheetTrigger>
    <SheetContent side="left">
      <SheetHeader>
        <SheetTitle class="text-center">
          {{ $t('arcs_act') }}
        </SheetTitle>
        <SheetDescription>
          <img
            :alt="$t('arcs_act')"
            class="logo"
            src="@/assets/images/archivist.png?h=128"
          />
        </SheetDescription>
      </SheetHeader>

      <ul @click="open = false">
        <li>
          <RouterLink to="/">
            {{ $t('common.home') }}
          </RouterLink>
        </li>
        <li>
          <RouterLink :to="{ path: '/campaign', query: { mode: 'create' } }">
            {{ $t('campaign.new') }}
          </RouterLink>
        </li>
        <!-- <li>
          <RouterLink :to="{ path: '/campaign', query: { mode: 'edit' } }">
            {{ $t('campaign.current') }}
          </RouterLink>
        </li>
        <li>
          <RouterLink to="/map">
            {{ $t('campaign.map_view') }}
          </RouterLink>
        </li>
        <li>
          <RouterLink to="/list">
            {{ $t('campaign.list_view') }}
          </RouterLink>
        </li> -->
        <li>
          <GameLoader @loaded="viewGame">
            <Button
              class="w-full my-2"
              @click.stop
            >
              {{ $t('common.load') }} / {{ $t('common.export') }}
            </Button>
          </GameLoader>
        </li>
        <li>
          <Button
            class="w-full my-2"
            @click="loadTestGame(GAME_TEST_ID)"
            >{{ $t('common.load') }} test</Button
          >
        </li>
      </ul>
    </SheetContent>
  </Sheet>
</template>

<style scoped>
.logo {
  height: 128px;
  margin: 0 auto;
}

.menu {
  /* position: fixed; */
  top: 8px;
  left: 8px;
}

li > * {
  display: inline-block;
  padding-top: theme('spacing.2');
  padding-bottom: theme('spacing.2');
}
</style>
