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
    上传进度:<el-progress :percentage="processData" ></el-progress>
    <br />
    <el-link disabled>可以传任意格式的文件，支持大文件上传</el-link>
  </div>
</template>

<script>
// eslint-disable-next-line no-unused-vars
import instance from "../../api/instance";
import SparkMd5 from "spark-md5";
export default {
  name: "ChunkUploadPage",
  data() {
    return {
      inputInstance: null, // dom的实例
      isLoading: false,
      fileData: null, //保存文件
      processData:0
    };
  },
  mounted() {
    this.inputInstance = document.getElementById("file-upload");
    //获取选择文件的监听
    this.monitorChooseUploadFile();
  },
  methods: {
    chooseFile() {
      this.inputInstance.click();
    },

    //获取已经上传的分片
    getChunkData(HashData) {
      return instance.get("/upload_already", {
        params: {
          HashData,
        },
      });
    },
    //根据文件类容获取唯一的hash名称
    getHashName(file) {
      return new Promise((resolve) => {
        let fileReader = new FileReader();
        fileReader.readAsArrayBuffer(file);
        fileReader.onload = (e) => {
          let bufferData = e.target.result, //buffer数据，用于后续的分片处理
            spark = new SparkMd5.ArrayBuffer(),
            HashData = spark.append(bufferData).end(), //生成的hash名
            suffix = /\.([a-zA-Z0-9]+)$/.exec(file.name)[1]; //后缀名
          resolve({
            bufferData,
            HashData,
            suffix,
            filename: `${HashData}.${suffix}`,
          });
        };
      });
    },
    //上传文件
    async uploadFile() {
      let {HashData, suffix } = await this.getHashName(
        this.fileData
      );

      //获取服务器有没有该文件的断点分片
      let alreadUploadFile = (await this.getChunkData(HashData)).fileList || [];

      //1.确定分片策略,拿到分片的数量和分片大小
      let maxChunkDataSize = 1024 * 100; //默认最大的分片大小
      let chunkCount = Math.ceil(this.fileData.size / maxChunkDataSize); //向上取值获取分片数量
      //如果分片超过100，就可以用100个分片来重新算分片大小
      if (chunkCount > 100) {
        chunkCount = 100;
        maxChunkDataSize = this.fileData.size / chunkCount;
      }

      //设置进度条
      this.processData=parseInt(alreadUploadFile.length/chunkCount *100)
      console.log('this.processData',this.processData)

      //2.进行分片数据的获取
      let index = 0;
      let chunkFileData = [];
      while (index < chunkCount) {
        chunkFileData.push({
          file: this.fileData.slice(
            index * maxChunkDataSize,
            (index + 1) * maxChunkDataSize
          ),
          filename: `${HashData}_${index + 1}.${suffix}`,
        });
        index++;
      }

      //3. 进行分片的上传，采用promise all的原理来实现
       this.uploadChunkFile(chunkFileData, chunkCount, alreadUploadFile).then((res)=>{
         console.log(res)
      //4. 对分片进行合并
      this.mergeChunkFile(HashData, chunkCount);
       })

     
    },
    mergeChunkFile(HashData, chunkCount) {
      instance
        .post(
          "/upload_merge",
          {
            HashData,
            chunkCount,
          },
          {
            headers: {
              "Content-Type": "application/x-www-form-urlencoded",
            },
          }
        )
        .then(() => {
          this.processData=100
          this.$message.info("上传成功");
        })
        .catch((err) => {
          this.$message.error(err);
        });
    },
    uploadChunkFile(chunkFileData, chunkCount, alreadUploadFile) {
      let index = 0;
      let isComplete = (resolve) => {
        index++;
        if (index === chunkCount) {
          resolve("分片上传成功");
        }
      };
      return new Promise((resolve, reject) => {
        chunkFileData.forEach((chunkFile) => {
          if (!alreadUploadFile.includes(chunkFile.filename)) {
            let fm = new FormData();
            fm.append("file", chunkFile.file);
            fm.append("filename", chunkFile.filename);
            instance
              .post("/upload_chunk", fm)
              .then(() => {
                //进度条增加
                 this.processData++
              })
              .catch((err) => {
                reject(err);
              })
              .finally(() => {
                isComplete(resolve);
              });
          } else {
            isComplete(resolve);
          }
        });
      });
    },

    //获取上传文件
    monitorChooseUploadFile() {
      this.inputInstance.addEventListener("change", () => {
        let file = this.inputInstance.files[0];
        if (!file) return;
        //获取到文件
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
