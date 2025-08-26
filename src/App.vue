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
      <div v-loading="loading" style="padding: 2em;" ref="previewPane">
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

  <!-- GitHub credit button -->
  <div class="credit">
    <el-button type="text" size="small" @click="openGithub" class="github-btn">
      <svg
      xmlns="http://www.w3.org/2000/svg"
      width="16"
      height="16"
      fill="currentColor"
      viewBox="0 0 16 16"
      class="mr-1"
      style="margin-right: 1em;"
    >
      <path d="M8 0C3.58 0 0 3.58 0 8a8 8 0 005.47 7.59c.4.07.55-.17.55-.38 
               0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52
               0-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95
               0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82a7.52 7.52 0 012 0c1.53-1.03 
               2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 
               1.27.82 2.15 0 3.07-1.87 3.75-3.65 
               3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 
               2.2 0 .21.15.46.55.38A8.01 8.01 0 0016 8c0-4.42-3.58-8-8-8z"/>
    </svg>
      View on GitHub
    </el-button>
  </div>
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
      size: 200,
      exportType: "PNG",
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
    openGithub() {
      window.open("https://github.com/LanLP0/qrid", "_blank");
    },
  },
  computed: {
    fileName() {
      return `ORID-${this.form.name}`;
    },
    content() {
      return `qrid=v1\n[Main]\nName=${this.form.name}\nPhoneFirst4=${this.form.phone.slice(0, 4)}\nSalt=${this.salt}\nPhoneHash=${sjcl.codec.hex.fromBits(sjcl.hash.sha256.hash(this.form.phone + this.salt))}`;
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

.credit {
  margin-top: 20px;
  text-align: center;
}

.github-btn {
  font-size: 14px;
  color: #333;
}

.github-btn:hover {
  color: #000;
}
</style>