<template>
  <header class="header">
    <button class="header__logo" @click="onBackHome">
      <svgicon name="logo" class="header__logo__icon"/>
    </button>
    <div class="separator"></div>
    <button class="btn-icon btn-layout" v-tooltip.bottom="'Layout Settings'"
            :class="{ 'active': showSettings }" @click="setShowSettings(!showSettings)">
      <svgicon name="layout"/>
    </button>
    <da-switch class="header__switch" icon="bookmark" :checked="showBookmarks"
               v-tooltip.bottom="showBookmarks ? 'Back to feed' : 'Show your bookmarks'"
               @toggle="toggleBookmarks"></da-switch>
    <div class="space"></div>
    <template v-if="showTopSites">
      <a v-for="(item, index) in topSites" :key="index" class="header__top-site"
         :href="item.url" v-tooltip.bottom="item.title" @mouseup="mouseUp('Top Sites')">
        <img :src="getIconUrl(item.url)" class="top-site__image"/>
      </a>
      <div class="separator"></div>
    </template>
    <a class="btn-icon" href="https://github.com/dailydotdev/daily" target="_blank"
       v-tooltip.bottom="'Check out our repository'">
      <svgicon name="github"/>
    </a>
    <a class="btn-icon" href="https://www.producthunt.com/posts/daily-2-0" target="_blank"
       v-tooltip.bottom="'Our Product Hunt page'">
      <svgicon name="ph"/>
    </a>
    <div class="separator"></div>
    <button class="btn-icon" v-tooltip.bottom="'Daily Go'" @click="$emit('go')">
      <svgicon name="mobile"/>
    </button>
    <button class="btn-icon btn-dnd" v-tooltip.bottom="'Do Not Disturb'"
            :class="{ 'active': showDndMenu }" @click="$emit('menu', $event)">
      <svgicon name="timer"/>
    </button>
    <button class="btn-icon btn-terminal" v-tooltip.bottom="'Latest updates'"
            :class="{ 'active': notificationsOpened }" @click="toggleNotifications">
      <svgicon name="terminal"/>
      <span class="header__badge" v-if="showNotificationBadge"></span>
    </button>
    <button class="header__profile" v-if="isLoggedIn" @click="$emit('profile')">
      <img :src="profileImage" alt="Profile image"/>
    </button>
    <button class="btn btn-water-cheese header__sign-in" v-else
            @click="$emit('login')">
      <svgicon name="user_daily"/>
      <span>Sign in</span>
    </button>
    <div class="instructions invert" v-if="topSitesInstructions">
      <div class="instructions__top-sites">
        <div class="header__top-site">
          <img :src="getIconUrl('https://www.google.com/')" class="top-site__image"/>
        </div>
        <div class="header__top-site">
          <img :src="getIconUrl('https://web.whatsapp.com/')" class="top-site__image"/>
        </div>
        <div class="header__top-site">
          <img :src="getIconUrl('https://www.youtube.com/')" class="top-site__image"/>
        </div>
        <div class="header__top-site">
          <img :src="getIconUrl('https://www.twitter.com/')" class="top-site__image"/>
        </div>
        <div class="header__top-site">
          <img :src="getIconUrl('https://www.facebook.com/')" class="top-site__image"/>
        </div>
      </div>
      <div class="instructions__desc">
        Should we show your recently visited sites?
      </div>
      <div class="instructions__buttons">
        <button class="btn btn-invert" @click="onToggleTopSites(false)">
          No
        </button>
        <button class="btn btn-invert" @click="onToggleTopSites(true)">
          Yes
        </button>
      </div>
    </div>
  </header>
</template>

<script>
import {
  mapState, mapMutations, mapGetters, mapActions,
} from 'vuex';

export default {
  name: 'DaHeader',

  components: {
    DaSwitch: () => import('@daily/components/src/components/DaSwitch.vue'),
  },

  data() {
    return {
      topSites: [],
    };
  },

  computed: {
    ...mapState('ui', [
      'showTopSites', 'showNotificationBadge', 'notificationsOpened', 'showDndMenu', 'showSettings',
    ]),
    ...mapState('feed', ['showBookmarks']),
    ...mapGetters('ui', ['topSitesInstructions']),
    ...mapGetters('user', ['isLoggedIn']),
    ...mapState({
      notificationsOpened: state => state.ui.showNotifications,
      profileImage(state) {
        if (this.isLoggedIn) {
          return state.user.profile.image;
        }

        return '';
      },
    }),
  },

  watch: {
    async showTopSites(val) {
      if (val) {
        await this.getTopSites();
      }
    },
  },

  async mounted() {
    this.loadIcons();

    if (this.showTopSites) {
      await this.getTopSites();
    }
  },

  methods: {
    loadIcons() {
      import('@daily/components/icons/logo');
      import('@daily/components/icons/layout');
      import('@daily/components/icons/bookmark');
      import('@daily/components/icons/user_daily');
      import('@daily/components/icons/terminal');
      import('@daily/components/icons/mobile');
      import('@daily/components/icons/ph');
      import('@daily/components/icons/github');
      import('@daily/components/icons/timer');
    },

    async getTopSites() {
      try {
        if ('topSites' in browser) {
          this.topSites = (await browser.topSites.get()).slice(0, 5);
        }
        return [];
      } catch {
        return [];
      }
    },

    getIconUrl(url) {
      return `https://api.daily.dev/icon?url=${encodeURIComponent(url)}&size=20`;
    },

    mouseUp(data) {
      ga('send', 'event', 'Header', 'Click', data);
    },

    toggleBookmarks(pressed) {
      this.$store.dispatch('feed/setShowBookmarks', pressed);
      ga('send', 'event', 'Header', 'Bookmarks', pressed);
    },

    toggleNotifications() {
      ga('send', 'event', 'Header', 'Terminal', !this.notificationsOpened);
      if (this.notificationsOpened) {
        this.hideNotifications();
      } else {
        this.showNotifications();
      }
    },

    onBackHome() {
      ga('send', 'event', 'Header', 'Home');
      this.backToMainFeed();
    },

    async onToggleTopSites(val) {
      ga('send', 'event', 'Instructions', 'Click', 'Top Sites');
      // TODO: handle error
      this.$store.commit('ui/nextInstruction');
      await this.$store.dispatch('ui/setShowTopSites', val);
    },

    ...mapMutations({
      hideNotifications: 'ui/hideNotifications',
      showNotifications: 'ui/showNotifications',
      setShowSettings: 'ui/setShowSettings',
    }),

    ...mapActions({
      backToMainFeed: 'feed/backToMainFeed',
    }),
  },
};
</script>

<style>
.header {
  position: absolute;
  left: 0;
  top: 0;
  display: flex;
  width: 100%;
  height: 48px;
  flex-direction: row;
  align-items: center;
  padding: 0 8px;
  background: var(--theme-background-highlight);
  border-bottom: 1px solid var(--theme-separator);
  z-index: 30;
  contain: layout size;

  & > * {
    margin: 0 4px;
  }

  & > .btn-icon {
    margin-left: 2px;
    margin-right: 2px;
  }

  & .header__logo {
    background: none;
    border: none;
    padding: 0;
    cursor: pointer;
  }

  & .header__logo__icon {
    width: 40px;
    height: 40px;
    color: var(--theme-primary);
  }

  & .separator {
    margin-left: 8px;
    margin-right: 8px;
  }

  & .header__switch {
    width: var(--da-switch-width);
    height: var(--da-switch-height);
    margin-left: 8px;

    --da-switch-checked-color: var(--color-burger-60);
    --da-switch-checked-background: var(--color-burger-90);

    &:hover {
      --da-switch-checked-color: var(--color-burger-50);
    }

    .bright & {
      --da-switch-checked-background: var(--color-burger-30);

      &:hover {
        --da-switch-checked-color: var(--color-burger-70);
      }
    }
  }

  & .header__sign-in {
    margin-left: 8px;
    margin-right: 0;
  }

  & .header__top-site {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    border: 1px solid var(--color-pepper-70);
    overflow: hidden;
  }

  & .top-site__image {
    width: 23px;
    height: 23px;
    background: var(--color-salt-10);
  }

  & .space {
    flex: 1;
  }

  & .header__profile {
    width: 30px;
    height: 30px;
    padding: 0;
    margin: 0 8px 0 14px;
    border-radius: 4px;
    overflow: hidden;
    background: none;
    border: none;
    cursor: pointer;

    & img {
      width: 100%;
      height: 100%;
    }
  }

  & .instructions {
    top: 0;
    right: 190px;
    width: 222px;
  }

  & .instructions__desc {
    text-align: center;
  }
}

.header__badge {
  position: absolute;
  left: 17px;
  bottom: 16px;
  width: 10px;
  height: 10px;
  padding: 2px;
  background: var(--theme-background-highlight);
  border-radius: 100%;

  &:before {
    content: '';
    display: block;
    width: 100%;
    height: 100%;
    background: var(--color-water-30);
    border-radius: 100%;
  }
}

.instructions__top-sites {
  display: flex;
  flex-direction: row;
  margin: 0 -4px 12px;

  & .header__top-site {
    margin: 0 4px;
    cursor: default;
  }
}

.instructions__buttons {
  display: flex;
  flex-direction: row;
  align-self: stretch;
  margin-top: 8px;

  & .btn {
    margin: 0 4px;
    flex: 1;
    justify-content: center;
  }
}

@media (min-width: 1062px) {
  .header .header__switch {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
  }
}
</style>
