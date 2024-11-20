<template>
  <a-modal
    :visible="visible"
    :title="title"
    destroyOnClose
    :confirmLoading="loading"
    width="800px"
    @cancel="close"
    @ok="submit"
  >
    <a-form-model ref="form" :model="form" :label-col="{ span: 6 }" :wrapper-col="{ span: 17 }">
      <a-row>
        <a-col span="12">
          <a-form-model-item label="题目" required>
            <a-input v-model="form.subject" allow-clear />
          </a-form-model-item>
        </a-col>
        <a-col span="12">
          <a-form-model-item label="字数" required>
            <a-input-number v-model="form.word_number" :min="0" :precision="0" />
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <a-col span="12">
          <a-form-model-item label="订单总额">
            <a-input-number v-model="form.amount" :min="0" :disabled="isEdit && $auth.isService" />
          </a-form-model-item>
        </a-col>
        <a-col span="12">
          <a-form-model-item label="已收金额">
            <a-input-number v-model="form.received_amount" :min="0" />
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <a-col span="12">
          <a-form-model-item label="任务类型" required>
            <a-select v-model="form.task_type" allowClear :dropdownMatchSelectWidth="false">
              <a-select-option v-for="(option, index) in typeList" :key="index" :value="option.value | string">
                {{ option.label }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col span="12">
          <a-form-model-item label="客户名称" required>
            <a-input v-model="form.name" allow-clear />
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <!-- <a-col span="12">
          <a-form-model-item label="客户电话">
            <a-input v-model="form.phone" allow-clear />
          </a-form-model-item>
        </a-col> -->
        <a-col span="12">
          <a-form-model-item label="销售账号">
            <a-input v-model="form.want_name" allow-clear />
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-form-model-item :label-col="{ span: 3 }" :wrapper-col="{ span: 20 }" label="写作要求">
        <a-textarea v-model="form.task_ask" :autoSize="{ minRows: 3, maxRows: 5 }" placeholder="写作要求" />
      </a-form-model-item>
      <a-form-model-item :label-col="{ span: 3 }" :wrapper-col="{ span: 20 }" label="详细要求" required>
        <a-upload
          :fileList="askList"
          :customRequest="(e) => cusImgUpload('askList', e)"
          :remove="
            () => {
              askList = [];
            }
          "
        >
          <a-button>上传</a-button>
        </a-upload>
      </a-form-model-item>
      <a-row>
        <a-col span="12">
          <a-form-model-item label="交稿日期" required>
            <a-date-picker v-model="form.submission_time" valueFormat="YYYY-MM-DD" />
          </a-form-model-item>
        </a-col>
        <a-col span="12">
          <a-form-model-item label="支付方式">
            <a-select v-model="form.pay_type" allowClear :dropdownMatchSelectWidth="false">
              <a-select-option v-for="(option, index) in payList" :key="index" :value="option.value | string">
                {{ option.label }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <a-col span="12">
          <a-form-model-item label="付款截图">
            <a-upload
              list-type="picture-card"
              :file-list="imgList_1"
              :customRequest="(e) => cusImgUpload('imgList_1', e)"
              :remove="
                () => {
                  imgList_1 = [];
                }
              "
              @preview="toPreview"
            >
              <div v-if="!imgList_1.length">
                <a-icon type="plus" />
                <div class="ant-upload-text">上传</div>
              </div>
            </a-upload>
            <!-- 图片预览 -->
            <img-preview v-model="previewVisible" :urls="previewUrl"></img-preview>
          </a-form-model-item>
        </a-col>
        <a-col span="12">
          <a-form-model-item label="尾款截图">
            <a-upload
              list-type="picture-card"
              :file-list="imgList_2"
              :customRequest="(e) => cusImgUpload('imgList_2', e)"
              :remove="
                () => {
                  imgList_2 = [];
                }
              "
              @preview="toPreview"
            >
              <div v-if="!imgList_2.length">
                <a-icon type="plus" />
                <div class="ant-upload-text">上传</div>
              </div>
            </a-upload>
          </a-form-model-item>
        </a-col>
      </a-row>
      <!-- <a-form-model-item
        :label-col="{ span: 3 }"
        :wrapper-col="{ span: 20 }"
        label="尾款金额"
      >
        <a-input-number
          v-model="form.receipt_time"
          :min="0"
          :disabled="!($auth.isAdmin || $auth.isFinance)"
        />
      </a-form-model-item> -->
      <a-form-model-item :label-col="{ span: 3 }" :wrapper-col="{ span: 20 }" label="备注">
        <a-textarea v-model="form.remark" :autoSize="{ minRows: 3, maxRows: 5 }" placeholder="备注" />
      </a-form-model-item>
    </a-form-model>
  </a-modal>
</template>

<script>
import editMixin from "../../mixins/edit";
import OrderApi from "../../apis/order";
import utils from "../../libs/utils";
import { taskTypeMap, payTypeMap } from "./mapping";
import upload from "../../libs/upload";

export default {
  mixins: [editMixin],
  data() {
    return {
      loading: false,
      typeList: utils.mapToArray(taskTypeMap),
      payList: utils.mapToArray(payTypeMap),
      imgList_1: [],
      imgList_2: [],
      askList: [],
      form: {},
      previewVisible: false,
      previewUrl: "",
    };
  },
  watch: {
    visible(e) {
      if (e) {
        this.form = {
          id: this.R.id,
          subject: this.R.subject,
          word_number: this.R.word_number,
          task_type: this.R.task_type && this.R.task_type.toString(),
          task_ask: this.R.task_ask,
          submission_time: this.R.submission_time,
          name: this.R.name,
          // phone: this.R.phone,
          want_name: this.R.want_name,
          amount: this.R.amount,
          received_amount: this.R.received_amount,
          pay_type: this.R.pay_type,
          remark: this.R.remark,
          receipt_time: this.R.receipt_time,
        };
        this.askList = this.R.detail_re
          ? [
              {
                uid: utils.uuid(),
                status: "done",
                name: this.R.detail_re.split("/").pop(),
                url: this.R.detail_re,
              },
            ]
          : [];
        this.imgList_1 = this.R.pay_img
          ? [
              {
                uid: utils.uuid(),
                status: "done",
                name: this.R.pay_img.split("/").pop(),
                url: this.R.pay_img,
              },
            ]
          : [];
        this.imgList_2 = this.R.receipt_account
          ? [
              {
                uid: utils.uuid(),
                status: "done",
                name: this.R.receipt_account.split("/").pop(),
                url: this.R.receipt_account,
              },
            ]
          : [];
      }
    },
  },
  computed: {
    title() {
      return this.isEdit ? "编辑订单" : "新增订单";
    },
  },
  methods: {
    async cusImgUpload(fileList, { file }) {
      file.url = await utils.getBase64(file);
      this[fileList] = [file];
    },
    toPreview(e) {
      this.previewUrl = e.url;
      this.previewVisible = true;
    },
    submit() {
      this.loading = true;
      Promise.all([
        upload.uploadList(this.askList, ["lingganyin"]),
        upload.uploadList(this.imgList_1, ["lingganyin"]),
        upload.uploadList(this.imgList_2, ["lingganyin"]),
      ])
        .then(() => {
          this.form.detail_re = upload.getRources(this.askList)[0];
          this.form.pay_img = upload.getRources(this.imgList_1)[0];
          this.form.receipt_account = upload.getRources(this.imgList_2)[0];
          if (this.isEdit) {
            return OrderApi.update({ ...this.form }).then((res) => {
              this.$message.success("保存成功");
              this.$emit("refresh", res);
              this.close();
            });
          } else {
            return OrderApi.create({ ...this.form }).then((res) => {
              this.$message.success("保存成功");
              this.$emit("refresh", res);
              this.close();
            });
          }
        })
        .finally(() => (this.loading = false));
    },
  },
};
</script>

<style lang="less" scoped>
/deep/ .ant-upload-list-text {
  width: 200px;
}
</style>
