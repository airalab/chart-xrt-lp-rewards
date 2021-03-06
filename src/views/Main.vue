<template>
  <div>
    <h1>XRT Liquidity stake reward calc</h1>
    <Chart ref="chart" :log="points" :staked="stakedPoint" />
    <div class="tb">
      <div class="row">
        <div class="col">Total staked</div>
        <div class="col-left">{{ staked }} XRT</div>
      </div>
      <div class="row">
        <div class="col">Real-time circulation</div>
        <div class="col-left">{{ total }} XRT</div>
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
      >Add XRT / ETH Liquidity on Uniswap</a
    >
    <h2>XRT Liquidity events last updates</h2>
    <div style="width: 1024px; margin: 0 auto; text-align: left;">
      <ul>
        <li>XRT Liquidity pool staking event will be continued in August.</li>
        <li>
          The reward will change from the const amount to the weekly reward
          calculation algorithm.
        </li>
        <li>
          New XRT Liquidity pool staking reward distribution algorithm similar
          to
          <a
            href="https://wiki.polkadot.network/docs/en/learn-staking#inflation"
            target="_blank"
            >Polkadot Inflation</a
          >. Depending on the staking participation, the distribution of the
          emission to liquidity providers will change dynamically to provide
          incentives to participate (or not participate) in staking.
        </li>
        <li>
          The ideal stake will be 50% of real-time circulation XRT. For example,
          now real-time circ is 182,559 XRT and staked 29,114 XRT ~16%.
        </li>
        <li>
          XRT Inflation is designed to be up to 25% in the first year, and
          before we launch Robonomics Parachain CC1 emission of XRT will be
          distributed for XRT Liquidity providers.
        </li>
        <li>
          Minimal XRT Inflation designed to be 5%. It means the corridor for XRT
          Liquidity pool staking reward distribution algorithm is 5% - 25% of
          real-time circulation.
        </li>
        <li>
          Minimal stake in August will be 0.1% XRT/ETH pool to be rewarded.
        </li>
        <li>
          This application is a useful tool to get actual data regarding
          real-time circ, current stake amount and weekly reward estimation.
        </li>
      </ul>
      <iframe
        style="width: 100%; height: 500px;"
        src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRpSXinniUIYVr37zWXJDG8PXGY9Xpl0iLLCaBE-ZWUBRHO45pw7wPF6_rQThpqKbE_0cleMy57-18R/pubhtml?gid=1850083171&amp;single=true&amp;widget=true&amp;headers=false"
      ></iframe>
      <br />
      <br />
      <br />
    </div>
  </div>
</template>

<script>
import Web3 from "web3";
import Chart from "../components/Chart.vue";
import init, { total_payout } from "../rewards/xrt_lp_rewards.js";

const web3 = new Web3(
  "https://mainnet.infura.io/v3/c1ea69ab1e0a4c6aa6a9dcd0641aecc7"
);
const contract = new web3.eth.Contract(
  [
    {
      constant: true,
      inputs: [],
      name: "totalSupply",
      outputs: [{ name: "", type: "uint256" }],
      payable: false,
      stateMutability: "view",
      type: "function",
    },
    {
      constant: true,
      inputs: [{ name: "owner", type: "address" }],
      name: "balanceOf",
      outputs: [{ name: "", type: "uint256" }],
      payable: false,
      stateMutability: "view",
      type: "function",
    },
  ],
  "0x7de91b204c1c737bcee6f000aaa6569cf7061cb7"
);

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
      const result = await contract.methods
        .balanceOf("0x3185626c14acb9531d19560decb9d3e5e80681b1")
        .call();
      this.staked = Math.round(Number(result) / 1000000000).toString();
    },
    async loadTotal() {
      const tokensupply = await contract.methods.totalSupply().call();
      const dutchAuction = await contract.methods
        .balanceOf("0x86da63b3341924c88baa5adbb2b8f930cc02e586")
        .call();
      const DAO = await contract.methods
        .balanceOf("0x28a3d3467a3198d1bb5311836036d53c3c64b999")
        .call();
      const publicAmbix = await contract.methods
        .balanceOf("0x06d77d039a6bd049fc9e651b7ecbb2694ac1f96f")
        .call();
      this.total = Math.round(
        (Number(tokensupply) -
          Number(dutchAuction) -
          Number(DAO) -
          Number(publicAmbix)) /
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
  text-align: center;
}
.btn-red:hover {
  background-color: #44713f;
  border: 1px solid #478a43;
}
li {
  margin: 15px;
}
</style>
