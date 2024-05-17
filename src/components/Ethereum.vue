<script setup lang="ts">
import { onMounted, ref, reactive, computed } from "vue";
import axios from "axios";
import { Ethereum } from "../types/Ethereum";
import EthereumTable from "../components/EthereumTable.vue";
import Pagination from "../components/Pagination.vue";
import MetaMask from "../components/MetaMask.vue";

const BASE_URL: string = "https://api.ethrewards.io/entities";

type AprSortType = "monthly" | "annual" | "all";
interface SortValues {
  sort_by: keyof Ethereum | "";
  order: "asc" | "desc";
}

const ethereumFromAPI = ref<Ethereum[]>([]);
const ethereumPriceFromAPI = ref<number>(0);
const aprSortValue = ref<AprSortType>("monthly");
const sortValues: SortValues = reactive({ sort_by: "", order: "asc" });
const maxVisiblePages: number = 10;
const currentPage = ref<number>(1);
let totalItems = ref(0);
const totalPages = computed(() => Math.ceil(totalItems.value / 20));
const isLoading = ref<boolean>(true);

const getEthereumFromAPI = async (page: number = 1) => {
  await axios
    .get(`${BASE_URL}/stats?page=${page}&limit=20`)
    .then(({ data }) => {
      ethereumFromAPI.value = data.items;
      totalItems.value = data.count;
    })
    .catch((error) => console.log(error))
    .finally(() => (isLoading.value = false));
};

const getEthereumPriceFromAPI = async () => {
  const price = await axios
    .get("https://api.ethrewards.io/historical/info")
    .then(({ data }) => data.eth_price)
    .catch((error) => console.log(error));

  ethereumPriceFromAPI.value = +price.current;
};
onMounted(async () => {
  await getEthereumFromAPI();
  await getEthereumPriceFromAPI();
});

const columns = ref([
  { label: "Name", field: "name", sortable: true },
  { label: "Staked", field: "staked", sortable: true },
  { label: "APR", field: "apr", sortable: true },
  { label: "Execution Rewards", field: "executed_rewards", sortable: true },
  { label: "Consensus Rewards", field: "consensus_rewards", sortable: true },
  { label: "Produced Blocks", field: "produced_blocks", sortable: true },
]);

const convertAPR = (val: string) => {
  const sliceAPR = +val * 100;

  return sliceAPR.toFixed(1);
};

const convertName = (val: string) => {
  const splitName = val.split(".");

  return splitName[0];
};

const formatRewards = (val: string) => {
  return Number(val).toFixed(2);
};

const formatStaked = (val: string) => {
  let str = val;
  let result = "";

  if (val.length <= 3) {
    return val;
  }

  while (str.length > 0) {
    if (str.length >= 3) {
      result = " " + str.slice(-3) + result;
      str = str.slice(0, -3);
    } else {
      result = str + result;
      str = "";
    }
  }

  return result;
};

const convertEthInUsd = (value: string) => {
  return formatStaked(
    String((ethereumPriceFromAPI.value * Number(value)).toFixed(0))
  );
};

const toggleAprSortBtn = (val: AprSortType) => {
  if (aprSortValue.value === val) {
    return;
  }

  aprSortValue.value = val;
};

const convertAprValue = (val: Ethereum) => {
  let aprStr = "";

  switch (aprSortValue.value) {
    case "monthly":
      aprStr = val.apr_30days;
      break;
    case "annual":
      aprStr = val.apr_365days;
      break;
    default:
      aprStr = val.apr;
  }

  return (Number(aprStr) * 100).toFixed(1);
};

const getSortedEthereum = async (sortName: keyof Ethereum) => {
  isLoading.value = true;

  const order =
    sortValues.sort_by === sortName
      ? sortValues.order === "asc"
        ? "desc"
        : "asc"
      : "asc";

  sortValues.sort_by = sortName;
  sortValues.order = order;

  try {
    const response = await axios.get(
      `${BASE_URL}/stats?page=${currentPage.value}&amp;limit=20&amp;sort_by=${sortName}&amp;order=${order}`
    );
    ethereumFromAPI.value = response.data.items;
  } catch (error) {
    console.log(error);
  } finally {
    isLoading.value = false;
  }
};

type ActionType =
  | "goNextPage"
  | "goPrevPage"
  | "goOnFirstPage"
  | "goOnLastPage";

const onClickPageChange = (action: ActionType, page: number): void => {
  switch (action) {
    case "goNextPage":
      currentPage.value += 1;
      getEthereumFromAPI(currentPage.value);
      break;

    case "goPrevPage":
      currentPage.value -= 1;
      getEthereumFromAPI(currentPage.value);
      break;

    case "goOnFirstPage":
      currentPage.value = 1;
      getEthereumFromAPI(currentPage.value);
      break;

    case "goOnLastPage":
      currentPage.value = totalPages.value;
      getEthereumFromAPI(currentPage.value);
      break;

    default:
      currentPage.value = page;
      getEthereumFromAPI(page);
  }
};
</script>
<template>
  <section class="ethereum">
    <MetaMask />
    <h1 class="ethereum__title">
      Stay on top of crypto.<br />Task for front end.
    </h1>
    <div class="ethereum__validator">
      <p>Ethereum validator rating</p>
      <div class="ethereum__rating-btns">
        <button
          class="ethereum__rating-btn"
          :class="{ 'ethereum__rating-btn--active': aprSortValue === 'monthly' }"
          @click="toggleAprSortBtn('monthly')"
        >
          monthly
        </button>
        <button
          class="ethereum__rating-btn"
          :class="{ 'ethereum__rating-btn--active': aprSortValue === 'annual' }"
          @click="toggleAprSortBtn('annual')"
        >
          annual
        </button>
        <button
          class="ethereum__rating-btn"
          :class="{ 'ethereum__rating-btn--active': aprSortValue === 'all' }"
          @click="toggleAprSortBtn('all')"
        >
          all
        </button>
      </div>
    </div>
    <article>
      <template v-if="isLoading">
        <div class="info-msg">Data is loading...</div>
      </template>
      <template v-else-if="!ethereumFromAPI.length">
        <div class="info-msg">No data received from server.</div>
      </template>
      <template v-else>
        <EthereumTable
          :columns="columns"
          :rows="ethereumFromAPI"
          :sortValues="sortValues"
          @getSortedEthereum="getSortedEthereum"
        >
          <template #name="{ item }">
            <div>{{ convertName(item.name) }}</div>
          </template>
          <template #staked="{ item }">
            <div>{{ formatStaked(item.staked) }} ETH</div>
            <div class="eth-usd">$ {{ convertEthInUsd(item.staked) }}</div>
          </template>
          <template #apr="{ item }">
            <div>{{ convertAprValue(item) }}%</div>
          </template>
          <template #executed_rewards="{ item }">
            <div>{{ convertAPR(item.executed_rewards) }} ETH</div>
          </template>
          <template #consensus_rewards="{ item }">
            <div>{{ formatRewards(item.consensus_rewards) }} ETH</div>
          </template>
        </EthereumTable>
        <Pagination
          :totalPages="totalPages"
          :currentPage="currentPage"
          :maxVisiblePages="maxVisiblePages"
          @onClickPageChange="onClickPageChange"
        />
      </template>
    </article>
  </section>
</template>
<style lang="scss">
@import '../styles/vars';

h1 {
  font-weight: 500;
  font-size: 36px;
  line-height: 43px;
}

.ethereum {
  padding-inline: 33px;
  padding-bottom: 40px;
  max-width: 1440px;
  margin-inline: auto;

  &__validator {
    margin-top: 41px;
    display: flex;
    justify-content: space-between;
    height: 48px;
    background-color: $bg-color;
    align-items: center;
    padding-inline: 16px;
    border-radius: 8px;

    & p {
      font-size: 14px;
      color: $text-colot-gray;
    }
  }

  &__rating-btns {
    display: flex;
    border-radius: 50px;
    background-color: #18181b;
    padding-block: 3.5px;
    padding-inline: 4px;
  }

  &__rating-btn {
    font-size: 8px;
    text-transform: uppercase;
    color: #494949;
    border: none;
    border-radius: 40px;
    line-height: 10.42px;
    background-color: #18181b;
    padding-block: 3px;

    transition: color ease 0.3s;
  }

  &__rating-btn:hover {
    color: $text-color-white;
  }

  &__rating-btn--active {
    background-color: $bg-color;
    color: $text-color-white;
  }

  & article {
    margin-top: 16px;
    padding-left: 8px;
    padding-right: 9px;
  }
}

.eth-usd {
  color: #575757;
}

.info-msg {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  font-size: 16px;
}
</style>
