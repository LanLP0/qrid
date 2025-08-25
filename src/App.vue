<template>
  <el-space wrap direction="horizontal" alignment="baseline" spacer=" ">
    <el-card class="box-card">
      <template #header>
        <div class="card-header">
          <span>Thông tin</span>
        </div>
      </template>
      <el-form ref="formRef" :model="form" :rules="rules" label-width="auto" label="right">
        <el-form-item label="Họ và Tên" prop="name">
          <el-input v-model="form.name" maxlength="20" show-word-limit clearable @input="checkFormValid" />
        </el-form-item>
        <el-form-item label="Số Điện Thoại" prop="phone">
          <el-input v-model.trim="form.phone" minlength="10" maxlength="10" show-word-limit clearable
            @input="checkFormValid" />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" size="large" :loading="loading" :disabled="!formValid" @click="saveToFile">
            Lưu QR
            <el-icon class="el-icon--right">
              <Download />
            </el-icon>
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <el-card class="box-card" body-style="background: white;">
      <template #header>
        <div class="card-header">
          <span>Preview</span>
        </div>
      </template>
      <div v-loading="loading" style="padding-bottom: 1rem;" ref="previewPane">
        <svg :width="size" :height="size" id="svgcontainer" ref="svgcontainer">
          <svg :width="size" :height="size" version="1.1" xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink">
            <svg v-html="preview"></svg>
          </svg>
        </svg>
        <div class="info-preview">
          <p><strong>{{ form.name }}</strong></p>
        </div>
      </div>
    </el-card>
  </el-space>
</template>

<script>
import {
  ElForm,
  ElFormItem,
  ElCard,
  ElSpace,
  ElInput,
  ElButton,
  ElIcon,
  ElLoading,
} from "element-plus";

import { Download } from "@element-plus/icons-vue";
import { saveAs } from "file-saver";
import QRCode from "qrcode-svg";
import sjcl from 'sjcl';
import html2canvas from "html2canvas";

export default {
  name: "App",
  components: {
    ElForm,
    ElFormItem,
    ElCard,
    ElSpace,
    ElInput,
    ElButton,
    ElIcon,
    Download,
  },
  directives: {
    loading: ElLoading.directive,
  },
  data() {
    const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz!@#$%^&*";
    const length = 10;
    let salt = "";
    for (let i = 0; i < length; i++) {
      salt += chars.charAt(Math.floor(Math.random() * chars.length));
    }

    return {
      form: {
        name: "",
        phone: "",
      },
      size: 350,
      exportType: "PNG",
      exportSize: 1200,
      loading: false,
      formValid: false,
      salt: salt,
      rules: {
        name: [
          { required: true, message: "Vui lòng nhập họ và tên", trigger: "change" },
        ],
        phone: [
          {
            required: true,
            message: "Vui lòng nhập số điện thoại",
            trigger: "change",
          },
          {
            validator: (rule, value, callback) => {
              const trimmed = (value || "").trim();
              const regex = /^\d{10}$/;
              if (!regex.test(trimmed)) {
                callback(new Error("Số điện thoại phải có 10 chữ số"));
              } else {
                callback();
              }
            },
            trigger: "change",
          },
        ],
      },
    };
  },
  methods: {
    checkFormValid() {
      this.$refs.formRef.validate((valid) => {
        this.formValid = valid;
      });
    },
    async export() {
      const fileName = this.fileName;
      const node = this.$refs.previewPane;

      // Take screenshot of preview pane
      const canvas = await html2canvas(node, { backgroundColor: "#ffffff" });

      canvas.toBlob((blob) => {
        saveAs(blob, `${fileName}.png`);
      });
    },
    saveToFile() {
      this.$refs.formRef.validate((valid) => {
        if (!valid) return;

        this.loading = true;
        this.export().finally(() => {
          setTimeout(() => (this.loading = false), 1000);
        });
      });
    },
  },
  computed: {
    fileName() {
      return `ORID-${this.form.name}`;
    },
    content() {
      return `qrid v1\n[Main]\nName=${this.form.name}\nPhoneFirst4=${this.form.phone.slice(0, 4)}\nSalt=${this.salt}\nPhoneHash=${sjcl.codec.hex.fromBits(sjcl.hash.sha256.hash(this.form.phone + this.salt))}`;
    },
    preview() {
      var svg = new QRCode({
        content: this.content,
        padding: 1,
        width: this.size,
        height: this.size,
        ecl: "Q",
        xmlDeclaration: true,
        container: "none",
        join: true,
        pretty: true,
      });
      return svg.svg();
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}

.card-header {
  text-align: left;
}

.box-card {
  max-width: 480px;
  margin: 0 auto;
}

.el-space .el-space__item {
  margin: 8px auto !important;
}
</style>
