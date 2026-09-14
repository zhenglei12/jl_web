<template>
  <div>
    <div class="cus-table-header">
      <list-search v-model="search" :condition="condition" :collection="collection" show-reset></list-search>
    </div>
    <div class="cus-table-header">
      <div class="metric-grid">
        <div v-if="statistic" v-acl="'order-statistics'" class="metric-card">
          <span class="metric-card__icon"><a-icon type="account-book" /></span>
          <span class="metric-card__label">总金额</span>
          <strong class="metric-card__value">{{ formatAmount(statistic.amount_count) }}</strong>
        </div>
        <div v-if="statistic" v-acl="'order-statistics'" class="metric-card metric-card--green">
          <span class="metric-card__icon"><a-icon type="check-circle" /></span>
          <span class="metric-card__label">已回收金额</span>
          <strong class="metric-card__value">{{ formatAmount(statistic.received_amount_count) }}</strong>
        </div>
        <div v-if="statistic" v-acl="'order-statistics'" class="metric-card metric-card--orange">
          <span class="metric-card__icon"><a-icon type="pay-circle" /></span>
          <span class="metric-card__label">本月总金额</span>
          <strong class="metric-card__value">{{ formatAmount(statistic.month_amount_count) }}</strong>
        </div>
        <div v-if="statistic" v-acl="'order-statistics'" class="metric-card metric-card--purple">
          <span class="metric-card__icon"><a-icon type="wallet" /></span>
          <span class="metric-card__label">本月已回收</span>
          <strong class="metric-card__value">{{ formatAmount(statistic.month_received_amount_count) }}</strong>
        </div>
        <div v-if="numbers" v-acl="'order-count.num'" class="metric-card metric-card--cyan">
          <span class="metric-card__icon"><a-icon type="file-text" /></span>
          <span class="metric-card__label">文字统计</span>
          <strong class="metric-card__value">{{ numbers }}</strong>
        </div>
      </div>
      <div class="order-toolbar">
        <a-button class="column-toggle" @click="showAllColumns = !showAllColumns">
          <a-icon :type="showAllColumns ? 'up' : 'down'" />
          {{ showAllColumns ? "收起字段" : "展开字段" }}
        </a-button>
        <a-button-group>
          <a-button v-acl="'order-export'" @click="toExport()">导出</a-button>
          <a-button v-acl="'order-add'" type="primary" @click="toEdit()">新增</a-button>
        </a-button-group>
      </div>
    </div>
    <a-table
      class="order-table"
      :columns="tableColumns"
      :data-source="collection.list"
      :loading="collection.loading"
      :pagination="{
        total: collection.total,
        current: collection.page,
        pageSize: collection.pageSize,
        showSizeChanger: true,
      }"
      :rowClassName="getRowClass"
      :scroll="{ x: showAllColumns ? 2850 : 1470 }"
      bordered
      rowKey="id"
      @change="listChange"
    >
      <template slot="ovhidden" slot-scope="data">
        <a-tooltip :title="data">
          <div class="ov-hidden">{{ data }}</div>
        </a-tooltip>
      </template>
      <template slot="type" slot-scope="data">
        <a-tag :class="['order-tag', getTaskTypeClass(data)]">
          {{ taskTypeMap[data] || "未知类型" }}
        </a-tag>
      </template>
      <template slot="user" slot-scope="data">
        <div class="order-customer">
          <a-tooltip :title="`名称：${data.name || '—'}`">
            <p class="order-customer__line">名称：{{ data.name || "—" }}</p>
          </a-tooltip>
          <!-- <p>电话：{{ data.phone }}</p> -->
          <a-tooltip :title="`销售账号：${data.want_name || '—'}`">
            <p class="order-customer__line">销售账号：{{ data.want_name || "—" }}</p>
          </a-tooltip>
        </div>
      </template>
      <template slot="money" slot-scope="data">
        {{ data.amount - data.received_amount > 0 ? formatAmount(data.amount - data.received_amount) : "已结清" }}
      </template>
      <template slot="amount" slot-scope="data">
        {{ formatAmount(data) }}
      </template>
      <template slot="image" slot-scope="data">
        <img v-if="data" :src="data" alt="图片" class="image" @click="toPreview(data)" />
      </template>
      <template slot="wk_image" slot-scope="data">
        <img v-if="data" :src="data" alt="图片" class="image" @click="toPreview(data)" />
      </template>
      <template slot="ask" slot-scope="data">
        <a v-if="data" @click="toDownload(data)">下载附件</a>
        <span v-else>无附件</span>
      </template>
      <template slot="status" slot-scope="data">
        <a-tag :class="['order-tag', getStatusClass(data)]">
          <i class="order-tag__dot"></i>{{ orderStatusMap[data] || "未知状态" }}
        </a-tag>
      </template>
      <template slot="file" slot-scope="data">
        <a v-if="data" @click="toDownload(data)">下载稿件</a>
        <span v-else>未提交</span>
      </template>
      <template slot="operate" slot-scope="data">
        <div class="cus-nowrap order-actions" @click.stop>
          <span v-acl="'order-update'">
            <a-icon type="edit" title="编辑" @click="toEdit(data)" />
          </span>
          <span v-acl="'order-edit.name'">
            <a-icon type="api" title="分配编辑" @click="toAllot(data.id)" />
          </span>
          <span v-acl="'order-manuscript'">
            <a-icon type="upload" title="上传稿件" @click="toUpload(data.id)" />
          </span>
          <a-dropdown :trigger="['click']" placement="bottomRight">
            <a-button class="action-more" title="更多操作" @click.stop>
              <a-icon type="ellipsis" />
            </a-button>
            <a-menu slot="overlay" class="order-action-menu">
              <a-menu-item v-acl="'order-logs'" @click="toLog(data.id)">
                <a-icon type="file" />订单日志
              </a-menu-item>
              <a-menu-item v-acl="'order-status'" @click="toStatus(data)">
                <a-icon type="swap" />修改状态
              </a-menu-item>
              <a-menu-item v-acl="'order-after'" @click="toAfter(data)">
                <a-icon type="rocket" />售后处理
              </a-menu-item>
              <a-menu-item v-acl="'order-hard.grade'" @click="toGrade(data)">
                <a-icon type="stock" />设置难度
              </a-menu-item>
              <a-menu-divider v-acl="'order-delete'" />
              <a-menu-item v-acl="'order-delete'" class="danger-menu-item">
                <a-popconfirm title="确认删除？" placement="left" @confirm="toDelete(data.id)">
                  <span class="menu-action"><a-icon type="delete" />删除订单</span>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </div>
      </template>
    </a-table>

    <!-- 编辑 -->
    <cus-edit v-model="editVisible" :data="temp" @refresh="_getList"></cus-edit>

    <!-- 分配编辑 -->
    <cus-allot v-model="allotVisible" :data="temp" @refresh="_getList"></cus-allot>

    <!-- 修改状态 -->
    <cus-status v-model="statusVisible" :data="temp" @refresh="_getList"></cus-status>

    <!-- 上传稿件 -->
    <cus-upload v-model="uploadVisible" :data="temp" :classifyList="classifyList" @refresh="_getList"></cus-upload>

    <!-- 详情 -->
    <!-- <cus-detail
      v-model="detailVisible"
      :data="temp"
      @refresh="_getList"
    ></cus-detail> -->

    <!-- 图片预览 -->
    <img-preview v-model="previewVisible" :urls="previewUrl"></img-preview>

    <cus-log v-model="logVisible" :data="temp"></cus-log>

    <!-- 售后 -->
    <cus-after v-model="afterVisible" :data="temp" @refresh="_getList" />

    <!-- 难度 -->
    <cus-grade v-model="gradeVisible" :data="temp" @refresh="_getList" />
  </div>
</template>

<script>
const condition = [
  {
    key: "id",
    placeholder: "ID",
  },
  {
    key: "subject",
    placeholder: "题目",
  },
  {
    key: "word_number",
    placeholder: "字数",
  },
  {
    key: "name",
    placeholder: "客户姓名",
  },
  {
    key: "task_type",
    type: "select",
    options: utils.mapToArray(taskTypeMap),
    placeholder: "任务类型",
  },
  {
    key: "staff_name",
    type: "select",
    placeholder: "客服名称",
    options: [],
    labelKey: "name",
    valueKey: "name",
  },
  {
    key: "edit_name",
    type: "select",
    placeholder: "编辑名称",
    options: [],
    labelKey: "name",
    valueKey: "name",
  },
  {
    key: "classify_id",
    type: "cascader",
    placeholder: "文档分类",
    changeOnSelect: true,
    options: [],
    labelKey: "name",
    valueKey: "id",
  },
  {
    key: "status",
    type: "select",
    options: utils.mapToArray(orderStatusMap),
    placeholder: "状态",
  },
  {
    key: "_date",
    type: "date-in",
  },
  {
    key: "_time",
    type: "date-in",
    placeholder: ["开始截止时间", "结束截止时间"],
  },
  {
    key: "is_final",
    type: "select",
    placeholder: "尾款是否完成",
    options: utils.mapToArray(boolMap),
  },
];

const columns = [
  {
    title: "ID",
    dataIndex: "id",
    width: 70,
  },
  {
    title: "任务类型",
    dataIndex: "task_type",
    width: 100,
    scopedSlots: { customRender: "type" },
  },
  {
    title: "题目",
    dataIndex: "subject",
    width: 180,
    scopedSlots: { customRender: "ovhidden" },
  },
  {
    title: "字数",
    dataIndex: "word_number",
    width: 80,
  },
  {
    title: "任务要求",
    dataIndex: "task_ask",
    width: 180,
    optional: true,
    scopedSlots: { customRender: "ovhidden" },
  },
  {
    title: "客户",
    hidden: ["edit", "edit_admin"],
    width: 160,
    scopedSlots: { customRender: "user" },
  },
  // {
  //   title: "客户电话",
  //   hidden: ["edit", "edit_admin"],
  //   dataIndex: "phone",
  // },
  {
    title: "创建时间",
    dataIndex: "created_at",
    width: 160,
  },
  {
    title: "截止时间",
    dataIndex: "submission_time",
    width: 110,
  },
  {
    title: "订单总额",
    hidden: ["edit", "edit_admin"],
    dataIndex: "amount",
    width: 110,
    scopedSlots: { customRender: "amount" },
  },
  {
    title: "已收金额",
    hidden: ["edit", "edit_admin"],
    dataIndex: "received_amount",
    width: 110,
    scopedSlots: { customRender: "amount" },
  },
  {
    title: "未收尾款",
    hidden: ["edit", "edit_admin"],
    width: 110,
    optional: true,
    scopedSlots: { customRender: "money" },
  },
  {
    title: "付款截图",
    hidden: ["edit", "edit_admin"],
    dataIndex: "pay_img",
    width: 100,
    optional: true,
    scopedSlots: { customRender: "image" },
  },
  {
    title: "尾款截图",
    hidden: ["edit", "edit_admin"],
    dataIndex: "receipt_account",
    width: 100,
    optional: true,
    scopedSlots: { customRender: "wk_image" },
  },
  {
    title: "财务审核",
    hidden: ["edit", "edit_admin"],
    dataIndex: "finance_check",
    width: 100,
    optional: true,
    customRender: (data) => (data == 1 ? "是" : "否"),
  },
  {
    title: "售后金额",
    hidden: ["edit", "edit_admin"],
    dataIndex: "after_banlace",
    width: 100,
    optional: true,
  },
  {
    title: "详细要求",
    dataIndex: "detail_re",
    width: 110,
    optional: true,
    scopedSlots: { customRender: "ask" },
  },
  {
    title: "状态",
    dataIndex: "status",
    width: 110,
    scopedSlots: { customRender: "status" },
  },
  {
    title: "创建客服",
    dataIndex: "staff_name",
    width: 100,
    optional: true,
  },
  {
    title: "文档分类",
    dataIndex: "classify.name",
    width: 110,
    optional: true,
  },
  {
    title: "责任编辑",
    dataIndex: "edit_name",
    width: 100,
  },
  {
    title: "难度等级",
    dataIndex: "hard_grade",
    width: 100,
    optional: true,
  },
  {
    title: "备注",
    dataIndex: "remark",
    width: 160,
    optional: true,
    scopedSlots: { customRender: "ovhidden" },
  },
  {
    title: "稿件下载",
    dataIndex: "manuscript",
    width: 110,
    optional: true,
    scopedSlots: { customRender: "file" },
  },
  {
    title: "操作",
    key: "operate",
    align: "center",
    fixed: "right",
    width: 180,
    scopedSlots: { customRender: "operate" },
  },
];

import listMixin from "../../mixins/list";
import OrderApi from "../../apis/order";
import PublicApi from "../../apis/public";
import utils from "../../libs/utils";
import CusEdit from "./Edit";
import CusStatus from "./Status";
import CusAllot from "./Allot";
import CusUpload from "./Upload";
import CusLog from "./Log";
import CusAfter from "./After";
import CusGrade from "./Grade";
import { taskTypeMap, orderStatusMap, boolMap } from "./mapping";

export default {
  components: {
    CusEdit,
    CusAllot,
    CusStatus,
    CusUpload,
    CusLog,
    CusAfter,
    CusGrade,
  },
  mixins: [listMixin],
  data() {
    return {
      condition,
      columns,
      taskTypeMap,
      orderStatusMap,
      statistic: null,
      numbers: null,
      editVisible: false,
      statusVisible: false,
      allotVisible: false,
      previewVisible: false,
      uploadVisible: false,
      logVisible: false,
      afterVisible: false,
      gradeVisible: false,
      previewUrl: "",
      editorList: [],
      classifyList: [],
      download: false,
      showAllColumns: false,
    };
  },
  computed: {
    tableColumns() {
      return this.showAllColumns ? this.columns : this.columns.filter((column) => !column.optional);
    },
  },
  created() {
    PublicApi.roleUserList("staff").then((res) => {
      let temp = this.condition.find((_) => _.key == "staff_name");
      if (temp) {
        temp.options = res.list;
      }
    });
    PublicApi.roleUserList("edit").then((res) => {
      let temp = this.condition.find((_) => _.key == "edit_name");
      this.editorList = res.list;
      if (temp) {
        temp.options = res.list;
      }
    });
    PublicApi.documentClassify({
      page: 1,
      pageSize: 200,
    }).then((res) => {
      function fmtList(list, level = 1) {
        return list.map((_) => {
          _.level = level;
          if (_.children && _.children.length) {
            _.children = fmtList(_.children, level + 1);
          } else {
            _.isLeaf = true;
            delete _.children;
          }
          return _;
        });
      }
      let temp = this.condition.find((_) => _.key == "classify_id");
      this.classifyList = fmtList(res.list);
      if (temp) {
        temp.options = res.list;
      }
    });
    if (this.$auth.isService) {
      this.condition = this.condition.filter((_) => _.key != "staff_name");
    }
    if (this.$auth.isEditor) {
      this.condition = this.condition.filter((_) => _.key != "edit_name");
      this.columns = this.columns.filter((_) => {
        if (_.hidden) {
          return !~_.hidden.indexOf("edit");
        }
        return true;
      });
    }
    if (this.$auth.isEditAdmin) {
      this.columns = this.columns.filter((_) => {
        if (_.hidden) {
          return !~_.hidden.indexOf("edit_admin");
        }
        return true;
      });
    }
  },
  methods: {
    getTaskTypeClass(value) {
      return `order-tag--type-${value}`;
    },
    formatAmount(value) {
      const amount = Number(value);
      return Number.isFinite(amount) ? amount.toFixed(2) : "0.00";
    },
    getStatusClass(value) {
      const classMap = {
        "-1": "order-tag--waiting",
        1: "order-tag--writing",
        2: "order-tag--revision",
        3: "order-tag--completed",
        4: "order-tag--submitted",
        5: "order-tag--delivered",
      };
      return classMap[value] || "order-tag--default";
    },
    getStatistic() {
      OrderApi.statistic().then((res) => {
        this.statistic = res;
      });
      OrderApi.numbers().then((res) => {
        this.numbers = res.count_num;
      });
    },
    getRowClass(data) {
      switch (data.status) {
        case "1":
          return "bg-yellow";
        case "2":
          return "bg-pink";
        case "3":
          return "bg-purple";
        case "5":
          return "bg-blue";
        case "4":
          return "bg-green";
        default:
          break;
      }
    },
    toStatus(e) {
      this.temp = e;
      this.statusVisible = true;
    },
    toAllot(e) {
      this.temp = {
        id: e,
        editorList: this.editorList,
      };
      this.allotVisible = true;
    },
    toEdit(e) {
      this.temp = e;
      this.editVisible = true;
    },
    toPreview(e) {
      this.previewUrl = e;
      this.previewVisible = true;
    },
    toDownload(e) {
      if (this.download) {
        return this.$message.warn("正在下载...");
      }
      this.download = true;
      utils
        .download(e, e.split("/").pop())
        .then(() => {
          this.$message.success("下载完成");
        })
        .finally(() => (this.download = false));
    },
    toUpload(e) {
      this.temp = e;
      this.uploadVisible = true;
    },
    toDelete(e) {
      OrderApi.remove(e).then(() => {
        this.$message.success("操作成功");
        this._getList();
      });
    },
    toAfter(e) {
      this.temp = e;
      this.afterVisible = true;
    },
    toGrade(e) {
      this.temp = e;
      this.gradeVisible = true;
    },
    toLog(e) {
      this.temp = e;
      this.logVisible = true;
    },
    _getList() {
      this.getStatistic();
      this.collection.loading = true;
      const _search = JSON.parse(JSON.stringify(this.search));
      if (_search && _search._date) {
        _search.created_at = _search._date[0];
        _search.end_time = _search._date[1];
      }
      if (_search && _search._time) {
        _search.submission_time = _search._time[0];
        _search.submission_end_time = _search._time[1];
      }
      if (_search && _search.classify_id) {
        _search.classify_id = _search.classify_id.push();
      }
      OrderApi.list(
        Object.assign(
          {},
          {
            page: this.collection.page,
            pageSize: this.collection.pageSize,
            staff_name: this.$auth.isService ? this.$auth.user().name : undefined,
            edit_name: this.$auth.isEditor ? this.$auth.user().name : undefined,
          },
          _search
        )
      ).then((res) => {
        this.collection.list = res.list;
        this.collection.total = res.total;
        this.collection.loading = false;
      });
    },
    toExport() {
      const _search = JSON.parse(JSON.stringify(this.search));
      if (_search && _search._date) {
        _search.created_at = _search._date[0];
        _search.end_time = _search._date[1];
      }
      if (_search && _search._time) {
        _search.submission_time = _search._date[0];
        _search.submission_end_time = _search._date[1];
      }
      if (_search && _search.classify_id) {
        _search.classify_id = _search.classify_id.push();
      }
      OrderApi.export(
        Object.assign(
          {},
          {
            page: this.collection.page,
            pageSize: this.collection.pageSize,
            staff_name: this.$auth.isService ? this.$auth.user().name : undefined,
            edit_name: this.$auth.isEditor ? this.$auth.user().name : undefined,
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
        utils.export(res, "订单列表").then(() => {
          this.$message.success("下载完成");
        });
      });
    },
  },
};
</script>

<style lang="less" scoped>
.image {
  width: 48px;
  height: 48px;
  padding: 3px;
  border: 1px solid #e1eaf5;
  border-radius: 7px;
  object-fit: cover;
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover {
    border-color: #76a8f5;
    box-shadow: 0 6px 15px rgba(48, 102, 173, 0.14);
    transform: scale(1.04);
  }

  &--small {
    width: 38px;
    height: 38px;
  }
}

.ov-hidden {
  width: 160px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.order-customer {
  width: 100%;
  max-width: 128px;
  min-width: 0;
  overflow: hidden;

  &__line {
    display: block;
    width: 100%;
    margin: 0;
    overflow: hidden;
    line-height: 1.8;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
}

.order-actions {
  justify-content: center;
  gap: 5px;

  .action-more {
    display: inline-flex;
    width: 30px;
    height: 28px;
    align-items: center;
    justify-content: center;
    padding: 0;
    border-color: #dce8f7;
    color: #2f7cf6;
    background: #f4f8ff;

    &:hover,
    &:focus {
      border-color: #2f7cf6;
      color: #fff;
      background: #2f7cf6;
    }
  }
}

.order-toolbar {
  display: flex;
  flex-shrink: 0;
  align-items: center;
  gap: 10px;
}

.column-toggle {
  color: #2f7cf6;
  border-color: #cfe0f6;
  background: #f7faff;
}

.order-tag {
  margin: 0;
  min-width: 58px;
  padding: 3px 11px;
  border: 1px solid transparent;
  border-radius: 14px;
  font-weight: 600;
  font-size: 12px;
  line-height: 20px;
  text-align: center;
  box-shadow: 0 3px 8px rgba(42, 83, 135, 0.08);

  &__dot {
    display: inline-block;
    width: 5px;
    height: 5px;
    margin: 0 6px 1px 0;
    border-radius: 50%;
    background: currentColor;
  }

  &--type-1 {
    color: #176ee5;
    border-color: #c5dcff;
    background: #deecff;
  }

  &--type-2 {
    color: #6846d8;
    border-color: #d9ccff;
    background: #ebe5ff;
  }

  &--type-3 {
    color: #d8670b;
    border-color: #ffd3ad;
    background: #ffead7;
  }

  &--type-4 {
    color: #07866f;
    border-color: #b8ecdf;
    background: #d9f7ef;
  }

  &--type-5 {
    color: #087da9;
    border-color: #bee7f7;
    background: #daf3fc;
  }

  &--waiting {
    color: #66768c;
    border-color: #d5dde8;
    background: #e9eef4;
  }

  &--writing {
    color: #176ee5;
    border-color: #c5dcff;
    background: #deecff;
  }

  &--revision {
    color: #e34b3f;
    border-color: #ffc9c5;
    background: #ffe4e2;
  }

  &--completed {
    color: #078a6e;
    border-color: #b6ecdd;
    background: #d8f7ed;
  }

  &--submitted {
    color: #6342d5;
    border-color: #d7caff;
    background: #e9e2ff;
  }

  &--delivered {
    color: #bd7000;
    border-color: #ffdaa1;
    background: #ffedcf;
  }

  &--default {
    color: #71829a;
    background: #f0f3f7;
  }
}

.order-expanded {
  display: grid;
  grid-template-columns: repeat(4, minmax(150px, 1fr));
  gap: 1px;
  overflow: hidden;
  border: 1px solid #e1eaf5;
  border-radius: 9px;
  background: #e1eaf5;

  .expanded-item {
    display: flex;
    min-height: 64px;
    flex-direction: column;
    justify-content: center;
    padding: 10px 14px;
    background: #fbfdff;

    &--wide {
      grid-column: span 2;
    }
  }

  .expanded-label {
    margin-bottom: 5px;
    color: #8b9ab0;
    font-size: 11px;
  }

  .expanded-value {
    color: #405574;
    line-height: 1.6;
    white-space: normal;
    word-break: break-word;
  }
}

/deep/ .ant-table-expanded-row > td {
  padding: 12px 16px !important;
  background: #f5f9fe !important;
}

/deep/ .ant-table-row-expand-icon {
  border-color: #bdd0e8;
  color: #2f7cf6;
  background: #f2f7ff;
}

/deep/ .ant-table-fixed-right {
  box-shadow: -8px 0 18px rgba(32, 69, 113, 0.08);
}

/deep/ .ant-table-fixed-right .ant-table-thead > tr > th,
/deep/ .ant-table-fixed-right .ant-table-tbody > tr > td {
  background: #fff;
}

/deep/ .ant-table-fixed-right .ant-table-thead > tr > th {
  background: #f7faff;
}

@media (max-width: 1100px) {
  .order-expanded {
    grid-template-columns: repeat(2, minmax(140px, 1fr));
  }
}

/deep/ .bg {
  &-pink {
    background-color: #fff4f4;
  }

  &-green {
    background-color: #f0fbf7;
  }

  &-yellow {
    background-color: #fffaf0;
  }

  &-blue {
    background-color: #f2f7ff;
  }

  &-purple {
    background-color: #f8f4ff;
  }
}

/deep/ tr:hover > td {
  filter: brightness(0.99);
}
</style>
