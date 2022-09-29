<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from "vue";

export interface Props {
  text: string;

  type?: "bounce" | "slide" | "color" | "background" | "size" | "none";
  effect?: "sentence" | "word" | "letter";
  from?: string;
  to?: string;
  offset?: number;

  delay?: number;
  duration?: number;
  iteration?: string;
  direction?: string;
  timing?: string;

  run?: boolean;
  restart?: any;
}
const props = withDefaults(defineProps<Props>(), {
  type: "none",
  effect: "letter",
  from: "",
  to: "",
  offset: 0.01,

  delay: 0,
  duration: 1,
  iteration: "infinite",
  direction: "alternate",
  timing: "ease-in-out",

  run: true,
  restart: null,
});

const emit = defineEmits<{
  (e: "animation-over"): void;
}>();

const animations = ref();
const animationsEnded = ref(0);
onMounted(() => {
  animations.value.forEach((element: HTMLElement) => {
    element.addEventListener(
      "animationend",
      () => (animationsEnded.value -= 1),
      false
    );
    element.addEventListener(
      "animationstart",
      () => (animationsEnded.value += 1),
      false
    );
  });
});
onUnmounted(() => {
  animations.value.forEach((element: HTMLElement) => {
    element.removeEventListener(
      "animationend",
      () => (animationsEnded.value -= 1),
      false
    );
    element.removeEventListener(
      "animationstart",
      () => (animationsEnded.value += 1),
      false
    );
  });
});
watch(animationsEnded, (newValue, oldValue) => {
  if (animationsEnded.value == 0 && oldValue - newValue === 1)
    emit("animation-over");
});
watch(
  () => props.restart,
  () => {
    animationsEnded.value = 0;
    animations.value.forEach((element: HTMLElement) => {
      element.style.animation = "none";
      element.offsetHeight; /* trigger reflow */
      element.style.animation = "";
    });
  }
);

const animationClass = computed(() => `animate--${props.type}`);

const words = computed(() => props.text.split(" "));
const sentence = computed(() => props.text.replace(/\s/g, "&nbsp;"));
</script>

<template>
  <div class="text">
    <template v-if="props.effect === 'sentence'">
      <div
        v-html="sentence"
        :class="[{ animate: props.run }, animationClass]"
        :ref="(el) => (animations = [el])"
        aria-hidden="true"
      ></div>
    </template>

    <template v-else-if="props.effect === 'word'">
      <div
        v-for="(word, w) in words"
        :key="w"
        :style="{ 'animation-delay': `${props.delay + props.offset * w}s` }"
        :class="[{ animate: props.run }, animationClass]"
        ref="animations"
        aria-hidden="true"
      >
        {{ word }}
        <template v-if="w != words.length - 1">&nbsp;</template>
      </div>
    </template>

    <template v-else>
      <div
        v-for="(letter, l) in props.text"
        :key="l"
        :style="{ 'animation-delay': `${props.delay + props.offset * l}s` }"
        :class="[{ animate: props.run }, animationClass]"
        ref="animations"
        aria-hidden="true"
      >
        <template v-if="/\s/.test(letter)">&nbsp;</template>
        <template v-else>{{ letter }}</template>
      </div>
    </template>

    <div :aria-label="props.text" />
  </div>
</template>

<style scoped lang="scss">
.text {
  display: inline-block;

  & * {
    display: inline-block;
  }
}

.animate {
  animation-duration: v-bind("`${props.duration}s`");
  animation-iteration-count: v-bind("props.iteration");
  animation-direction: v-bind("props.direction");
  animation-timing-function: v-bind("props.timing");
  animation-delay: v-bind("`${props.delay}s`");

  &--bounce {
    animation-name: bounce;
  }
  &--slide {
    animation-name: slide;
  }
  &--color {
    animation-name: color;
  }
  &--background {
    animation-name: background;
  }
  &--size {
    animation-name: size;
  }
}

@keyframes bounce {
  from {
    transform: translateY(v-bind("props.from"));
  }
  to {
    transform: translateY(v-bind("props.to"));
  }
}

@keyframes slide {
  from {
    transform: translateX(v-bind("props.from"));
  }
  to {
    transform: translateX(v-bind("props.to"));
  }
}

@keyframes color {
  from {
    color: v-bind("props.from");
  }
  to {
    color: v-bind("props.to");
  }
}

@keyframes background {
  from {
    background-color: v-bind("props.from");
  }
  to {
    background-color: v-bind("props.to");
  }
}

@keyframes size {
  from {
    font-size: v-bind("props.from");
  }
  to {
    font-size: v-bind("props.to");
  }
}
</style>
