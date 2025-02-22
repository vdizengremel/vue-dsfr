<script lang="ts" setup>
import { ref, watch } from 'vue'

import { useCollapsable } from '../../composables'
import { getRandomId } from '../../utils/random-utils'

import DsfrSideMenuList from './DsfrSideMenuList.vue'
import type { DsfrSideMenuProps } from './DsfrSideMenu.types'

export type { DsfrSideMenuProps }

withDefaults(defineProps<DsfrSideMenuProps>(), {
  buttonLabel: 'Dans cette rubrique',
  id: () => getRandomId('sidemenu'),
  sideMenuListId: () => getRandomId('sidemenu', 'list'),
  collapseValue: '-492px',
  // @ts-expect-error this is really undefined
  menuItems: () => undefined,
  headingTitle: '',
})

defineEmits<{ (e: 'toggleExpand', payload: string): void }>()

const {
  collapse,
  collapsing,
  cssExpanded,
  doExpand,
  onTransitionEnd,
} = useCollapsable()

const expanded = ref(false)

/*
 * @see https://github.com/GouvernementFR/dsfr/blob/main/src/core/script/collapse/collapse.js
 */
watch(expanded, (newValue, oldValue) => {
  if (newValue !== oldValue) {
    doExpand(newValue)
  }
})
</script>

<template>
  <nav
    class="fr-sidemenu"
    :aria-labelledby="id"
  >
    <div class="fr-sidemenu__inner">
      <button
        class="fr-sidemenu__btn"
        :aria-controls="id"
        :aria-expanded="expanded"
        @click.prevent.stop="expanded = !expanded"
      >
        {{ buttonLabel }}
      </button>
      <div
        :id="id"
        ref="collapse"
        class="fr-collapse"
        :class="{
          'fr-collapse--expanded': cssExpanded, // Need to use a separate data to add/remove the class after a RAF
          'fr-collapsing': collapsing,
        }"
        @transitionend="onTransitionEnd(expanded)"
      >
        <div class="fr-sidemenu__title">
          {{ headingTitle }}
        </div>
        <!-- @slot Slot par défaut du contenu du menu latéral -->
        <slot>
          <DsfrSideMenuList
            :id="sideMenuListId"
            :menu-items="menuItems"
            @toggle-expand="$emit('toggleExpand', $event)"
          />
        </slot>
      </div>
    </div>
  </nav>
</template>
