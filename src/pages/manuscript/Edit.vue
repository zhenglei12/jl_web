<template>
  <a-modal :visible="visible" title="上传稿件" destroyOnClose :confirmLoading="loading" @cancel="close" @ok="submit">
    <a-form-model ref="form" :model="form" :label-col="{ span: 4 }" :wrapper-col="{ span: 19 }">
      <a-form-model-item label="题目" required>
        <a-input v-model="form.subject" allow-clear />
      </a-form-model-item>
      <a-form-model-item label="字数" required>
        <a-input-number v-model="form.word_number" :min="0" :precision="0" />
      </a-form-model-item>
      <a-form-model-item label="内容类型" required>
        <a-cascader
          :default-value="form.classify_id"
          v-model="form.classify_id"
          :options="classifyList"
          placeholder="内容类型"
          :fieldNames="{
            label: 'name',
            value: 'id',
            children: 'children',
          }"
        />
      </a-form-model-item>
      <a-upload-dragger :fileList="fileList" :customRequest="request" :remove="remove">
        <p class="ant-upload-drag-icon">
          <a-icon type="inbox" />
        </p>
        <p class="ant-upload-text">点击或拖拽文件到此区域</p>
      </a-upload-dragger>
    </a-form-model>
  </a-modal>
</template>

<script>
import editMixin from "../../mixins/edit";
import ManuscriptApi from "../../apis/manuscript";
import upload from "../../libs/upload";
import utils from "../../libs/utils";

export default {
  mixins: [editMixin],
  props: {
    classifyList: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      loading: false,
      fileList: [],
      // fileType: ["xls", "xlsx", "csv"],
      form: {},
    };
  },
  watch: {
    visible() {
      this.form = {
        id: this.R.id,
        subject: this.R.subject,
        word_number: this.R.word_number,
        classify_id: this.R.classify_id,
      };
      this.fileList = this.R.manuscript
        ? [
            {
              uid: utils.uuid(),
              status: "done",
              name: this.R.manuscript.split("/").pop(),
              url: this.R.manuscript,
            },
          ]
        : [];
    },
  },
  methods: {
    // beforeUpload(e) {
    //   let type = e.name.substring(e.name.lastIndexOf(".") + 1);
    //   if (!~this.fileType.indexOf(type.toLowerCase())) {
    //     this.$message.error(`文件${e.name}上传失败,不支持该文件类型!`);
    //     e.status = "unqualified";
    //     return false;
    //   }
    //   return true;
    // },
    remove() {
      this.fileList = [];
    },
    request(e) {
      this.fileList = [e.file];
    },
    submit() {
      this.loading = true;
      upload
        .uploadList(this.fileList, ["lingganyin"])
        .then(() => {
          this.form.manuscript = upload.getRources(this.fileList)[0];
          if (this.isEdit) {
            return ManuscriptApi.update({ ...this.form }).then((res) => {
              this.$message.success("保存成功");
              this.$emit("refresh", res);
              this.close();
            });
          } else {
            return ManuscriptApi.create({ ...this.form }).then((res) => {
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
