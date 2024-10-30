<template>
  <a-modal
    :visible="visible"
    :title="title"
    destroyOnClose
    :confirmLoading="loading"
    @cancel="close"
    @ok="submit"
  >
    <a-form-model
      ref="form"
      :model="form"
      :label-col="{ span: 4 }"
      :wrapper-col="{ span: 19 }"
    >
      <a-form-model-item label="用户名" required prop="username">
        <a-input v-model="form.username" allow-clear />
      </a-form-model-item>
      <a-form-model-item label="密码" :required="!isEdit" prop="password">
        <a-input v-model="form.password" allow-clear />
      </a-form-model-item>
      <a-form-model-item label="邮箱" required prop="email">
        <a-input v-model="form.email" allow-clear />
      </a-form-model-item>
      <!-- <a-form-model-item label="状态" prop="jobGroupName">
      <a-radio-group name="radioGroup" :default-value="1">
        <a-radio :value="1">开启</a-radio>
        <a-radio :value="-1">关闭</a-radio>
      </a-radio-group>
    </a-form-model-item> -->
    </a-form-model>
  </a-modal>
</template>

<script>
import editMixin from "../../mixins/edit";
import UserApi from "../../apis/user";

export default {
  mixins: [editMixin],
  data() {
    return {
      loading: false,
      form: {},
    };
  },
  watch: {
    visible(e) {
      if (e) {
        this.form = {
          id: this.R.id,
          username: this.R.name,
          email: this.R.email,
        };
      }
    },
  },
  computed: {
    title() {
      return this.isEdit ? "编辑用户" : "新增用户";
    },
  },
  methods: {
    submit() {
      this.loading = true;
      if (this.isEdit) {
        return UserApi.update({ ...this.form })
          .then((res) => {
            this.$message.success("保存成功");
            this.$emit("refresh", res);
            this.close();
          })
          .finally(() => (this.loading = false));
      } else {
        return UserApi.create({ ...this.form })
          .then((res) => {
            this.$message.success("保存成功");
            this.$emit("refresh", res);
            this.close();
          })
          .finally(() => (this.loading = false));
      }
    },
  },
};
</script>
