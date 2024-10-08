<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger
} from '@/components/ui/dropdown-menu'
import { Redo, Sparkles } from 'lucide-vue-next'
import { Color, ShipType, TokenType } from '@/Archive'
import { OnClickOutside } from '@vueuse/components'
import { isFlagship } from '@/lib/utils'

import type { PieceState, PieceStateGroup } from '@/stores/systems'

const props = defineProps<{
  activePiece: PieceState | PieceStateGroup
  pointerPosition: {
    x: number
    y: number
  }
}>()

const emit = defineEmits<{
  add: [type: TokenType | ShipType, color?: Color]
  remove: [isFresh: boolean]
  flip: [isFresh: boolean]
  preview: []
  close: []
}>()

const { t } = useI18n()

const triggerStyle = computed(() => ({
  left: `${props.pointerPosition.x}px`,
  top: `${props.pointerPosition.y}px`
}))
// For some reason i18n modifiers are not working with variables
const pieceId = computed(() => {
  const color = t(`colors.${props.activePiece.color}`)
  const type = t(`pieces.${props.activePiece.type}`, 'group' in props.activePiece ? 2 : 1)
  const str = props.activePiece.color ? `${color} ${type}` : type
  return `${str.charAt(0).toUpperCase()}${str.slice(1).toLowerCase()}`
})

const hasFresh = computed(() => {
  if ('group' in props.activePiece) {
    return props.activePiece.group.fresh.length > 0
  }

  return props.activePiece.isFresh
})

const hasDamaged = computed(() => {
  if ('group' in props.activePiece) {
    return props.activePiece.group.damaged.length > 0
  }

  return !props.activePiece.isFresh
})

function onAdd() {
  if (props.activePiece.color) {
    emit('add', props.activePiece.type as ShipType, props.activePiece.color)
  } else {
    emit('add', props.activePiece.type as TokenType)
  }
}

function onRemove(isFresh: boolean = true) {
  emit('remove', isFresh)
}

function onFlip(isFresh: boolean = true) {
  emit('flip', isFresh)
}

function preview() {
  emit('preview')
}
</script>

<template>
  <DropdownMenu
    :default-open="true"
    :modal="false"
    @update:open="emit('close')"
  >
    <DropdownMenuTrigger
      class="fixed dropdown-trigger"
      :style="triggerStyle"
    ></DropdownMenuTrigger>
    <DropdownMenuContent>
      <OnClickOutside @trigger="emit('close')">
        <DropdownMenuLabel>{{ pieceId }}</DropdownMenuLabel>
        <DropdownMenuSeparator />
        <!-- PieceStateGroup menu -->
        <template v-if="'group' in activePiece">
          <DropdownMenuItem @select="onAdd">
            <Sparkles
              class="mr-2"
              :size="16"
            />
            {{ $t('piece_menu.add') }}
          </DropdownMenuItem>
          <DropdownMenuItem
            v-if="hasFresh"
            @select="onFlip(true)"
          >
            <Sparkles
              class="mr-2"
              :size="16"
            />
            {{ $t('piece_menu.damage_fresh') }}
          </DropdownMenuItem>
          <DropdownMenuItem
            v-if="hasFresh"
            @select="onRemove(true)"
          >
            <Sparkles
              class="mr-2"
              :size="16"
            />
            {{ $t('piece_menu.remove_fresh') }}
          </DropdownMenuItem>
          <DropdownMenuItem
            v-if="hasDamaged"
            @select="onFlip(false)"
          >
            <Redo
              class="mr-2"
              :size="16"
            />
            {{ $t('piece_menu.repair_damaged') }}
          </DropdownMenuItem>
          <DropdownMenuItem
            v-if="hasDamaged"
            @select="onRemove(false)"
          >
            <Redo
              class="mr-2"
              :size="16"
            />
            {{ $t('piece_menu.remove_damaged') }}
          </DropdownMenuItem>
        </template>
        <!-- PieceState menu -->
        <template v-else>
          <DropdownMenuItem
            v-if="!isFlagship(activePiece.type)"
            @select="onFlip(activePiece.isFresh)"
          >
            {{ activePiece.isFresh ? $t('piece_menu.damage') : $t('piece_menu.repair') }}
          </DropdownMenuItem>
          <DropdownMenuItem
            v-if="isFlagship(activePiece.type)"
            @select="preview()"
          >
            {{ $t('common.view') }}
          </DropdownMenuItem>
          <DropdownMenuItem @select="onRemove(activePiece.isFresh)">
            {{ $t('piece_menu.remove') }}
          </DropdownMenuItem>
        </template>
      </OnClickOutside>
    </DropdownMenuContent>
  </DropdownMenu>
</template>

<style scoped>
.dropdown-trigger {
  width: 1px;
  height: 1px;
}
</style>
