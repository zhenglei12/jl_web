<template>
  <div class="mobile-order">
    <header class="mobile-order__header">
      <div><strong>新增订单</strong><span>DeerVison</span></div>
      <a-button type="link" @click="logout">退出</a-button>
    </header>
    <main class="mobile-order__card">
      <a-form-model ref="form" :model="form" :rules="rules" layout="vertical">
        <a-form-model-item label="题目" prop="subject"><a-input v-model="form.subject" placeholder="请输入题目" allow-clear /></a-form-model-item>
        <div class="mobile-order__row">
          <a-form-model-item label="任务类型" prop="task_type">
            <a-select v-model="form.task_type" placeholder="请选择"><a-select-option v-for="item in typeList" :key="item.value" :value="String(item.value)">{{ item.label }}</a-select-option></a-select>
          </a-form-model-item>
          <a-form-model-item label="字数" prop="word_number"><a-input-number v-model="form.word_number" class="full-width" :min="0" :precision="0" placeholder="字数" /></a-form-model-item>
        </div>
        <div class="mobile-order__row">
          <a-form-model-item label="客户名称" prop="name"><a-input v-model="form.name" placeholder="客户名称" allow-clear /></a-form-model-item>
          <a-form-model-item label="销售账号"><a-input v-model="form.want_name" placeholder="销售账号" allow-clear /></a-form-model-item>
        </div>
        <div class="mobile-order__row">
          <a-form-model-item label="订单总额"><a-input-number v-model="form.amount" class="full-width" :min="0" :precision="2" /></a-form-model-item>
          <a-form-model-item label="已收金额"><a-input-number v-model="form.received_amount" class="full-width" :min="0" :precision="2" /></a-form-model-item>
        </div>
        <a-form-model-item label="交稿日期" prop="submission_time"><a-date-picker v-model="form.submission_time" class="full-width" valueFormat="YYYY-MM-DD" /></a-form-model-item>
        <a-form-model-item label="支付方式"><a-select v-model="form.pay_type" placeholder="请选择支付方式"><a-select-option v-for="item in payList" :key="item.value" :value="String(item.value)">{{ item.label }}</a-select-option></a-select></a-form-model-item>
        <a-form-model-item label="写作要求"><a-textarea v-model="form.task_ask" :autoSize="{ minRows: 3, maxRows: 6 }" placeholder="请输入写作要求" /></a-form-model-item>
        <a-form-model-item class="mobile-file-item" label="详细要求附件"><a-upload :fileList="askList" :customRequest="(e) => pickFile('askList', e)" :remove="() => (askList = [])"><a-button block><a-icon type="upload" />选择附件</a-button></a-upload></a-form-model-item>
        <a-form-model-item class="mobile-file-item" label="付款截图"><a-upload :fileList="payListFile" :customRequest="(e) => pickFile('payListFile', e)" :remove="() => (payListFile = [])" list-type="picture"><a-button block><a-icon type="picture" />选择图片</a-button></a-upload></a-form-model-item>
        <a-form-model-item label="备注"><a-textarea v-model="form.remark" :autoSize="{ minRows: 2, maxRows: 4 }" placeholder="备注" /></a-form-model-item>
        <a-button block type="primary" size="large" :loading="loading" @click="submit">提交新增订单</a-button>
      </a-form-model>
    </main>
  </div>
</template>

<script>
import OrderApi from "../../apis/order";
import upload from "../../libs/upload";
import utils from "../../libs/utils";
import { taskTypeMap, payTypeMap } from "../order/mapping";

export default {
  data() {
    return {
      typeList: utils.mapToArray(taskTypeMap), payList: utils.mapToArray(payTypeMap),
      loading: false, askList: [], payListFile: [],
      form: { amount: 0, received_amount: 0 },
      rules: {
        subject: [{ required: true, message: "请输入题目" }], task_type: [{ required: true, message: "请选择任务类型" }],
        word_number: [{ required: true, message: "请输入字数" }], name: [{ required: true, message: "请输入客户名称" }],
        submission_time: [{ required: true, message: "请选择交稿日期" }],
      },
    };
  },
  methods: {
    pickFile(target, { file }) { this[target] = [file]; },
    submit() {
      if (this.loading) return;
      this.loading = true;
      this.$refs.form.validate()
        .then(() => Promise.all([upload.uploadList(this.askList, ["lingganyin"]), upload.uploadList(this.payListFile, ["lingganyin"])]))
        .then(() => OrderApi.create({ ...this.form, detail_re: upload.getRources(this.askList)[0], pay_img: upload.getRources(this.payListFile)[0] }))
        .then(() => {
          this.$message.success("提交成功");
          this.form = { amount: 0, received_amount: 0 };
          this.askList = [];
          this.payListFile = [];
        })
        .finally(() => (this.loading = false));
    },
    logout() { this.$auth.logout("/mobile/login"); },
  },
};
</script>

<style lang="less" scoped>
.mobile-order { min-height: 100vh; padding-bottom: 24px; background: #f4f8fd; }
.mobile-order__header { display: flex; align-items: center; justify-content: space-between; padding: 16px 18px; color: #fff; background: #173b70; }
.mobile-order__header strong { display: block; font-size: 18px; }.mobile-order__header span { color: #b9cceb; font-size: 10px; letter-spacing: 1.5px; }.mobile-order__header .ant-btn { color: #dbe9ff; }
.mobile-order__card { margin: 18px 14px; padding: 18px 16px 20px; border-radius: 14px; background: #fff; box-shadow: 0 8px 26px rgba(31, 67, 111, .08); }
.mobile-order__row { display: flex; gap: 12px; }.mobile-order__row > * { min-width: 0; flex: 1; }.full-width { width: 100%; }.mobile-order /deep/ .ant-select, .mobile-order /deep/ .ant-calendar-picker { width: 100%; }
.mobile-file-item { margin-bottom: 30px !important; }
.mobile-order /deep/ .ant-upload-list { display: block; clear: both; min-height: 0; margin-top: 8px; margin-bottom: 4px; overflow: visible; }
.mobile-order /deep/ .ant-upload-list-item { position: relative; display: flex; align-items: center; }
@media (max-width: 420px) { .mobile-order__row { display: block; } }
</style>
