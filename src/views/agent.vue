<template>
  <div class="agent">
    <template v-if="is_vip === '普通用户'">
      <div class="agent-module">
        <h3>选择VIP</h3>
        <div class="agent-module-bank">
          <ul>
            <template v-for="(item, index) in agent_list">
              <li @click="checkAgent(item, index)" :key="item">
                <div
                  :class="{'agent-module-bank-num': true,'agent-module-bank-checked': index === agent_index}"
                >
                  <span>{{item.name | format_agent}}</span>
                </div>
              </li>
            </template>
          </ul>
        </div>
      </div>
      <div class="agent-module agent-module-abeam">
        <h3>推荐码</h3>
        <div class="agent-module-recommend">
          <Input type="text" v-model="recommend" placeholder="请输入邀请码" />
        </div>
      </div>
      <div class="agent-module agent-module-abeam">
        <h3>我要当VIP</h3>
        <div class="agent-module-way">
          <ul>
            <li @click="joinAgent">
              <div class="agent-module-pay">
                <span>{{agent_check_pay[0]}}</span>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </template>
    <template v-if="agent.is_pass">
      <div class="agent-module">
        <h3>VIP分润</h3>
        <div class="agent-module-count">
          <div class="agent-module-count-invest">
            <p>{{income_total}}</p>
            <span>用户分润</span>
          </div>
          <div class="agent-module-count-income" @click="goToNext">
            <p>{{recommend_total}}</p>
            <span>VIP推广</span>
          </div>
        </div>
      </div>
      <div class="agent-module agent-module-abeam">
        <h3>我要提现</h3>
        <div class="agent-module-way">
          <div class="agent-module-pay" @click="cashAgent">
            <span>提现到余额</span>
          </div>
        </div>
      </div>
      <div class="agent-module">
        <h3>收益列表</h3>
        <div class="agent-module-income">
          <ul>
            <li>
              <span>获得收益</span>
              <span>团队收益</span>
              <span>下级VIP</span>
              <span>创建时间</span>
            </li>
            <template v-for="item in income_list">
              <li :key="item.agent_num">
                <span>{{item.agent_num}}</span>
                <span>{{item.team_income}}</span>
                <span>{{item.household}}</span>
                <span>{{item.create_time}}</span>
              </li>
            </template>
          </ul>
        </div>
      </div>
      <div class="agent-module">
        <h3>下级VIP</h3>
        <div class="agent-module-income">
          <ul>
            <li>
              <span>VIP级别</span>
              <span>VIP</span>
              <span>创建时间</span>
            </li>
            <template v-for="item in next_agent_list">
              <li :key="item.id">
                <span>{{item.agent_name}}</span>
                <span>{{item.household}}</span>
                <span>{{item.create_time}}</span>
              </li>
            </template>
          </ul>
        </div>
      </div>
    </template>
    <div class="agent-module" @click="is_show_agent =! is_show_agent">
      <h3>VIP协议</h3>
      <div class="agent-module-description" v-show="is_show_agent">
        <p class="agent-module-description-title">白银VIP一次性投资1.5万</p>
        <p class="agent-module-description-title">黄金VIP一次性投资3万</p>
        <p class="agent-module-description-title">钻石VIP一次性投资6万</p>
        <p class="agent-module-description-title">VIP收益</p>
        <p class="agent-module-description-cont">白银VIP拿团体盈利的百分之4</p>
        <p class="agent-module-description-cont">白银VIP直推荐白银VIP得5000和推广盈利的百分之10,间接拿一代2000</p>
        <p class="agent-module-description-cont">黄金VIP拿团体盈利的百分之6</p>
        <p class="agent-module-description-cont">黄金VIP直推白银VIP得7000和推广盈利的百分之10,间接无限2000</p>
        <p class="agent-module-description-cont">黄金VIP直推黄金VIP得1万和推广盈利的百分之15，间接拿一代3000</p>
        <p class="agent-module-description-cont">钻石VIP拿团体盈利的百分之8</p>
        <p class="agent-module-description-cont">钻石VIP直推白银VIP得1万和推广盈利的百分之10，间接拿无限1000</p>
        <p class="agent-module-description-cont">钻石VIP直推黄金VIP得1.6万和推广盈利的百分之15,间接拿无限3000</p>
        <p class="agent-module-description-cont">钻石VIP直推荐钻石VIP得2万和推广盈利的百分之20,间接拿一代1万</p>
      </div>
    </div>
    <div class="agent-sure">
      <div class="agent-sure-btn"></div>
    </div>
    <foot></foot>
    <Modal v-model="show_module" width="220">
      <p slot="header">
        <span>加入VIP</span>
      </p>
      <div>
        <p>请输入交易密码：</p>
        <div class="card-body-handler-password">
          <i-input v-model="cash_pwd" type="password" placeholder="请输入交易密码" style="width: 220px"></i-input>
        </div>
      </div>
      <div slot="footer">
        <i-button type="primary" long :loading="loading" @click="trade_Sure">
          <span>确定</span>
        </i-button>
      </div>
    </Modal>
  </div>
</template><script>
import Clipboard from "clipboard";
import card from "./../components/card_list";
import foot from "./../components/foot";

export default {
  name: "agent",
  data() {
    return {
      invest_show_way: false,
      show_module: false,
      balance_count: 0,
      show_text_trade: false,
      cash_pwd: "",
      agent_list: [],
      agent_index: null,
      check_agent: null,
      agent_check_pay: ["余额转账"],
      is_sure: true,
      recommend: "",
      is_vip: "普通用户",
      super_agent: {},
      income_list: [],
      income_total: 0,
      recommend_total: 0,
      is_create: true,
      is_show_agent: true,
      next_agent_list: [],
      agent: {},
      agents: {}
    };
  },

  created() {
    this.getBalance();
    this.getAgent();
    this.is_vip = localStorage.getItem("is_vip");
    this.getAgentIncome();
    this.getAgentByUser();
    this.getNextAgents();
  },

  filters: {
    format_agent(value) {
      return value + "商";
    }
  },

  components: {
    "card-view": card,
    foot
  },
  methods: {
    goToNext() {
      this.$router.push({ name: "agents" });
    },
    async getAgentByUser() {
      let data = {
        id: localStorage.getItem("user_id")
      };
      try {
        let res = await this.$Http.queryAgentByUser(data);
        this.agent = res.data;
      } catch (error) {
        this.$Message.error(`请求失败:${error}`);
      }
    },
    async getNextAgents() {
      let data = {
        id: localStorage.getItem("user_id")
      };
      let res;
      try {
        res = await this.$Http.queryNextAgents(data);
        res.data.forEach(item => {
          item["agent_name"] = this.agents[`name_${item.agent_id}`];
        });

        this.next_agent_list = res.data;
      } catch (error) {
        if (res.msg) {
          return this.$Message.error(`请求失败:${res.msg}`);
        }
        this.$Message.error(`请求失败:${error}`);
      }
    },
    async getAgentIncome() {
      let data = {
        id: 2,
        user_id: localStorage.getItem("user_id")
      };
      try {
        let res = await this.$Http.queryAgentIncome(data);
        this.income_list = res.data.list;
        this.income_total = res.data.income_total.toFixed(2);
        this.recommend_total = res.data.recommend_total;
      } catch (error) {
        this.$Message.error(`请求失败:${error}`);
      }
    },
    async getAgent() {
      try {
        let res = await this.$Http.queryAgentList();
        this.agent_list = res.data;
        res.data.forEach(item => {
          this.agents[`name_${item.id}`] = item.name;
        });
      } catch (error) {
        this.$Message.error(`请求失败:${error}`);
      }
    },
    checkAgent(item, index) {
      this.agent_index = index;
      this.check_agent = item;
    },
    async joinAgent() {
      if (this.agent.is_pass) {
        return this.$Message.warning("您已经申请VIP了，等候审批！");
      }

      if (this.agent_index === null) {
        return this.$Message.warning("请先选择一个VIP");
      }

      if (Number(this.balance_count) < Number(this.check_agent.cost)) {
        return this.$Message.warning("您的余额不足，请先去充值中心充值！");
      }

      if (this.recommend !== "") {
        let data = {
          invite_code: this.recommend
        };
        let res = await this.$Http.queryInviteAgent(data);
        if (res.success === false) {
          let msg = "请求错误";
          if (res.msg) {
            msg = res.msg;
          }
          return this.$Message.error(msg);
        }

        if (!res.data.is_pass) {
          return this.$Message.warning("推荐您的VIP还未审核！");
        }

        this.super_agent = res.data;
        this.is_create = true;
      }
      this.show_module = true;
    },
    async getBalance() {
      let data = {
        user_id: localStorage.getItem("user_id"),
        is_cash: 1
      };

      let res = null;

      try {
        res = await this.$Http.queryBalanceTotal(data);
        this.balance_count = res.data.count;
      } catch (error) {
        if (res.msg) {
          return this.$Message.error(`请求失败:${res.msg}`);
        }

        this.$Message.error(`请求失败:${error}`);
      }
    },
    async trade_Sure() {
      if (this.cash_pwd === "") {
        return this.$Message.error("密码不能为空！");
      }

      let data = {
        id: localStorage.getItem("user_id"),
        cash_pwd: this.cash_pwd
      };

      this.loading = true;
      let res = null;

      try {
        res = await this.$Http.queryUser(data);
        if (this.is_vip) {
          this.createAgent();
          this.getAgentByUser();
        } else {
          if (Number(this.income_total) + Number(this.recommend_total) === 0) {
            this.loading = false;
            return this.$Message.warning(`提现金额为0不能提现！`);
          }
          this.cashAgent();
        }
      } catch (error) {
        this.loading = false;
        if (res.msg) {
          return this.$Message.error(`请求失败:${res.msg}`);
        }
        this.$Message.error(`请求失败:${error}`);
      }
    },
    async createAgent() {
      let data = {
        user_id: localStorage.getItem("user_id"),
        agent_id: this.check_agent.id,
        name: localStorage.getItem("username")
      };
      if (this.super_agent.agent_id) {
        data.super_name = this.super_agent.super_name;
        data.super_agent = this.super_agent.agent_id;
        data.super_user_id = this.super_agent.user_id;
      }

      let res = null;
      try {
        res = await this.$Http.createAgent(data);
        this.is_vip = this.check_agent.name;
        this.loading = false;
        this.show_module = false;
        this.cash_pwd = "";
        this.$Message.success(`创建成功`);
      } catch (error) {
        this.loading = false;
        this.show_module = false;
        if (res.msg) {
          return this.$Message.error(`请求失败:${res.msg}`);
        }
        this.$Message.error(`请求失败:${error}`);
      }
    },
    async cashAgent() {
      let data = {
        user_id: localStorage.getItem("user_id"),
        balance_num: Number(this.income_total) + Number(this.recommend_total)
      };

      let res = null;
      try {
        res = await this.$Http.updateAgentIncome(data);
        this.loading = false;
        this.show_module = false;
        this.cash_pwd = "";
        this.$Message.success(`提现成功`);
      } catch (error) {
        this.loading = false;
        this.show_module = false;
        if (res.msg) {
          return this.$Message.error(`请求失败:${res.msg}`);
        }
        this.$Message.error(`请求失败:${error}`);
      }
    },
    modal_common(msg) {
      this.modal2 = true;
      this.modal_loading = true;
      this.invest_msg = msg;
    }
  }
};
</script><style lang="less"scoped>
@import "./../assets/css/agent.less";
</style>