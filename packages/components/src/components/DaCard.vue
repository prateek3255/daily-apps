<template>
  <div class="card" :class="cls">
    <a :href="url" target="_blank" class="card__link post__link" @click="$emit('click')">
      <div class="card__background" :style="imgStyle">
        <img
          class="card__background__image lazyload"
          :data-lowsrc="placeholder"
          :data-src="image"
          :key="image"
        />
      </div>
      <div class="card__content card__hover">
        <h5 class="card__title">
          <da-line-clamp :text="title" :lines="lines"/>
        </h5>
        <slot name="content"></slot>
      </div>
    </a>
    <div class="card__footer card__hover shadow1">
      <slot name="footer"></slot>
    </div>
    <slot name="other"></slot>
  </div>
</template>

<script>
import 'lazysizes/plugins/blur-up/ls.blur-up';
import 'lazysizes';
import DaLineClamp from './DaLineClamp.vue';

export default {
  name: 'DaCard',

  components: {
    DaLineClamp,
  },

  props: {
    size: {
      type: String,
      required: true,
    },
    url: {
      type: String,
      required: true,
    },
    placeholder: String,
    image: {
      type: String,
      required: true,
    },
    title: {
      type: String,
      required: true,
    },
    imageBackground: {
      type: String,
      default: 'none',
    },
    lines: {
      type: Number,
      default: 3,
    },
  },

  computed: {
    imgStyle() {
      return {
        background: this.imageBackground,
      };
    },

    cls() {
      return {
        [this.size]: true,
      };
    },
  },

  methods: {
    truncateTitle(text) {
      return text[0];
    },
  },
};
</script>
<style>
.card {
  display: flex;
  flex-direction: column;
  cursor: pointer;

  &.small {
    & .card__link:before {
      padding-top: 89.36%;
    }

    & .card__background {
      height: 57.14%;
    }
  }

  &.large {
    & .card__link:before {
      padding-top: 110.63%;
    }

    & .card__background {
      height: 65.38%;
    }
  }

  & .ls-blur-up-img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  & .ls-blur-up-img {
    opacity: 1;
    transition: opacity 200ms;
  }

  & .ls-blur-up-img.ls-inview.ls-original-loaded {
    opacity: 0;
  }
}

.card__link {
  position: relative;
  flex: 1;
  outline: none;

  &:before {
    content: '';
    display: block;
    padding-top: 100.71%;
  }
}

.card__background {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 62%;
  border-radius: 8px;
  overflow: hidden;
}

.card__background__image {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease-out;
  transform-origin: center;
  will-change: transform;
}

.card__content {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  min-height: 124px;
  background: var(--theme-background-highlight);
  border-radius: 8px 8px 0 0;
  z-index: 1;

  &:before {
    content: '';
    display: block;
    padding-top: 44%;
  }
}

.card__title {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  margin: 24px;
  color: var(--theme-primary);
  text-align: center;
  word-break: break-word;
}

.card__footer {
  display: flex;
  height: 48px;
  flex-direction: row;
  align-items: center;
  padding: 0 12px;
  background: var(--theme-background-highlight);
  border-radius: 0 0 8px 8px;

  & > * {
    margin: 0 4px;
  }

  & > .btn-icon {
    margin-left: 2px;
    margin-right: 2px;
  }
}

.animate-cards {
  & .card {
    transition: transform 0.4s ease-out;

    &::after {
      content: '';
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      margin: 16px 16px 0 16px;
      opacity: 0;
      border-radius: 8px;
      z-index: -1;
      box-shadow: 0 16px 24px 0 rgba(0, 0, 0, 0.64);
      transition: opacity 0.4s ease-in-out;
    }
  }

  & .card.hover,
    .card:hover {
    transform: translateY(-4px);

    &:after {
      opacity: 1;
    }
  }
}

.card:hover,
.card.hover {
  & .card__background__image {
    transform: translate(0, 4px) scale(1.05);
  }
}

.bright .animate-cards .card::after {
  box-shadow: 0 16px 24px 0 rgba(0, 0, 0, 0.24);
}
</style>
