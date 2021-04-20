<template>
  <el-dialog title="上传附件" :visible.sync="dialogVisible" width="600px" :close-on-click-modal="false" @close="onClose"
             append-to-body>
    <el-upload class="upload-demo" ref="uploader" :action="apiUrl" :auto-upload="false"
               :multiple="false" :limit="1" :on-remove="handleChanged" :on-change="handleChanged"
               :on-success="handleSuccess">
      <el-button slot="trigger" size="small" type="primary" style="margin-right: 10px;">选取文件
      </el-button>
      <el-button size="small" type="success" @click="upload">上传到服务器</el-button>
      <div slot="tip" class="el-upload__tip">上传文件的大小不能超过10MB</div>
    </el-upload>
  </el-dialog>
</template>

<script>
export default {
  name: "uploader",
  props: {
    apiUrl: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      dialogVisible: false,
      fileList: [],
      displayTrigger: true
    }
  },
  methods: {
    show() {
      this.dialogVisible = true;
    },
    close() {
      this.fileList = [];
      this.displayTrigger = true;
    },
    upload() {
      if (this.fileList.length > 0) {
        this.$refs.uploader.submit();
      }
    },
    handleChanged(file, fileList) {
      this.fileList = fileList;
      this.displayTrigger = !(fileList.length > 0);
    },
    handleSuccess(response, file, fileList) {
      response.display_name = file.name;
      this.$emit("uploadSuccess", response);
      this.$nextTick(() => {
        this.dialogVisible = false;
      })
    }
  }
}
</script>

<style scoped>
.el-upload__tip {
  margin-top: 10px;
}
</style>
