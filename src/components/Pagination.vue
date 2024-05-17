<script setup lang="ts">
import { defineProps, computed } from "vue";

const props = defineProps({
  totalPages: { type: Number, required: true },
  currentPage: { type: Number, required: true },
  maxVisiblePages: { type: Number, required: true },
});

const isFirstPage = computed(() => props.currentPage === 1);
const isLastPage = computed(() => props.currentPage === props.totalPages);
const center = computed(() => Math.floor(props.maxVisiblePages / 2));

const firstVisiblePage = () => {
  if (props.currentPage <= center.value) {
    return 1;
  }

  if (
    props.currentPage >= (props.totalPages - center.value) &&
    props.currentPage <= props.totalPages
  ) {
    return props.totalPages - props.maxVisiblePages + 1;
  }

  return props.currentPage - center.value;
};

const lastVisiblePage = () => {
  return Math.min(
    firstVisiblePage() + props.maxVisiblePages - 1,
    props.totalPages
  );
};

const pages = () => {
  const range: number[] = [];

  for (let i = firstVisiblePage(); i <= lastVisiblePage(); i += 1) {
    range.push(i);
  }

  return range;
};
</script>
<template>
  <ul class="pagination">
    <li>
      <button
        type="button"
        :disabled="isFirstPage"
        @click="$emit('onClickPageChange', 'goOnFirstPage')"
      >
        <
      </button>
    </li>

    <li>
      <button
        type="button"
        :disabled="isFirstPage"
        @click="$emit('onClickPageChange', 'goPrevPage')"
      >
        <<
      </button>
    </li>

    <li v-for="page in pages()" :key="page">
      <button
        type="button"
        :disabled="currentPage === page"
        @click="$emit('onClickPageChange', '', page)"
        :class="{ 'pagination__btn--active': currentPage === page }"
      >
        {{ page }}
      </button>
    </li>

    <li>
      <button
        type="button"
        :disabled="isLastPage"
        @click="$emit('onClickPageChange', 'goNextPage')"
      >
        >>
      </button>
    </li>

    <li>
      <button
        type="button"
        :disabled="isLastPage"
        @click="$emit('onClickPageChange', 'goOnLastPage')"
      >
        >
      </button>
    </li>
  </ul>
</template>
<style lang="scss">
@import '../styles/vars';

.pagination {
  display: flex;
  justify-content: center;

  & button {
    height: 30px;
    width: 30px;
    background-color: $bg-color;
    color: $text-color-white;
    border: none;
    border-radius: 8px;
    margin-inline: 2px;
    transition: transform ease 0.3s;
  }

  & button:disabled {
    color: $text-colot-gray;
  }

  &__btn--active,
  & button:hover {
    transform: translateY(-2px);
  }
}
</style>
