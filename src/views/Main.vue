<template>
  <div>
    <h1>XRT Liquidity stake reward calc</h1>
    <Chart ref="chart" :log="points" :staked="stakedPoint" />
    <div class="tb">
      <div class="row">
        <div class="col">Total staked</div>
        <div class="col-left">{{staked}} XRT</div>
      </div>
      <div class="row">
        <div class="col">Real-time circulation</div>
        <div class="col-left">{{total}} XRT</div>
      </div>
    </div>
    <h2>Why stake?</h2>
    <p>25% year emission</p>
    <p>50% targeted active staking</p>
    <br />
    <a
      href="https://app.uniswap.org/#/add/0x7de91b204c1c737bcee6f000aaa6569cf7061cb7/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"
      target="_blank"
      class="btn-red"
    >Add XRT / ETH Liquidity on Uniswap</a>
    <h2>XRT Liquidity events last updates</h2>
    <div style="width: 1024px;margin: 0 auto;text-align: left;">
      <ul>
        <li>XRT Liquidity pool staking event will be continued in August.</li>
        <li>The reward will change from the const amount to the weekly reward calculation algorithm.</li>
        <li>
          New XRT Liquidity pool staking reward distribution algorithm similar to
          <a
            href="https://wiki.polkadot.network/docs/en/learn-staking#inflation"
            target="_blank"
          >Polkadot Inflation</a>. Depending on the staking participation, the distribution of the emission to liquidity providers will change dynamically to provide incentives to participate (or not participate) in staking.
        </li>
        <li>The ideal stake will be 50% of real-time circulation XRT. For example, now real-time circ is 182,559 XRT and staked 29,114 XRT ~16%.</li>
        <li>XRT Inflation is designed to be up to 25% in the first year, and before we launch Robonomics Parachain CC1 emission of XRT will be distributed for XRT Liquidity providers.</li>
        <li>Minimal XRT Inflation designed to be 5%. It means the corridor for XRT Liquidity pool staking reward distribution algorithm is 5% - 25% of real-time circulation.</li>
        <li>Minimal stake in August will be 0.1% XRT/ETH pool to be rewarded.</li>
        <li>This application is a useful tool to get actual data regarding real-time circ, current stake amount and weekly reward estimation.</li>
      </ul>
    </div>
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
      total: "0",
    };
  },
  async mounted() {
    await init("xrt_lp_rewards_bg.wasm");
    await this.loadStaked();
    await this.loadTotal();
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
    async loadTotal() {
      const tokensupply = await axios.get(
        "https://api.etherscan.io/api?module=stats&action=tokensupply&contractaddress=0x7de91b204c1c737bcee6f000aaa6569cf7061cb7&apikey=U6AM52QNQZRTE6P4J8RUH1U5JB3I3SUWIZ"
      );
      const dutchAuction = await axios.get(
        "https://api.etherscan.io/api?module=account&action=tokenbalance&contractaddress=0x7de91b204c1c737bcee6f000aaa6569cf7061cb7&address=0x86da63b3341924c88baa5adbb2b8f930cc02e586&tag=latest&apikey=U6AM52QNQZRTE6P4J8RUH1U5JB3I3SUWIZ"
      );
      const DAO = await axios.get(
        "https://api.etherscan.io/api?module=account&action=tokenbalance&contractaddress=0x7de91b204c1c737bcee6f000aaa6569cf7061cb7&address=0x28a3d3467a3198d1bb5311836036d53c3c64b999&tag=latest&apikey=U6AM52QNQZRTE6P4J8RUH1U5JB3I3SUWIZ"
      );
      const publicAmbix = await axios.get(
        "https://api.etherscan.io/api?module=account&action=tokenbalance&contractaddress=0x7de91b204c1c737bcee6f000aaa6569cf7061cb7&address=0x06d77d039a6bd049fc9e651b7ecbb2694ac1f96f&tag=latest&apikey=U6AM52QNQZRTE6P4J8RUH1U5JB3I3SUWIZ"
      );
      this.total = Math.round(
        (Number(tokensupply.data.result) -
          Number(dutchAuction.data.result) -
          Number(DAO.data.result) -
          Number(publicAmbix.data.result)) /
          1000000000
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

<style scoped>
.tb {
  width: 500px;
  margin: 40px auto;
}
.tb .row {
  margin: 10px;
  overflow: hidden;
}
.tb .col {
  float: left;
  width: 50%;
  text-align: right;
}
.tb .col-left {
  text-align: left;
  padding-left: 20px;
  font-weight: bold;
  float: left;
}
button {
  font-size: 20px;
  padding: 10px;
}
.btn-red {
  display: block;
  background-color: #2e9a21;
  border: 1px solid #478a43;
  width: 300px;
  color: blanchedalmond;
  margin: 0 auto;
  padding: 10px;
  text-decoration: none;
}
.btn-red:hover {
  background-color: #44713f;
  border: 1px solid #478a43;
}
li {
  margin: 15px;
}
</style>
