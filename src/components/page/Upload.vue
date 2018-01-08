<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i> 表单</el-breadcrumb-item>
                <el-breadcrumb-item>文件上传</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="content-title">支持拖拽</div>
 
        
        <el-dialog
          title="文件内容"
          :visible.sync="dialogVisible"
          width="30%"
          :before-close="handleClose">
          <el-input type="textarea" autosize v-model="fileContent" :disabled="true"></el-input>
          <span slot="footer" class="dialog-footer">
            <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
          </span>
        </el-dialog>

        <el-upload
          class="upload-demo"
          action=""
          :on-change="handleChange"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :on-success="handleSuccess"
          :file-list="fileList"
          :auto-upload="false">
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击选中</em></div>
          <div class="el-upload__tip" slot="tip"></div>
        </el-upload>

        <!-- <el-button type="primary" class="handle-del mr10" @click="createFile">新建文件</el-button> -->
        <el-button type="primary" class="handle-del mr10" @click="writeFile">上传文件</el-button>
        <el-button type="primary" class="handle-del mr10" @click="readFile">读取文件</el-button>

    </div>
</template>

<script>
    import server from '../../../config/index';
    export default {
        data: function(){
            return {
                file : {},
                dialogVisible: false,
                fileContent: ''
            }
        },
        components: {
            
        },
        methods:{
            submitUpload() {
                   this.$refs.upload.submit();
                 },
            handleChange(file, fileList){
                console.log("handleProgress");
                this.file = file;
                // this.$message.info('文件'+file.name+'已选中！');
            },
             handleRemove(file, fileList) {
                console.log("handleRemove");
               console.log(file, fileList);
             },
             handlePreview(file) {
                console.log("handlePreview");
               console.log(file);
             },
             handleSuccess(file){
                this.$message.info('文件'+file.name+'已选中！');
             },
            createFile(){

                 if(typeof(this.file.name) == 'undefined'){
                    this.$message.error('未选择文件！');
                    return;
                 }
                    

                 this.$http.put(server.url + '/webhdfs/v1/' + this.file.name + '?op=CREATE&user.name=root', "", 
                    {headers: {'content-type':'application/octet-stream'}}).then(response => {

                   console.log(this.file.name + " is created success");
                    console.log(response);
                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
            },
            writeFile(){

                 if(typeof(this.file.name) == 'undefined'){
                    this.$message.error('未选择文件！');
                    return;
                 }

                 this.$http.put(server.url + '/webhdfs/v1/' + this.file.name + '?op=CREATE&user.name=root', this.file.raw, 
                    {headers: {'content-type':'application/octet-stream'}}).then(response => {

                   console.log(this.file.name + " is written success");
                   console.log(response);
                   this.$message.info('文件'+ this.file.name +'上传成功！');
                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
            },
            readFile(){

                if(typeof(this.file.name) == 'undefined'){
                    this.$message.error('未上传文件！');
                    return;
                 }

                 this.$http.get(server.url + '/webhdfs/v1/' + this.file.name + '?op=OPEN&user.name=root').then(response => {

                   console.log(this.file.name + " is read success");
                   this.fileContent = response.bodyText;

                   this.dialogVisible = true;


                   console.log(response);
                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
            }
        }
    }
</script>

<style scoped>
    .content-title{
        font-weight: 400;
        line-height: 50px;
        margin: 10px 0;
        font-size: 22px;
        color: #1f2f3d;
    }
    .pre-img{
        width:250px;
        height: 250px;
        margin-bottom: 20px;
    }
</style>