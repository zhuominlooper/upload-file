<template>
  <div class="main">
    <input
      v-show="false"
      id="file-upload"
      type="file"
      class="upload_inp"
      accept="*"

    />
    <el-button @click="chooseFile" type="primary">选择文件</el-button>
    <el-button
      @click="uploadFile"
      style="margin-bottom: 16px"
      type="success"
      :loading="isLoading"
      >上传文件</el-button
    >
    <el-row v-if="fileData" :gutter="16">
      <el-col :span="16"
        >文件名称:<span>{{ fileData.name }}</span></el-col
      >
      <el-col :span="8"
        >文件大小:<span>{{ fileData.size }}b</span></el-col
      >
    </el-row>
    <br />
    <el-link disabled
      >只能上传[.png,.jpg,.jpeg,.txt]的格式,上传的大小不能超过5MB</el-link
    >
  </div>
</template>

<script>
import instance from "../../api/instance";
export default {
    name:'Base64Page',
  data() {
    return {
      inputInstance: null, // dom的实例
      isLoading: false,
      fileData: null, //保存文件
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
     getBase64Stream(file){
          return new Promise((resolve)=>{
            let fileReader=new FileReader()
            fileReader.readAsDataURL(file);
            fileReader.onload = ev => {
                resolve(ev.target.result);
            };
          })
     },
    //base64上传文件
   async uploadFile() {
       let base64File=await this.getBase64Stream(this.fileData)
       console.log('base64File',this.fileData)
      this.isLoading = true;
      instance
        .post("/upload_single_base64", {
          file:encodeURIComponent(base64File),
          filename:this.fileData.name
        },{
          headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
        })
        .then((data) => {
          if (+data.code === 0) {
            this.$message.info('上传成功')
            return;
          }
          return Promise.reject(data.codeText);
        })
        .catch((reason) => {
          alert("文件上传失败，请您稍后再试~~" + reason);
        })
        .finally(() => {
          this.fileData=null
          this.isLoading = false;
        });
    },

    //获取上传文件
    monitorChooseUploadFile() {
      this.inputInstance.addEventListener("change", () => {
        let file = this.inputInstance.files[0];
        if (!file) return;
        // 限制文件上传的大小
        if (file.size > 5 * 1024 * 1024) {
          alert("上传的文件不能超过5MB~~");
          return;
        }
        this.fileData = file;
      });
    },
  },
};
</script>

<style scoped>
.main {
  border: 1px dashed rgb(236, 236, 236);
  width: 400px;
  height: 200px;
  padding: 16px;
}
</style>
