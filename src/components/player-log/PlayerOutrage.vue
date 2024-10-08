<script lang="ts" setup>
import { ComboboxAnchor, ComboboxInput, ComboboxPortal, ComboboxRoot } from 'radix-vue'
import {
  TagsInput,
  TagsInputInput,
  TagsInputItem,
  TagsInputItemDelete
} from '@/components/ui/tags-input'
import { Label } from '@/components/ui/label'
import { CommandEmpty, CommandGroup, CommandItem, CommandList } from '@/components/ui/command'
import { ref, computed } from 'vue'
import { Resource } from '@/Archive'
import { vOnClickOutside } from '@vueuse/components'

import type { Player } from '@/Archive'
import { isResource } from '@/lib/utils'

const props = defineProps<{
  player: Player
}>()

const emit = defineEmits<{
  update: [Resource]
}>()

const outrageOpen = ref(false)
const filteredResources = computed(() =>
  Object.values(Resource).filter((r) => !props.player.outrage?.includes(r))
)
</script>

<template>
  <div>
    <Label for="player-outrage">{{ $t('player_area.outrage') }}</Label>
    <ComboboxRoot
      v-model:open="outrageOpen"
      :value="player.outrage"
      multiple
      class="flex flex-grow"
    >
      <ComboboxAnchor as-child>
        <TagsInput
          id="player-outrage"
          class="w-full gap-0 px-3"
          :model-value="player.outrage"
        >
          <div class="flex flex-wrap items-center gap-2">
            <TagsInputItem
              v-for="resource in player.outrage"
              :key="resource"
              :value="resource"
            >
              <img
                :src="`./images/${resource.toLowerCase()}.png`"
                :alt="resource"
                class="h-6"
              />
              <span class="ml-2 mr-1">{{ $t(`resources.${resource}`) }}</span>
              <TagsInputItemDelete />
            </TagsInputItem>
          </div>

          <ComboboxInput
            @click="outrageOpen = true"
            as-child
          >
            <TagsInputInput
              class="w-full px-0"
              readonly
              @keydown.enter.prevent
            />
          </ComboboxInput>
        </TagsInput>
      </ComboboxAnchor>
      <ComboboxPortal disabled>
        <CommandList
          avoid-collisions
          position="popper"
          :collision-padding="{ bottom: 180 }"
          class="w-[--radix-popper-anchor-width] rounded-md mt-2 border bg-popover text-popover-foreground shadow-md outline-none data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2"
          v-on-click-outside="
            () => {
              outrageOpen = false
            }
          "
        >
          <CommandEmpty />
          <CommandGroup>
            <CommandItem
              v-for="resource in filteredResources"
              :key="resource"
              :value="resource"
              @select.prevent="
                (e) => {
                  if (e.detail.value && isResource(e.detail.value)) {
                    emit('update', e.detail.value)
                  }
                }
              "
            >
              <img
                :src="`./images/${resource.toLowerCase()}.png`"
                :alt="resource"
                class="h-6 mr-2"
              />
              {{ $t(`resources.${resource}`) }}
            </CommandItem>
          </CommandGroup>
        </CommandList>
      </ComboboxPortal>
    </ComboboxRoot>
  </div>
</template>
