<template>
  <v-app>
    <v-row>
      <matic-bridge></matic-bridge>
      <any-swap-bridge></any-swap-bridge>
      <binance-bridge></binance-bridge>
    </v-row>
  </v-app>
</template>

<script>
import MaticBridge from "@/components/MaticBridge";
import BinanceBridge from "@/components/BinanceBridge";
import AnySwapBridge from "@/components/AnySwapBridge";

export default {
  name: 'App',

  components: {
    AnySwapBridge,
    MaticBridge,
    BinanceBridge,
  },

  data: () => ({
    //
  }),
};
</script>
