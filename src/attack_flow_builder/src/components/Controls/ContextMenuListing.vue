<template>
  <div class="context-menu-listing-control" :style="offset" @contextmenu.prevent="">
    <!-- Menu Sections -->
    <div class="section" v-for="(section, i) of sections" :key="section.id">
      <!-- Menu Section -->
      <template v-for="item of section.items" :key="item.text">
        <!-- Submenu Item -->
        <li 
          v-if="item.type === MenuType.Submenu"
          :class="{ disabled: item.disabled }"
          @mouseenter="submenuEnter(item)"
          @mouseleave="submenuLeave"
        >
          <a class="item">
            <span class="text">{{ item.text }}</span>
            <span class="more-arrow"></span>
          </a>
          <div class="submenu" v-if="item.text === focusedSubMenu">
            <ContextMenuListing :sections="item.sections" @select="onChildItemSelect"/>
          </div>
        </li>
        <!-- Regular Item -->
        <li v-else :class="{ disabled: item.disabled }" @click="onItemClick(item)">
          <a class="item" :href="item.disabled ? null : item.link" target="_blank">
            <span class="check" v-show="item.value">✓</span>
            <span class="text">{{ item.text }}</span>
            <span v-if="item.shortcut" class="shortcut">
                {{ formatShortcut(item.shortcut) }}
            </span>
          </a>
        </li>
      </template>
      <!-- Section Divider -->
      <a v-if="i < sections.length - 1" class="section-divider"></a>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';
import { ContextMenu, ContextMenuItem, ContextMenuSection, MenuType } from "@/assets/scripts/ContextMenuTypes";

const KeyToText: { [key: string]: string } = {
  Control    : "Ctrl",
  Escape     : "Esc",
  ArrowLeft  : "←",
  ArrowUp    : "↑",
  ArrowRight : "→",
  ArrowDown  : "↓",
  Delete     : "Del"
}

export default defineComponent({
  name: 'ContextMenuListing',
  props: {
    sections: {
      type: Array as PropType<ContextMenuSection[]>,
      required: true
    },
    forceInsideWindow: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      xOffset: 0,
      yOffset: 0,
      leaveTimeout: 500,
      leaveTimeoutId: 0,
      focusedSubMenu: null as string | null,
      MenuType
    }
  },
  computed: {

    /**
     * Returns the ContextMenuListing's offset styling.
     * @returns
     *  The ContextMenuListing's offset styling.
     */
    offset(): { marginTop: string, marginLeft: string } {
      return {
        marginTop: `${ this.yOffset }px`,
        marginLeft: `${ this.xOffset }px`
      }
    }

  },
  methods: {
    
    /**
     * Submenu mouse enter behavior.
     * @param item
     *  The hovered submenu item.
     */
    submenuEnter(item: ContextMenu) {
      clearTimeout(this.leaveTimeoutId);
      if(!item.disabled) {
        this.focusedSubMenu = item.text;
      }
    },

    /**
     * Submenu mouse leave behavior.
     */
    submenuLeave() {
      this.leaveTimeoutId = setTimeout(() => {
        this.focusedSubMenu = null;
      }, this.leaveTimeout)
    },

    /**
     * Menu item selection behavior.
     * @param item
     *  The selected menu item.
     */
    onItemClick(item: ContextMenuItem) {
      if(item.disabled)
        return;
      this.$emit("select", item.data, !item.keepMenuOpenOnSelect);
    },

    /**
     * Submenu item selection behavior.
     * @param data
     *  The menu item's data.
     * @param closeSubmenu
     *  If the active submenu should close.
     */
    onChildItemSelect(data: any, closeSubmenu: boolean) {
      this.$emit("select", data, closeSubmenu);
      if(closeSubmenu) {
        this.focusedSubMenu = null;
      }
    },

    /**
     * Formats a keyboard shortcut.
     * @param shortcut
     *  The keyboard shortcut to format.
     * @returns
     *  The formatted keyboard shortcut.
     */
    formatShortcut(shortcut?: string): string | undefined {
      if(!shortcut) {
        return shortcut;
      } else {
        return shortcut
          .split("+")
          .map(c => c in KeyToText ? KeyToText[c] : c)
          .join("+");
      }
    }

  },
  emits: ["select"],
  mounted() {
    if(!this.forceInsideWindow) return;
    // Offset submenu if outside of viewport
    let viewWidth  = window.innerWidth;
    let viewHeight = window.innerHeight;
    let { top, left, bottom, right } = this.$el.getBoundingClientRect();
    this.xOffset = right > viewWidth ? -Math.min(left, right - viewWidth) : 0;
    this.yOffset = bottom > viewHeight ? -Math.min(top, bottom - viewHeight) : 0;
  }
});
</script>

<style scoped>

/** === Main Control === */

.context-menu-listing-control {
  display: flex;
  flex-direction: column;
  width: max-content;
  min-width: 130px;
  color: #d1d1d1;
  font-size: 10pt;
  padding: 6px 4px;
  border: solid 1px #242424;
  border-radius: 3px;
  box-sizing: border-box;
  background: #383838;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.22);
}

/** === Section Divider === */

.section .section-divider {
  padding: 0px;
  border-bottom: solid 1px #545454;
  margin: 4px 4px;
  cursor: default;
}

/** === Submenu === */

.submenu {
  position: absolute;
  top: -4px;
  left: 100%;
  padding-left: 6px;
  z-index: 1;
}

/** === Menu Item === */

li {
  position: relative;
  list-style: none;
  user-select: none;
}
li:not(.disabled):hover {
  color: #fff;
  background: #726de2;
}

a {
  display: flex;
  padding: 4px 0px;
  cursor: pointer;
}
li.disabled a {
  color: #8f8f8f;
  cursor: unset;
}

.text, 
.shortcut,
.more-arrow {
  display: flex;
  align-items: center;
  padding: 0px 23px;
}
.text {
  flex: 1 1 auto;
}
.shortcut {
  flex: 2 1 auto;
  justify-content: right;
}
.check {
  position: absolute;
  left: 5px;
}
.more-arrow::before {
  content: "";
  display: block;
  width: 6px;
  height: 6px;
  border-top: solid 1px;
  border-right: solid 1px;
  transform: rotate(45deg);
}

</style>
