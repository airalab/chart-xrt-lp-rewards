<template>
  <div>
    <Chart ref="chart" :log="points" :staked="stakedPoint" />
    <p>
      staked
      <input v-model="staked" /> XRT
    </p>
    <p>
      total
      <input v-model="total" /> XRT
    </p>
    <button @click="draw">draw</button>
  </div>
</template>

<script>
import axios from "axios";
import Chart from "../components/Chart.vue";
import init, { total_payout } from "../rewards/xrt_lp_rewards.js";

export default {
  components: {
    Chart,
  },
  data() {
    return {
      points: [],
      stakedPoint: [],
      staked: "0",
      total: "182559",
    };
  },
  async mounted() {
    await init("xrt_lp_rewards_bg.wasm");
    await this.loadStaked();
    this.draw();
  },
  methods: {
    async loadStaked() {
      const result = await axios.get(
        "https://api.etherscan.io/api?module=account&action=tokenbalance&contractaddress=0x7de91b204c1c737bcee6f000aaa6569cf7061cb7&address=0x3185626c14acb9531d19560decb9d3e5e80681b1&tag=latest&apikey=U6AM52QNQZRTE6P4J8RUH1U5JB3I3SUWIZ"
      );
      this.staked = Math.round(
        Number(result.data.result) / 1000000000
      ).toString();
    },
    async draw() {
      const points = [];
      const total = Number(this.total) * 1000000000;
      const step = Math.round(Number(total) / 100);
      for (let index = 0; index < 100; index++) {
        const payout = await total_payout(
          (step * index).toString(),
          total.toString()
        );
        points.push([(step * index) / 1000000000, Number(payout) / 1000000000]);
      }
      const total_reward = await total_payout(
        total.toString(),
        total.toString()
      );
      points.push([total / 1000000000, Number(total_reward) / 1000000000]);
      const payout = await total_payout(
        Math.round(Number(this.staked) * 1000000000).toString(),
        total.toString()
      );
      this.stakedPoint = [[Number(this.staked), Number(payout) / 1000000000]];
      this.points = points;
    },
  },
};
</script>
