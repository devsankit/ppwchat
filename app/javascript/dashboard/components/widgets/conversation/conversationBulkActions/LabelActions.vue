<script>
import { mapGetters } from 'vuex';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: {
    NextButton,
  },
  emits: ['update', 'close', 'assign'],
  data() {
    return {
      query: '',
      selectedLabels: [],
    };
  },
  computed: {
    ...mapGetters({ labels: 'labels/getLabels' }),
    filteredLabels() {
      return this.labels.filter(label =>
        label.title.toLowerCase().includes(this.query.toLowerCase())
      );
    },
  },
  methods: {
    isLabelSelected(label) {
      return this.selectedLabels.includes(label);
    },
    assignLabels(key) {
      this.$emit('update', key);
    },
    onClose() {
      this.$emit('close');
    },
  },
};
</script>

<template>
  <div v-on-clickaway="onClose" class="labels-container">
    <div class="triangle">
      <svg height="12" viewBox="0 0 24 12" width="24">
        <path d="M20 12l-8-8-12 12" fill-rule="evenodd" stroke-width="1px" />
      </svg>
    </div>
    <div class="flex items-center justify-between header">
      <span>{{ $t('BULK_ACTION.LABELS.ASSIGN_LABELS') }}</span>
      <NextButton ghost xs slate icon="i-lucide-x" @click="onClose" />
    </div>
    <div class="labels-list">
      <header class="labels-list__header">
        <div
          class="flex items-center justify-between h-8 gap-2 label-list-search"
        >
          <fluent-icon icon="search" class="search-icon" size="16" />
          <input
            v-model="query"
            type="search"
            :placeholder="$t('BULK_ACTION.SEARCH_INPUT_PLACEHOLDER')"
            class="reset-base !outline-0 !text-sm label--search_input"
          />
        </div>
      </header>
      <ul class="labels-list__body">
        <li
          v-for="label in filteredLabels"
          :key="label.id"
          class="label__list-item"
        >
          <label
            class="item"
            :class="{ 'label-selected': isLabelSelected(label.title) }"
          >
            <input
              v-model="selectedLabels"
              type="checkbox"
              :value="label.title"
              class="label-checkbox"
            />
            <span
              class="overflow-hidden label-title whitespace-nowrap text-ellipsis"
            >
              {{ label.title }}
            </span>
            <span
              class="label-pill"
              :style="{ backgroundColor: label.color }"
            />
          </label>
        </li>
      </ul>
      <footer class="labels-list__footer">
        <NextButton
          sm
          type="submit"
          :label="$t('BULK_ACTION.LABELS.ASSIGN_SELECTED_LABELS')"
          :disabled="!selectedLabels.length"
          @click="$emit('assign', selectedLabels)"
        />
      </footer>
    </div>
  </div>
</template>

<style scoped lang="scss">
.labels-list {
  @apply flex flex-col max-h-[15rem] min-h-[auto];

  .labels-list__header {
    @apply bg-n-alpha-3 backdrop-blur-[100px] py-0 px-2.5;
  }

  .labels-list__body {
    @apply flex-1 overflow-y-auto py-2.5 mx-0;
  }

  .labels-list__footer {
    @apply p-2;

    button {
      @apply w-full;

      .button__content {
        @apply text-center;
      }
    }
  }
}

.label-list-search {
  @apply bg-n-alpha-black2 py-0 px-2.5 border border-solid border-n-strong rounded-md;

  .search-icon {
    @apply text-n-slate-10;
  }

  .label--search_input {
    @apply border-0 text-xs m-0 dark:bg-transparent bg-transparent h-[unset] w-full;
  }
}

.labels-container {
  @apply absolute ltr:right-2 rtl:left-2 top-12 origin-top-right w-auto z-20 max-w-[15rem] min-w-[15rem] bg-n-alpha-3 backdrop-blur-[100px] border-n-weak rounded-lg border border-solid shadow-md;

  .header {
    @apply p-2.5;

    span {
      @apply text-sm font-medium;
    }
  }

  .container {
    @apply max-h-[15rem] overflow-y-auto;

    .label__list-container {
      @apply h-full;
    }
  }

  .triangle {
    @apply block z-10 absolute text-left -top-3 ltr:right-[--triangle-position] rtl:left-[--triangle-position];

    svg path {
      @apply fill-n-alpha-3 backdrop-blur-[100px]  stroke-n-weak;
    }
  }
}

ul {
  @apply m-0 list-none;
}

.labels-placeholder {
  @apply p-2;
}

.label__list-item {
  @apply my-1 mx-0 py-0 px-2.5;

  .item {
    @apply items-center rounded-md cursor-pointer flex py-1 px-2.5 hover:bg-n-slate-3 dark:hover:bg-n-solid-3;

    &.label-selected {
      @apply bg-n-slate-2;
    }

    span {
      @apply text-sm;
    }

    .label-checkbox {
      @apply my-0 ltr:mr-2.5 rtl:ml-2.5;
    }

    .label-title {
      @apply flex-grow w-full;
    }

    .label-pill {
      @apply rounded-md h-3 w-3 flex-shrink-0 border border-solid border-n-weak;
    }
  }
}

.search-container {
  @apply bg-n-alpha-3 backdrop-blur-[100px] py-0 px-2.5 sticky top-0 z-20;
}
</style>
