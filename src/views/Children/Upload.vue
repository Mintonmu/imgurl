<template>
  <div id="upload" v-loading.fullscreen.lock="fullscreenLoading">
    <el-row>
      <el-col>
        <el-upload
          style="width: 100%"
          class="upload-demo"
          drag
          action="/"
          multiple
          :before-upload="befUpload"
        >
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        </el-upload>
      </el-col>
      <el-col>
        <div class="tag-group">
          <el-tag effect="dark">仓库：{{ upForm.repos }}</el-tag>
          <el-tag effect="dark" v-if="upForm.iscant"
            >目录：{{ upForm.delimit }}</el-tag
          >
          <el-tag effect="dark" v-else>目录：{{ upForm.content }}</el-tag>
        </div>
      </el-col>
      <el-col style="padding-top: 30px">
        <el-radio v-model="nameType" label="1">使用源文件名字</el-radio>
        <el-radio v-model="nameType" label="2">使用加密文件名字</el-radio>
      </el-col>
    </el-row>
    <el-divider></el-divider>
    <el-row>
      <el-col :span="12" class="resimg">
        <!-- <img
          src="https://raw.githubusercontent.com/wozuinbs/video/main/72d5794d54484061a9a499ca0636d965.png"
          alt=""
        /> -->
        <el-tabs v-model="activeName" type="border-card" stretch>
          <el-tab-pane label="github" name="first">
            <!-- <img
              src="https://raw.githubusercontent.com/wozuinbs/video/main/72d5794d54484061a9a499ca0636d965.png"
              alt="" -->
            <!-- /> -->
            <el-image :src="resData[0]"></el-image>
          </el-tab-pane>
          <el-tab-pane label="jsdelivr" name="second">
            <el-image :src="resData[1]"></el-image>
          </el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span="12">
        <el-input v-model="resData[0]">
          <template slot="prepend">github</template>
        </el-input>
        <el-input v-model="resData[1]">
          <template slot="prepend">jsdelivr</template>
        </el-input>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { upload } from "@/api/upload";
import md5 from "md5";
export default {
  data() {
    return {
      activeName: "first",
      fullscreenLoading: false,
      nameType: "1",
      upForm: {},
      userInfo: {},
      resUrl: "",
      resData: [
          'https://cdn.jsdelivr.net/gh/wozuinbs/video/docs/Snipaste_2021-02-02_20-44-48.png',
          'https://cdn.jsdelivr.net/gh/wozuinbs/video/docs/Snipaste_2021-02-02_20-44-48.png'
          ],
    };
  },
  created() {
    this.userInfo = this.$store.state.userInfo;
    this.upForm = this.$store.state.uploadInfo;
  },
  methods: {
    befUpload(file) {
      var reader = new FileReader();
      reader.readAsDataURL(file);
      let urlInfo = {};
      // 判断是否自定义路劲
      if (!this.upForm.iscant) {
        urlInfo = {
          name: this.userInfo.login,
          repos: this.upForm.repos,
          cont: this.upForm.content,
        };
      } else {
        urlInfo = {
          name: this.userInfo.login,
          repos: this.upForm.repos,
          cont: this.upForm.delimit,
        };
      }
      let file_last = file.name.replace(/.+\./, "");
      //   判断文件名加密方式
      let fileName = "";
      if (this.nameType == "1") {
        fileName = file.name;
      } else {
        fileName = md5(Math.random() + file.name) + "." + file_last;
      }
      if (urlInfo.cont == "/") {
        urlInfo.fileName = fileName;
      } else {
        urlInfo.fileName = "/" + fileName;
      }
      let _this = this;
      reader.onload = function () {
        let data = {
          message: "Upload pictures via wishmelz-imgurl",
          content: reader.result.split(",")[1],
        };
        _this.fullscreenLoading = true;
        upload(urlInfo, data)
          .then((res) => {
            console.log(res);
            console.log(_this.userInfo);
            console.log(_this.upForm);
            _this.resData[0] = res.content.download_url;
            _this.resData[1] = `https://cdn.jsdelivr.net/gh/${_this.userInfo.login}/${_this.upForm.repos}${_this.upForm.content}/${res.content.name}`
            _this.fullscreenLoading = false;
            _this.$message.success("上传成功");
          })
          .catch((err) => {
            console.log(err);
            _this.fullscreenLoading = false;
          });
      };
      return false;
    },
  },
};
</script>

<style>
.el-upload,
.el-upload-dragger {
  width: 100%;
}
.tag-group .el-tag {
  margin-right: 10px;
}
.resimg img {
  max-width: 100%;
  max-height: 100%;
}
</style>