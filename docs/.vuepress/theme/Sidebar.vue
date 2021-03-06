<template>
  <div :class="['sidebar', isOpen ? 'sidebar--isOpen' : '']">
    <div class="sidebar__head">
      <div class="sidebar__toggle" @click="toggleMenu()">
        <span :class="['menuIcon', isOpen ? 'menuIcon--active' : '']"></span>
      </div>
      <Logo class="logo"/>
    </div>
    <div class="sidebar__body">
      <ul class="sidebar__inks" v-if="items.length">
        <li v-for="(item, i) in items" :key="i">
          <SidebarGroup
            v-if="item.type === 'group'"
            :item="item"
            :first="i === 0"
            :open="i === openGroupIndex"
            :collapsable="item.collapsable || item.collapsible"
            @toggle="toggleGroup(i)"
          />
          <SidebarLink v-else :item="item"/>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import SidebarGroup from './SidebarGroup.vue'
import SidebarLink from './SidebarLink.vue'
import Logo from './icons/logo.svg'
import { isActive } from './util'

export default {
  components: { Logo, SidebarGroup, SidebarLink },

  props: ['items'],

  data () {
    return {
      openGroupIndex: 0,
      isOpen: false,
    }
  },

  created () {
    this.refreshIndex()
  },

  watch: {
    '$route' () {
      this.refreshIndex()
    }
  },

  methods: {
    refreshIndex () {
      const index = resolveOpenGroupIndex(
        this.$route,
        this.items
      )
      if (index > -1) {
        this.openGroupIndex = index
      }
    },

    toggleGroup (index) {
      this.openGroupIndex = index === this.openGroupIndex ? -1 : index
    },

    toggleMenu () {
      this.isOpen = !this.isOpen;
    },

    isActive (page) {
      return isActive(this.$route, page.path)
    }
  }
}

function resolveOpenGroupIndex (route, items) {
  for (let i = 0; i < items.length; i++) {
    const item = items[i]
    if (item.type === 'group' && item.children.some(c => isActive(route, c.path))) {
      return i
    }
  }
  return -1
}
</script>

<style lang="scss">
@import "./styles/config.scss";

.sidebar {
  color: $text-invert;
  padding-left: 2em;

  @media (min-width: $breakpoint-medium) {
    background: $background-invert;
    overflow-y: auto;
    position: sticky;
    top: 0;
    left: 0;
    padding-left: 2em;
    height: 100vh;
    width: 280px;

    .sidebar__toggle {
      display: none;
    }
  }

  @media (min-width: $breakpoint-large) {
    width: 320px;
  }

  @media (max-width: $breakpoint-medium) {
    .sidebar__head {
      background: $background-invert;
      position: fixed;
      display: flex;
      align-items: center;
      height: $mobile-navbar-height;
      width: 100vw;
      padding: 0 1em;
      top: 0;
      left: 0;
      z-index: 10;

      &::after {
        flex: 1;
        content: '';
      }

      .sidebar__toggle {
        flex: 1;
      }
    }

    .sidebar__body {
      background: $background-invert;
      overflow-y: auto;
      position: fixed;
      width: 100vw;
      padding: 4em 1em;
      height: calc(100vh - #{$mobile-navbar-height});
      transform: translateX(-100vw);
      transition: transform 0.2s ease;
      top: $mobile-navbar-height;
      left: 0;
      z-index: 10;
    }

    &.sidebar--isOpen .sidebar__body {
      transform: translateX(0);
    }
  }

  ul {
    padding: 0;
    margin: 0;
    list-style-type: none;
  }

  a {
    // color: $text-invert;
    display: inline-block;
  }

  .logo {
    height: 48px;
    margin: 1.5em;
    margin-top: 4em;
    margin-bottom: 2.5em;

    path, ellipse {
      stroke: $text-invert;
    }

    @media (max-width: $breakpoint-medium) {
      height: 32px;
      margin: 0;

      path, ellipse {
        stroke-width: 1px;
      }
    }
  }

  .nav-links {
    display: none;
    padding: 0.5rem 0 0.75rem 0;

    a {
      font-weight: 600;
    }
    .nav-item, .repo-link {
      display: block;
      line-height: 1.25rem;
      font-size: 1.1em;
      padding: 0.5rem 0 0.5rem 1.5rem;
    }
  }
  .sidebar__links {
    // padding: 1.5rem 0;
  }
}
</style>
