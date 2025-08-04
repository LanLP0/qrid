<template>
  <el-space wrap direction="horizontal" alignment="baseline" spacer=" ">
    <el-card class="box-card">
      <template #header>
        <div class="card-header">
          <span>Thông tin</span>
        </div>
      </template>
      <el-form ref="formRef" :model="this" :rules="rules" label-width="auto" label="right">
        <el-form-item label="Họ và Tên" prop="name">
          <el-input v-model="name" clearable />
        </el-form-item>
        <el-form-item label="Số Điện Thoại" prop="phone">
          <el-input v-model="phone" clearable />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" size="large" :loading="loading" @click="saveToFile">Lưu QR<el-icon
              class="el-icon--right">
              <Download />
            </el-icon></el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <el-card class="box-card" body-style="
        background: white;
      ">
      <template #header>
        <div class="card-header">
          <span>Preview</span>
        </div>
      </template>
      <div v-loading="loading">
        <svg :width="size" :height="size" id="svgcontainer" ref="svgcontainer">
          <svg :width="size" :height="size" version="1.1" xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink">
            <svg v-html="preview"></svg>
          </svg>
        </svg>
        <div class="info-preview">
          <p><strong>{{ name }}</strong></p>
          <p>{{ phone }}</p>
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
    return {
      name: "",
      phone: "",
      size: 350,
      exportType: "PNG",
      exportSize: 1200,
      loading: false,
      rules: {
        name: [{ required: true, message: "Vui lòng nhập họ và tên", trigger: "blur" }],
        phone: [{ required: true, message: "Vui lòng nhập số điện thoại", trigger: "blur" }],
      },
    };
  },
  methods: {
    export() {
      const fileName = this.fileName;

      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      const size = this.exportSize;
      const textHeight = 60;
      const padding = 20;

      const tempImg = new Image();
      tempImg.onload = () => {
        canvas.width = size + padding * 2;
        canvas.height = size + textHeight + padding * 2;

        // Fill white background
        ctx.fillStyle = "#FFFFFF";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw QR image
        ctx.drawImage(tempImg, padding, padding, size, size);

        // Draw text below
        ctx.font = "bold 20px sans-serif";
        ctx.fillStyle = "#000";
        ctx.textAlign = "center";
        ctx.fillText(this.name, padding + size / 2, padding + size + textHeight / 2);

        ctx.font = "15px sans-serif";
        ctx.fillText(this.phone, padding + size / 2, padding + size + textHeight);

        canvas.toBlob((blob) => {
          saveAs(blob, `${fileName}.png`);
        });
      };

      tempImg.src =
        "data:image/svg+xml;charset=utf-8," +
        encodeURIComponent(this.$refs.svgcontainer.innerHTML);
    },
    saveToFile() {
      this.$refs.formRef.validate((valid) => {
        if (!valid) return;

        this.loading = true;
        [this.size, this.exportSize] = [this.exportSize, this.size];

        this.export();

        [this.size, this.exportSize] = [this.exportSize, this.size];
        setTimeout(() => (this.loading = false), 1500);
      });
    },
  },
  computed: {
    logoColor() {
      return this.inverseLogoColor ? this.backgroundColor : this.color;
    },
    logoBackgroundColor() {
      return this.inverseLogoColor ? this.color : this.backgroundColor;
    },
    fileName() {
      return `ORID-${this.name}`;
    },
    content() {
      return `qrid v1\n[Main]\nName=${this.name}\nPhone=${this.phone}`;
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
