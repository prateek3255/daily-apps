<template>
  <div class="app">
    <router-view></router-view>
  </div>
</template>

<script>
import 'focus-visible';
import { mapGetters } from 'vuex';
import initializeAnalytics from '../common/analytics';
import { getCache, ANALYTICS_CONSENT_KEY } from '../common/cache';
import { browserName } from '../common/browser';

export default {
  computed: {
    ...mapGetters({ isLoggedIn: 'user/isLoggedIn' }),
  },

  methods: {
    initializeAnalytics(consent) {
      initializeAnalytics(consent, this.isLoggedIn ? this.$store.state.user.profile.id : null);
    },

    startTracking() {
      if (browserName === 'firefox') {
        getCache(ANALYTICS_CONSENT_KEY, null)
          .then((consent) => {
            if (consent !== null) {
              this.initializeAnalytics(consent);
            }
          })
          // TODO: handle error
          // eslint-disable-next-line no-console
          .catch(console.error);
      } else {
        this.initializeAnalytics(true);
      }
    },
  },

  mounted() {
    this.startTracking();
  },
};
</script>

<style>
@import '../../node_modules/@daily/components/src/styles/global.pcss';

html.loaded {
  background: var(--theme-background-primary);
}

body {
  margin: 0;
  overflow-y: scroll;
  min-width: 705px;
}

html, body {
  height: 100%;
}

a {
  text-decoration: none;
}

.separator {
  display: block;
  border-left: 1px dotted var(--theme-separator);
  height: 32px;
  width: 1px;
}

.app {
  height: 100%;
}

.page {
  position: relative;
  display: flex;
  width: 100%;
  min-height: 100%;
  flex-direction: column;
  justify-content: stretch;
  color: var(--theme-primary);
}

.modal .modal__container {
  display: flex;
  width: 427px;
  flex-direction: column;
  align-items: center;
  padding: 40px 0;
  color: var(--color-salt-10);
  overflow: hidden;

  & h1 {
    margin: 16px 0;
    text-align: center;
    text-transform: uppercase;

    &.overlap {
      margin-top: -32px;
    }

    & a {
      color: var(--color-salt-10);
      outline: none;
    }
  }

  & p {
    margin: 0 48px;
    text-align: center;

    @mixin jr;
  }

  & .btn.btn-modal {
    width: 180px;
    margin-top: 32px;
    justify-content: center;
  }
}

.modal.full .modal__container {
  padding-top: 0;
}

.instructions {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 16px;
  border-radius: 4px;
  background: var(--theme-background-highlight);
  box-shadow: 0 8px 16px 4px rgba(0, 0, 0, 0.32);
  z-index: 10;

  & .btn {
    height: 24px;

    @mixin nuggets;
  }
}

.instructions__desc {
  color: var(--theme-primary);

  & {
    @mixin nuggets;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.15s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
