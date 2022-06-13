<template>
  <div class="main">
    <input
      v-show="false"
      id="file-upload"
      type="file"
      class="upload_inp"
      accept="*"
      :multiple="'multiple'"
    />
    <el-button @click="chooseFile" type="primary">选择文件</el-button>
    <el-button
      @click="uploadFile"
      style="margin-bottom: 16px"
      type="success"
      :loading="isLoading"
      >上传文件</el-button
    >
    <div v-if="fileData.length">
      <el-row :gutter="16" v-for="(data, index) in fileData" :key="index">
        <el-col :span="16"
          >文件名称:<span>{{ data.name }}</span></el-col
        >
        <el-col :span="8"
          >文件大小:<span>{{ data.size }}b</span></el-col
        >
        {{ progressData[data.name] }}
        <div>
          上传进度:<el-progress
            :percentage="progressData[data.name]"
          ></el-progress>
        </div>
      </el-row>
    </div>
  </div>
</template>

<script>
import instance from "../../api/instance";
export default {
  name: "MutipleBase64Page",
  data() {
    return {
      inputInstance: null, // dom的实例
      isLoading: false,
      fileData: [], //保存文件
      progressData: {},
    };
  },
  mounted() {
    this.inputInstance = document.getElementById("file-upload");
    //获取选择文件的监听
    this.monitorChooseUploadFile();
  },
  methods: {
    //触发选择文件
    chooseFile() {
      //模拟点击
      this.inputInstance.click();
    },
    //将文件转换成base64
    getBase64Stream(file) {
      return new Promise((resolve) => {
        let fileReader = new FileReader();
        fileReader.readAsDataURL(file);
        fileReader.onload = (ev) => {
          resolve(ev.target.result);
        };
      });
    },
    //base64上传文件
    async uploadFile() {
      this.isLoading = true;
      let control = 0;
      this.fileData.forEach(async (data) => {
        let base64File = await this.getBase64Stream(data);
        instance
          .post(
            "/upload_single_base64",
            {
              file: encodeURIComponent(base64File),
              filename: data.name,
            },
            {
              headers: {
                "Content-Type": "application/x-www-form-urlencoded",
              },
              onUploadProgress: (event) => {
                this.progressData[data.name] =
                  parseInt(event.loaded / event.total) * 100;
              },
            }
          )
          .then(() => {
            if (control === this.fileData.length - 1) {
              this.$message.info("上传成功");
              this.isLoading = false;
            } else {
              control++;
            }
          })
          .catch((reason) => {
            alert("文件上传失败，请您稍后再试~~" + reason);
          });
      });
    },

    //获取上传文件
    monitorChooseUploadFile() {
      this.inputInstance.addEventListener("change", () => {
        let file = Array.from(this.inputInstance.files || []); //拿到文件list
        if (!file.length) return;
        if (file.some((x) => x.size > 5 * 1024 * 1024)) {
          alert("上传的文件不能超过5MB~~");
          return;
        }
        this.fileData = file.map((x) => {
          this.$set(this.progressData, x.name, 0);
          return x;
        });
      });
    },
  },
};
</script>

<style scoped>
.main {
  border: 1px dashed rgb(236, 236, 236);
  width: 500px;
  min-height: 200px;
  padding: 16px;
}
</style>
