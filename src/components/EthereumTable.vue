<script setup lang="ts">
import { PropType } from "vue";
import { Ethereum } from "../types/Ethereum";

type TableColumn = {
  label: string;
  field: string;
  sortable?: boolean;
  width?: string;
};

const props = defineProps({
  columns: { type: Array as PropType<Array<TableColumn>>, required: true },
  rows: { type: Array as PropType<Array<Ethereum>>, required: true },
  sortValues: { type: Object, required: true },
});

const getColValue = (value: any) => {
  if (value === null || value === undefined) {
    return "";
  }

  return value;
};

const isArrowDown = (field: string) => {
  return props.sortValues.sort_by === field && props.sortValues.order === 'desc';
};

</script>
<template>
  <table>
    <thead>
      <tr>
        <th v-for="(column, index) in columns" :key="index" class="table-underline">
          <div>
            <img src="../images/↑.svg" alt="Arrow" :class="{'desc-arrow': isArrowDown(column.field)}">
            <div @click="$emit('getSortedEthereum', column.field)">{{ column.label }}</div>
          </div>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, index) in rows" :key="index">
        <td v-for="(column, index) in columns" :key="index" class="table-underline">
          <slot v-if="$slots[column.field]" :name="column.field" :item="row" />

          <span v-else>{{ getColValue(row[column.field as keyof Ethereum]) }}%</span>
        </td>
      </tr>
    </tbody>
  </table>
</template>
<style lang="scss">
@import '../styles/vars';

table {
  border-collapse: collapse;
  width: 100%;
  margin-bottom: 30px;

  font-size: 14px;
}

thead th {
  padding-right: 40px;
  padding-bottom: 8px;
  color: $text-colot-gray;
  cursor: pointer;
  transition: color ease .3s;
  min-width: 150px;
}

thead th:first-child {
  width: 170px;
}

thead th img {
  margin-right: 5px;
  transition: transform ease .3s;
}

.desc-arrow {
  transform: rotate(180deg);
}

thead th div {
  text-transform: uppercase;
}

thead th div:first-child {
  display: flex;
}

thead th:hover {
  color: $text-color-white;
}

tbody td {
  height: 60px;
  padding-right: 40px;
}

tbody tr {
  transition: background-color ease .3s;
}

tbody tr:hover {
  background-color: $bg-color-dark-gray;
}

thead th:first-child,
tbody td:first-child {
  padding-left: 10px;
}

tbody td:first-child {
  text-transform: capitalize;
}

tbody td:nth-child(3) {
  color: #00FF47;
}

.table-underline {
  border-bottom: 1px solid $bg-color-dark-gray;
}
</style>
