<template>
  <div>
    <div class="cus-table-header">
      <list-search
        v-model="search"
        :condition="condition"
        :collection="collection"
      ></list-search>
    </div>
    <div class="cus-table-header">
      <div class="metric-grid staff-metrics">
        <div class="metric-card">
          <span class="metric-card__icon"><a-icon type="account-book" /></span>
          <span class="metric-card__label">总金额</span>
          <strong class="metric-card__value">{{ amountCount }}</strong>
        </div>
        <div class="metric-card metric-card--green">
          <span class="metric-card__icon"><a-icon type="check-circle" /></span>
          <span class="metric-card__label">已回收金额</span>
          <strong class="metric-card__value">{{ receivedAmountCount }}</strong>
        </div>
        <div class="metric-card metric-card--orange">
          <span class="metric-card__icon"><a-icon type="wallet" /></span>
          <span class="metric-card__label">总尾款金额</span>
          <strong class="metric-card__value">{{ receiptAmountCount }}</strong>
        </div>
        <div class="metric-card metric-card--purple">
          <span class="metric-card__icon"><a-icon type="tool" /></span>
          <span class="metric-card__label">总售后金额</span>
          <strong class="metric-card__value">{{ afterAmountCount }}</strong>
        </div>
      </div>
      <a-button
        v-acl="'staff-statistics.export'"
        type="primary"
        @click="toExport()"
        >导出</a-button
      >
    </div>
    <a-table
      :columns="columns"
      :data-source="collection.list"
      :loading="collection.loading"
      :pagination="{
        total: collection.total,
        current: collection.page,
        pageSize: collection.pageSize,
        showSizeChanger: true,
      }"
      bordered
      rowKey="id"
      @change="listChange"
    >
    </a-table>
  </div>
</template>

<script>
const condition = [
  {
    key: "name",
    type: "select",
    placeholder: "客服名称",
    options: [],
    labelKey: "name",
    valueKey: "name",
  },
  {
    key: "_date",
    type: "date-in",
  },
];

const columns = [
  {
    title: "",
    dataIndex: "id",
  },
  {
    title: "客服名称",
    dataIndex: "staff_name",
  },
  {
    title: "总金额",
    dataIndex: "amount",
  },
  {
    title: "已回收金额",
    dataIndex: "received_amount",
  },
  {
    title: "尾款金额",
    dataIndex: "receipt_time",
  },
  {
    title: "售后金额",
    dataIndex: "after_banlace",
  },
];

import listMixin from "@/mixins/list";
import StatisticApi from "@/apis/statistic";
import PublicApi from "@/apis/public";
import Utils from "@/libs/utils";

export default {
  mixins: [listMixin],
  data() {
    return {
      condition,
      columns,
      amountCount: 0,
      receivedAmountCount: 0,
      receiptAmountCount: 0,
      afterAmountCount: 0,
    };
  },
  created() {
    PublicApi.roleUserList("staff").then((res) => {
      let temp = this.condition.find((_) => _.key == "name");
      this.editorList = res.list;
      if (temp) {
        temp.options = res.list;
      }
    });
  },
  methods: {
    toExport() {
      let _search = JSON.parse(JSON.stringify(this.search));
      if (_search && _search._date) {
        _search.created_at = _search._date[0];
        _search.end_time = _search._date[1];
        delete _search._date;
      }
      StatisticApi.export(
        Object.assign(
          {},
          {
            page: this.collection.page,
            pageSize: this.collection.pageSize,
          },
          _search
        )
      ).then((res) => {
        if (res.type === "application/json") {
          try {
            const render = new FileReader();
            render.readAsText(res);
            render.onload = (e) => {
              const json = JSON.parse(e.target.result);
              this.$notification.error({
                message: "业务错误",
                description: json.message,
              });
            };
          } catch {
            // eslint-disable-next-line
          }
          return;
        }
        Utils.export(res, "订单列表").then(() => {
          this.$message.success("下载完成");
        });
      });
    },
    _getList() {
      this.collection.loading = true;
      let _search = JSON.parse(JSON.stringify(this.search));
      if (_search && _search._date) {
        _search.created_at = _search._date[0];
        _search.end_time = _search._date[1];
        delete _search._date;
      }
      StatisticApi.list(
        Object.assign(
          {},
          {
            page: this.collection.page,
            pageSize: this.collection.pageSize,
          },
          _search
        )
      ).then((res) => {
        this.amountCount = res.amount_count;
        this.receivedAmountCount = res.received_amount_count;
        this.receiptAmountCount = res.receipt_amount_count;
        this.afterAmountCount = res.after_amount_count;
        const list = res.list;
        list.data.forEach(
          (_, i) =>
            (_.id =
              (this.collection.page - 1) * this.collection.pageSize + i + 1)
        );
        this.collection.list = list.data;
        this.collection.total = list.total;
        this.collection.loading = false;
      });
    },
  },
};
</script>

<style lang="less" scoped>
.staff-metrics {
  grid-template-columns: repeat(4, minmax(170px, 1fr));
}
</style>
