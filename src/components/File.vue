<script setup>
import { computed, watch, reactive, ref, onMounted } from "vue";

import axios from "axios";
const filenames = ref([]);

const getFiles = () => {
  axios
    .get("http://10.250.217.14:8000/file/getfilenames")
    .then((response) => {
      if (response.data.msg == "success") {
        console.log("get filenames list success");
        filenames.value = response.data.filenames;
        console.log(filenames.value);
      }
    })
    .catch((error) => {
      console.log(error);
    })
    .finally(console.log("get filenames over"));
};

const down = reactive({
  index: -1,
  filename: "11",
});

const downfile = (index) => {
  down.index = index;
  down.filename = filenames.value[index];
  axios
    .post("http://10.250.217.14:8000/file/download", down, {
      responseType: "blob",
    })
    .then((response) => {
      console.log(response.data);

      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", down.filename);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    })
    .catch((error) => {
      console.log(error);
    })
    .finally(() => {
      console.log("download over");
    });
};

onMounted(()=>{
    getFiles()
})

</script>

<template>
  <h1 style="margin-left: 10px">文件的上传</h1>
  <el-upload drag action="http://10.250.217.14:8000/file/sendfile" style="width:auto;margin-left: 10px;margin-right: 10px;">
    <div class="el-upload__text">
      Drop file here or <em>click to upload</em> <br />
    </div>
  </el-upload>
  <h1 style="margin-left: 10px">文件的下载</h1>
  <el-button type="primary" @click="getFiles" style="margin-left: 10px">刷新文件列表</el-button>
  <li v-for="(item, index) in filenames" style="margin-top: 10px;margin-right: 10px;">
    <el-button type="success"  @click="downfile(index)">
      Download
    </el-button>
    {{ item }}
  </li>
</template>

<style scoped></style>
