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
      staked: "34218.087227096",
      total: "182559.096651262",
    };
  },
  async mounted() {
    await init("xrt_lp_rewards_bg.wasm");
    this.draw();
  },
  methods: {
    async draw() {
      const points = [];
      const total = Number(this.total) * 1000000000;
      const step = Math.round(Number(total) / 50);
      for (let index = 0; index < 50; index++) {
        const payout = await total_payout(
          (step * index).toString(),
          total.toString()
        );
        points.push([(step * index) / 1000000000, Number(payout) / 1000000000]);
      }
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
