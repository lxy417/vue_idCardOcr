<template>
  <div>
    <head-top></head-top>
    <el-row style="margin-top: 20px">
      <el-col :span="14" :offset="4">
        <header class="form_header">身份验证</header>
        <el-form
          :model="foodForm"
          :rules="foodrules"
          ref="foodForm"
          label-width="110px"
          class="form food_form"
        >
          <el-form-item label="姓名" prop="name">
            <el-input v-model="foodForm.name"></el-input>
          </el-form-item>
          <el-form-item label="身份证" prop="activity">
            <el-input v-model="foodForm.activity"></el-input>
          </el-form-item>
          <el-form-item label="上传身份证图片">
            <el-upload
              class="avatar-uploader"
              action="http://127.0.0.1:5000/uploadidcard"
              :show-file-list="false"
              :on-success="uploadIdCardImg"
              :before-upload="beforeImgUpload"
            >
              <img
                v-if="foodForm.image_idCard_path"
                :src="'http://localhost:5000/' + foodForm.image_idCard_path"
                class="avatar"
              />
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="上传四六级图片">
            <el-upload
              class="avatar-uploader"
              action="http://127.0.0.1:5000/upload"
              :show-file-list="false"
              :on-success="uploadCETImg"
              :before-upload="beforeImgUpload"
            >
              <img
                v-if="foodForm.image_CET_path"
                :src="'http://localhost:5000/' + foodForm.image_CET_path"
                class="avatar"
              />
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="摄像头">
            <el-button @click="showCamera">开启摄像头</el-button>
            <div id="bg-content" style="display: none">
              <div class="show-photo">
                <canvas id="myCanvas" width="400px" height="400px" />
                <video
                  id="video"
                  width="400px"
                  height="400px"
                  autoplay="autoplay"
                ></video>
                <canvas
                  id="canvas"
                  width="400px"
                  height="400px"
                  style="display: none"
                ></canvas>
                <a id="downloadA"></a>
              </div>
              <div class="take-photo">
                <el-button @click="takePhoto">拍照</el-button>
              </div>
              <div class="close-photo">
                <el-button @click="closePhoto">关闭</el-button>
              </div>
            </div>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitData('foodForm')"
              >提交验证</el-button
            >
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";
import axios from "axios";
import headTop from "@/components/headTop";
import { getCategory, addCategory, addFood } from "@/api/getData";
import { baseUrl, baseImgPath } from "@/config/env";
export default {
  data() {
    return {
      video: null,
      canvas: null,
      dir: "https://static.insta360.com/face/model",
      displaySize: {},
      timeout: null,
      faceapiReady: false,
      mediaStreamTrack: null,
      baseUrl,
      baseImgPath,
      restaurant_id: 1,
      categoryForm: {
        categoryList: [],
        categorySelect: "",
        name: "",
        description: "",
      },
      foodForm: {
        name: "",
        description: "",
        image_idCard_path: "",
        image_CET_path: "",
        image_path: "",
        activity: "",
        attributes: [],
        specs: [
          {
            specs: "默认",
            packing_fee: 0,
            price: 20,
          },
        ],
      },
      foodrules: {
        name: [{ required: true, message: "请输入姓名", trigger: "blur" }],
        activity: [
          { required: true, message: "请输入身份证", trigger: "blur" },
        ],
      },
      attributes: [
        {
          value: "新",
          label: "新品",
        },
        {
          value: "招牌",
          label: "招牌",
        },
      ],
      showAddCategory: false,
      foodSpecs: "one",
      dialogFormVisible: false,
      specsForm: {
        specs: "",
        packing_fee: 0,
        price: 20,
      },
      specsFormrules: {
        specs: [{ required: true, message: "请输入规格", trigger: "blur" }],
      },
    };
  },
  components: {
    headTop,
  },
  created() {
    // if (this.$route.query.restaurant_id) {
    //   this.restaurant_id = this.$route.query.restaurant_id;
    // } else {
    //   this.restaurant_id = Math.ceil(Math.random() * 10);
    //   this.$msgbox({
    //     title: "提示",
    //     message: "添加食品需要选择一个商铺，先去就去选择商铺吗？",
    //     showCancelButton: true,
    //     confirmButtonText: "确定",
    //     cancelButtonText: "取消",
    //     beforeClose: (action, instance, done) => {
    //       if (action === "confirm") {
    //         this.$router.push("/shopList");
    //         done();
    //       } else {
    //         this.$message({
    //           type: "info",
    //           message: "取消",
    //         });
    //         done();
    //       }
    //     },
    //   });
    // }
    this.initData();
  },
  mounted(){
    this.$nextTick(() => {
      this.initModel();
    });
  },
  computed: {
    selectValue: function () {
      return (
        this.categoryForm.categoryList[this.categoryForm.categorySelect] || {}
      );
    },
  },
  methods: {
    async initData() {
      try {
        const result = await getCategory(this.restaurant_id);
        if (result.status == 1) {
          result.category_list.map((item, index) => {
            item.value = index;
            item.label = item.name;
          });
          this.categoryForm.categoryList = result.category_list;
        } else {
          console.log(result);
        }
      } catch (err) {
        console.log(err);
      }
    },
    async initModel() {
      this.video = document.getElementById("video");
      this.canvas = document.getElementById("myCanvas");
      this.displaySize = { width: video.width, height: video.height };
      video.addEventListener("play", () => {
        console.log("play lisetner");
        // console.log(this.video)
        // this.canvas = faceapi.createCanvasFromMedia(this.video);
        // document.body.append(this.canvas);
        this.getResult();
        this.timeout = setInterval(this.getResult,1000);
      });
      // 加载模型
      await faceapi.nets.ssdMobilenetv1.loadFromUri(this.dir);
      await faceapi.nets.faceLandmark68Net.loadFromUri(this.dir);
      console.log("模型加载成功");
      this.faceapiReady = true
    },
    async detect() {
      // const options = new faceapi.TinyFaceDetectorOptions({ scoreThreshold: 0.2, inputSize: 608 });
      const options = new faceapi.SsdMobilenetv1Options({
        minConfidence: 0.5,
        maxResults: 3,
      });
      // 检测人脸
      const detections = await faceapi
        .detectAllFaces(this.video, options)
        .withFaceLandmarks();
      return detections;
    },
    draw(detections) {
      const dims = faceapi.matchDimensions(this.canvas, this.video, true);
      const resizedDetections = faceapi.resizeResults(detections, dims);
      this.canvas.getContext("2d").clearRect(0, 0, this.canvas.width, this.canvas.height);
      faceapi.draw.drawDetections(this.canvas, resizedDetections);
      faceapi.draw.drawFaceLandmarks(this.canvas, resizedDetections);
    },
    async getResult() {
      if (!this.faceapiReady) {
                return;
            }
      // console.log("aaaaaa")
      let detections = await this.detect();
      this.draw(detections);
    },
    addCategoryFun() {
      this.showAddCategory = !this.showAddCategory;
    },
    submitcategoryForm(categoryForm) {
      this.$refs[categoryForm].validate(async (valid) => {
        if (valid) {
          const params = {
            name: this.categoryForm.name,
            description: this.categoryForm.description,
            restaurant_id: this.restaurant_id,
          };
          try {
            const result = await addCategory(params);
            // console.log(result.status)
            if (result.status == 1) {
              this.initData();
              this.categoryForm.name = "";
              this.categoryForm.description = "";
              this.showAddCategory = false;
              this.$message({
                type: "success",
                message: "添加成功",
              });
            }
          } catch (err) {
            console.log(err);
          }
        } else {
          this.$notify.error({
            title: "错误",
            message: "请检查输入是否正确",
            offset: 100,
          });
          return false;
        }
      });
    },
    uploadIdCardImg(res, file) {
      if (res.status == 1) {
        this.foodForm.name = res.data.name;
        this.foodForm.activity = res.data.id;
        this.foodForm.image_idCard_path = res.image_path;
      } else {
        this.$message.error("上传图片失败！");
      }
    },
    uploadCETImg(res, file) {
      if (res.status == 1) {
        this.foodForm.image_CET_path = res.image_path;
      } else {
        this.$message.error("上传图片失败！");
      }
    },
    beforeImgUpload(file) {
      const isRightType =
        file.type === "image/jpeg" || file.type === "image/png";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isRightType) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isRightType && isLt2M;
    },
    addspecs() {
      this.foodForm.specs.push({ ...this.specsForm });
      this.specsForm.specs = "";
      this.specsForm.packing_fee = 0;
      this.specsForm.price = 20;
      this.dialogFormVisible = false;
    },
    handleDelete(index) {
      this.foodForm.specs.splice(index, 1);
    },
    tableRowClassName(row, index) {
      if (index === 1) {
        return "info-row";
      } else if (index === 3) {
        return "positive-row";
      }
      return "";
    },
    submitData(foodForm) {
      this.$refs[foodForm].validate(async (valid) => {
        if (valid) {
          await this.takePhoto();
          // console.log(this.foodForm);
          const params = {
            ...this.foodForm,
            category_id: this.selectValue.id,
            restaurant_id: this.restaurant_id,
          };
          try {
            var formData = new FormData();
            formData.append("name", this.foodForm.name);
            formData.append("idCard", this.foodForm.activity);
            formData.append("idImgPath", this.foodForm.image_idCard_path);
            formData.append("cetImgPath", this.foodForm.image_CET_path);
            formData.append("faceImgPath", this.foodForm.image_path);
            let config = {
              headers: {
                "Content-Type": "multipart/form-data",
              },
            };
            let result;
            await axios
              .post("http://127.0.0.1:5000/submit", formData, config)
              .then((response) => {
                console.log("111111111111",response.data);
                this.foodForm.image_path = response.data.image_path;
                // console.log(this.foodForm.image_path);
                result = response.data;
              });
            // const result = await addFood(params);
            if (result.status == 1) {
              console.log(result);
              this.$message({
                type: "success",
                message: "添加成功",
              });
              this.foodForm = {
                name: "",
                description: "",
                image_path: "",
                activity: "",
                attributes: [],
                specs: [
                  {
                    specs: "默认",
                    packing_fee: 0,
                    price: 20,
                  },
                ],
              };
            } else {
              this.$message({
                type: "error",
                message: result.message,
              });
            }
          } catch (err) {
            console.log(err);
          }
        } else {
          this.$notify.error({
            title: "错误",
            message: "请检查输入是否正确",
            offset: 100,
          });
          return false;
        }
      });
    },
    showCamera() {
      document.getElementById("bg-content").style = "display:block";
      // console.log(document.getElementsByClassName("bg-content")[0].style)
      let constraints = {
        video: { width: 400, height: 400 },
        audio: true,
      };
      //获得video摄像头区域
      let video = document.getElementById("video");
      //这里介绍新的方法，返回一个 Promise对象
      // 这个Promise对象返回成功后的回调函数带一个 MediaStream 对象作为其参数
      // then()是Promise对象里的方法
      // then()方法是异步执行，当then()前的方法执行完后再执行then()内部的程序
      // 避免数据没有获取到
      let promise = navigator.mediaDevices.getUserMedia(constraints);
      let that = this;
      promise.then(function (MediaStream) {
        that.mediaStreamTrack =
          typeof MediaStream.stop === "function"
            ? MediaStream
            : MediaStream.getTracks()[1];
        video.srcObject = MediaStream;
        video.muted = true;
        video.play();
      });
    },
    /**
     * 拍摄照片
     */
    async takePhoto() {
      let video = document.getElementById("video");
      let canvas = document.getElementById("canvas");
      let ctx = canvas.getContext("2d");
      ctx.drawImage(video, 0, 0);
      ctx.translate(canvas.width, 0);
      ctx.scale(-1, 1);
      let img = document.getElementById("canvas").toDataURL("image/png");
      var formData = new FormData();
      var blob = this.dataURItoBlob(img);
      formData.append("file", blob, Date.now() + ".jpg");
      let config = {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      };
      let response = await axios.post(
        "http://127.0.0.1:5000/upload",
        formData,
        config
      );
      this.foodForm.image_path = response.data.image_path;

      // var triggerDownload = $("#downloadA")
      //   .attr("href", img)
      //   .attr("download", "micro-blog.png");
      // triggerDownload[0].click();
    },

    /**
     * 关闭摄像头
     */
    closePhoto() {
      this.mediaStreamTrack.stop();
      document.getElementById("bg-content").style = "display:none";
      this.mediaStreamTrack = null;
    },
    //
    dataURItoBlob(dataURI) {
      // base64转buffer
      var byteString = atob(dataURI.split(",")[1]);
      var mimeString = dataURI.split(",")[0].split(":")[1].split(";")[0];
      var ab = new ArrayBuffer(byteString.length);
      var ia = new Uint8Array(ab);
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }
      return new Blob([ab], { type: mimeString });
    },
  },
};
</script>

<style lang="less">
@import "../style/mixin";
.form {
  min-width: 400px;
  margin-bottom: 30px;
  &:hover {
    box-shadow: 0 0 8px 0 rgba(232, 237, 250, 0.6),
      0 2px 4px 0 rgba(232, 237, 250, 0.5);
    border-radius: 6px;
    transition: all 400ms;
  }
}
.food_form {
  border: 1px solid #eaeefb;
  padding: 10px 10px 0;
}
.form_header {
  text-align: center;
  margin-bottom: 10px;
}
.category_select {
  border: 1px solid #eaeefb;
  padding: 10px 30px 10px 10px;
  border-top-right-radius: 6px;
  border-top-left-radius: 6px;
}
.add_category_row {
  height: 0;
  overflow: hidden;
  transition: all 400ms;
  background: #f9fafc;
}
.showEdit {
  height: 185px;
}
.add_category {
  background: #f9fafc;
  padding: 10px 30px 0px 10px;
  border: 1px solid #eaeefb;
  border-top: none;
}
.add_category_button {
  text-align: center;
  line-height: 40px;
  border-bottom-right-radius: 6px;
  border-bottom-left-radius: 6px;
  border: 1px solid #eaeefb;
  border-top: none;
  transition: all 400ms;
  &:hover {
    background: #f9fafc;
    span,
    .edit_icon {
      color: #20a0ff;
    }
  }
  span {
    .sc(14px, #999);
    transition: all 400ms;
  }
  .edit_icon {
    color: #ccc;
    transition: all 400ms;
  }
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #20a0ff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 120px;
  height: 120px;
  line-height: 120px;
  text-align: center;
}
.avatar {
  width: 120px;
  height: 120px;
  display: block;
}
.cell {
  text-align: center;
}
#bg-content{
  position: relative;
}
#myCanvas {
  position: absolute;
  top: 0;
  /* left: 0; */
}
</style>
