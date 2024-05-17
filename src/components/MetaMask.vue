<script setup lang="ts">
import { reactive } from "vue";

interface WalletType {
  address: string;
  text_signature: string;
  formated_address: string;
  formated_signature: string;
}

const wallet = reactive<WalletType>({
  address: "",
  text_signature: "",
  formated_address: "",
  formated_signature: "",
});

const formatAddress = (address: string) => {
  const start = address.slice(0, 6);
  const end = address.slice(-4);

  return start + "..." + end;
};

const getWalletAddress = async () => {
  const response = await (window as any).ethereum.request({
    method: "eth_requestAccounts",
    params: [],
  });
  wallet.address = response[0];
  wallet.formated_address = formatAddress(response[0]);
};

const getTextSignature = async () => {
  const response = await (window as any).ethereum.request({
    method: "personal_sign",
    params: [
      "0x506c65617365207369676e2074686973206d65737361676520746f20636f6e6669726d20796f7572206964656e746974792e",
      wallet.address,
    ],
  });

  wallet.text_signature = response;
  wallet.formated_signature = formatAddress(response);
};
</script>
<template>
  <ul class="metamask-btns">
    <li>
      <button class="metamask-btn" :class="{'metamask-btn--disabled': !wallet.formated_address}" @click="getTextSignature">
        {{
          !wallet.formated_signature
            ? "Sign Message"
            : wallet.formated_signature
        }}
      </button>
    </li>
    <li>
      <button class="metamask-btn" @click="getWalletAddress">
        {{
          !wallet.formated_address ? "Connect Wallet" : wallet.formated_address
        }}
      </button>
    </li>
  </ul>
</template>
<style lang="scss">
@import '../styles/vars';

.metamask-btns {
  display: flex;
  justify-content: flex-end;
  padding: 24px 40px 21px 0;
  gap: 25px;
}

.metamask-btn {
  width: 145px;
  height: 36px;
  border: none;
  border-radius: 5px;
  background-color: $bg-color;

  font-family: "DM Mono", monospace;
  font-weight: 400;
  color: $text-colot-gray;
  font-size: 14px;
  line-height: 18.23px;

  transition: color ease .3s;
}

.metamask-btn:hover {
  color: $text-color-white;
}

.metamask-btn--disabled {
  display: none;
}
</style>
