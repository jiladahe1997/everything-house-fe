<template>
  <div>

<van-row class="bg" type="flex" justify="center">
  <van-col span="4" display:inline-block>
 <van-image
  width="7.5rem"
  height="3rem"
  :src = logo
  class="logo"
/>
</van-col>

  <van-col span="8" display:inline-block>
<div class="logotext" >创作中心</div>
</van-col>
  
  <van-col class="useravatar" span="12">
    <van-image 
    class="avatar"
    round
    width="2.6rem"
    height="2.6rem"
    fit="fill"
    :src="useravatar"
/></van-col>
</van-row>

    <el-row>
  <el-col :span="6"><br></el-col>
  <el-col :span="12">
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm" label-position="top">
      <el-form-item class="nameinput" label="视频名称" prop="name">
        <el-input v-model="ruleForm.name"></el-input>
      </el-form-item>

      <el-form-item class="catagoryinput" label="视频类别" prop="videoCatagory">
        <el-select v-model="ruleForm.videoCatagory" placeholder="请选择视频类别">
          <el-option label="彩虹六号" value="0"></el-option>
          <el-option label="战地5" value="1"></el-option>
          <el-option label="CSGO" value="2"></el-option>
          <el-option label="云顶之奕" value="3"></el-option>
          <el-option label="日常生活" value="4"></el-option>
          <el-option label="鬼畜搞笑" value="5"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item class="introductioninput" label="视频简介" prop="introduction">
        <el-input type="textarea" v-model="ruleForm.introduction"></el-input>
      </el-form-item>


            <el-form-item label="图片封面" prop="imgUrl">
          <el-upload
            :limit="1"
            class="upload-demo"
            drag
            action=""
            :http-request="imgUpload"
            :before-upload="beforeImgUpload">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将视频封面图片拖到此处，或点击上传</div>
            <div class="el-upload__tip" slot="tip">只能上传jpg/png文件</div>
          </el-upload>
          <el-progress class="progress" :text-inside="true" :stroke-width="20" :percentage="imgPercentage"></el-progress>
          </el-form-item>

        <el-form-item label="视频上传" prop="videoUrl">
          <el-upload
            :limit="1"
            class="upload-demo"
            drag
            action=""
            :http-request="videoUpload"
            :before-upload="beforeVideoUpload">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将视频拖到此处，或点击上传</div>
            <div class="el-upload__tip" slot="tip">只能上传mp4文件</div>
          </el-upload>
          <el-progress class="progress" :text-inside="true" :stroke-width="20" :percentage="videoPercentage"></el-progress>
        </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">点击上传</el-button>
      </el-form-item>
    </el-form>

  </el-col>
  <el-col :span="6"><br></el-col>
</el-row>



  </div>
</template>

<script>
    var COS = require('cos-js-sdk-v5');
    import axios from 'axios'
    import { Loading } from 'element-ui';
        var cos = new COS({
        getAuthorization: function (options,callback) {
            axios.get('/api/TencentCloudKey'+'?bucket='+options.Bucket+'&region='+options.Region)   
               .then(function (response) {     
                    callback({
                        TmpSecretId: response.data.data.tmpSecretId,
                        TmpSecretKey: response.data.data.tmpSecretKey,
                        XCosSecurityToken: response.data.data.sessionToken,
                        // 建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
                        StartTime: response.data.data.startTime, // 单位是秒
                        ExpiredTime: response.data.data.expiredTime
                    });
                })
                console.log("获取签名")
        }
    });
    export default {
        name: 'videoUpload',
        data () {
            return {
                usernickname: '',
                useravatar: '',
                logo:require("../assets/logo.png"),
                videoPercentage:0,
                imgPercentage:0,
                ruleForm: {
                    name: '',
                    videoCatagory: '',
                    introduction: '',
                    videoUrl:"",
                    imgUrl:"",
                    videoKey:"",
                    imgKey:"",
                },
                rules: {
                    name: [
                        { required: true, message: '请输入视频名称', trigger: 'blur' },
                        { min: 1, max: 30, message: '长度在 1 到 30 个字符', trigger: 'blur' }
                    ],
                    videoCatagory: [
                        { required: true, message: '请选择视频类别', trigger: 'change' }
                    ],

                    introduction: [
                        { required: true, message: '请填写视频简介', trigger: 'blur' },
                        { min: 1, max: 500, message: '长度在 1 到 500 个字符', trigger: 'blur' }
                    ],

                    videoUrl: [
                        { required: true, message: '请上传视频', trigger: 'change' }
                    ],
                    imgUrl: [
                        { required: true, message: '请上传视频封面', trigger: 'change' }
                    ],
                }
            };
        },
          async created() {
         // 向服务器请求id和头像
        const res = await this.axios.get('/api/getUserInfo')
        this.usernickname = res.data.nickname
        this.useravatar = res.data.figureurl
       },
        methods:{
            // videoRemove(){
            //     var that=this;
            //     cos.deleteObject({
            //     Bucket: 'sls-cloudfunction-ap-guangzhou-code-1256609098', /* 必须 */
            //     Region: 'ap-guangzhou',     /* 存储桶所在地域，必须字段 */
            //     Key: that.videoKey,        /* 必须 */
            //     }, function(err, data) {
            //     console.log(that.videoKey);
            //     console.log(err || data);
            //    });
            // },
            // imgRemove(){
            //     var that=this;
            //     cos.deleteObject({
            //     Bucket: 'sls-cloudfunction-ap-guangzhou-code-1256609098', /* 必须 */
            //     Region: 'ap-guangzhou',     /* 存储桶所在地域，必须字段 */
            //     Key: that.imgKey,        /* 必须 */
            //     }, function(err, data) {
            //     console.log(that.imgKey);
            //     console.log(err || data);
            //    });
            // },
            beforeImgUpload (file) {
                var imgName = file.name;
                var idx = imgName.lastIndexOf(".");
                if (idx != -1){
                    var ext = imgName.substr(idx+1).toUpperCase();
                    ext = ext.toLowerCase( );
                    if (ext!='jpg' && ext!='png'){
                        this.$message.error('只能上传jpg/png文件')
                        return false
                    }else{
                        return true;
                    }
                }else{
                    this.$message.error('只能上传jpg/png文件')
                    return false
                }
            },
            imgUpload(item){
                var that=this;
                this.imgKey='temp/'+Number(Math.random().toString().substr(3, 3) + Date.now()).toString(36)+item.file.name;
                cos.putObject({
                    Bucket: 'sls-cloudfunction-ap-guangzhou-code-1256609098', /* 必须 */
                    Region: 'ap-guangzhou',     /* 存储桶所在地域，必须字段 */
                    Key: that.imgKey,              /* 必须 */
                    StorageClass: 'STANDARD',
                    Body: item.file, // 上传文件对象
                    onProgress: function(progressData) {
                        that.imgPercentage=progressData.percent*100;
                    }
                }, function(err, data) {
                    that.ruleForm.imgUrl=data.Location;
                    console.log(data);
                });
            },

            beforeVideoUpload (file) {
                var videoName = file.name;
                var idx = videoName.lastIndexOf(".");
                if (idx != -1){
                    var ext = videoName.substr(idx+1).toUpperCase();
                    ext = ext.toLowerCase( );
                    if (ext!='mp4'){
                        this.$message.error('只能上传mp4文件')
                        return false
                    }else{
                        return true;
                    }
                }else{
                    this.$message.error('只能上传mp4文件')
                    return false
                }
                },
            videoUpload(item){
                var that=this;
                this.videoKey='temp/'+Number(Math.random().toString().substr(3, 3) + Date.now()).toString(36)+item.file.name;
                cos.putObject({
                    Bucket: 'sls-cloudfunction-ap-guangzhou-code-1256609098', /* 必须 */
                    Region: 'ap-guangzhou',     /* 存储桶所在地域，必须字段 */
                    Key: that.videoKey,              /* 必须 */
                    StorageClass: 'STANDARD',
                    Body: item.file, // 上传文件对象
                    onProgress: function(progressData) {
                        that.videoPercentage=progressData.percent*100;
                    }
                }, function(err, data) {
                    that.ruleForm.videoUrl=data.Location;
                });
                },
            submitForm(formName) {
                var that =this;
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        let loadingInstance = Loading.service({fullscreen:true});
                        axios.post('/api/videoUpload',{
                            name:that.ruleForm.name,
                            videoCatagory:that.ruleForm.videoCatagory,
                            introduction: that.ruleForm.introduction,
                            videoUrl:that.ruleForm.videoUrl,
                            imgUrl: that.ruleForm.imgUrl
                        })
                            .then(function () {
                                loadingInstance.close();
                                that.$message.success('提交成功,回到首页')
                                window.location.href="//jiladahe1997.cn";
                            })
                            .catch(function (error) {
                                that.$message.error("提交失败，错误代码"+error)
                            });
                    } else {
                        return false;
                    }
                });
            },
        }
    }
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.nameinput{
    margin-top: 3rem;
    width:30rem;
}
.catagoryinput{
    width:16rem;
}
.introductioninput{
    width:45rem;
    height: 10rem;
}
.progress{
    width: 40rem;
    vertical-align: 0%;
}
.title{
    color:#c2b82c;
    font-weight:bold;
    font-size: 32px;
}

.bg{
    background-color: #EBEBEB
}
.logo{
    margin-top: 0.3rem;
    margin-left: 0.5rem;
}
.useravatar{
    text-align: right;
}
.avatar{
    margin-top: 0.5rem;
    margin-right: 0.5rem;
}
.logotext{
    font-size: 2rem;
    font-weight: bold;
    color: #dbd79d;
    margin-top: 0.7rem;
}
</style>
