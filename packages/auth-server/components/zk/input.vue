<template>
  <div class="w-full">
    <div class="flex w-full">
      <div class="relative grow items-center flex">
        <input
          v-model="model"
          :placeholder
          type="text"
          :class="twMerge(inputUI, ui.input, stateUI, loadingInputUI)"
          :disabled
          :required
        >
        <transition
          v-bind="TransitionOpacity"
          mode="out-in"
        >
          <CommonSpinner
            v-if="loading"
            class="-my-1.5 w-auto h-[2em] absolute z-10 right-3"
          />
        </transition>
      </div>
      <div
        v-if="postLabel"
        :class="twMerge(postLabelUI, ui.postLabel)"
      >
        {{ postLabel }}
      </div>
    </div>
    <div class="block pl-5">
      <span
        v-for="(message, index) in messages"
        :key="index"
        :class="
          twMerge('block text-xs text-red-600 dark:text-red-400 mt-1', ui.messages)
        "
      >
        {{ message }}
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { twMerge } from "tailwind-merge";

const model = defineModel({ default: "", type: String });

type UI = {
  input?: string;
  postLabel?: string;
  messages?: string;
};

const {
  placeholder = "",
  postLabel,
  // eslint-disable-next-line vue/require-valid-default-prop
  ui = {},
  error = false,
  messages,
  disabled = false,
  loading = false,
} = defineProps<{
  placeholder?: string;
  postLabel?: string;
  ui?: UI;
  error?: boolean;
  messages?: string[];
  disabled?: boolean;
  required?: boolean;
  loading?: boolean;
}>();

const baseInputUI
  = "px-4 py-3 rounded-zk border border-neutral-300 inline-block";

const inputUI = computed(() => {
  let baseClasses = twMerge(
    baseInputUI,
    "focus:ring-primary-400 focus:outline-none focus:border-neutral-700 z-10 relative flex-auto disabled:border-neutral-200 dark:bg-neutral-900 dark:border-neutral-700 dark:disabled:border-neutral-800 dark:text-neutral-100",
  );

  if (postLabel) {
    baseClasses = twMerge(baseClasses, "rounded-r-none z-10");
  }

  return baseClasses;
});

const postLabelUI = twMerge(
  baseInputUI,
  "text-neutral-600 bg-neutral-100 rounded-l-none rounded-r-zk border-l-none z-0 relative dark:bg-neutral-950 dark:border-neutral-700 dark:text-neutral-400",
);

const stateUI = computed(() => {
  if (error) {
    return "border-red-400 focus:border-red-400 dark:border-red-400 dark:focus:border-red-400/80";
  }

  return "";
});

const loadingInputUI = computed(() => {
  if (loading) {
    return "pr-12";
  }
  return "";
});
</script>
