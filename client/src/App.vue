<template>
  <div id="app">
      <el-tabs v-model="activeName">
    <el-tab-pane label="FormData方式上传" name="1"><FormDataComp  v-if="activeName==='1'"></FormDataComp></el-tab-pane>
    <el-tab-pane label="base64的上传方式" name="2"><base64UpdateComp  v-if="activeName==='2'"></base64UpdateComp></el-tab-pane>
    <el-tab-pane label="上传进度管理" name="3"><progressUploadComp v-if="activeName==='3'"></progressUploadComp></el-tab-pane>
    <el-tab-pane label="多文件上传" name="4"><multipleUploadComp v-if="activeName==='4'"></multipleUploadComp></el-tab-pane>
     <el-tab-pane label="大文件分片断点续传" name="5"><chunkUploadComp v-if="activeName==='5'"></chunkUploadComp></el-tab-pane>
  </el-tabs>
  
  </div>
</template>

<script>

import FormDataComp from './components/from-data-upload.vue'
import base64UpdateComp from './components/base64-upload.vue';
import progressUploadComp from './components/progress-upload.vue';
import multipleUploadComp from './components/multiple-upload.vue';
import chunkUploadComp from './components/chunk-upload.vue';
export default {
  name: "App",
  components: {
    FormDataComp,
    base64UpdateComp,
    progressUploadComp,
    multipleUploadComp,
    chunkUploadComp
  },
  data() {
    return {
      activeName:'1'
    
    };
  },
};
</script>

<style>

</style>
