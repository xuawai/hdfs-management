<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-menu"></i> 表格</el-breadcrumb-item>
                <el-breadcrumb-item>基础表格</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="handle-box">
            <!-- <el-button type="primary" icon="delete" class="handle-del mr10" @click="delAll">批量删除</el-button> -->
            <el-select v-model="select_cate" placeholder="筛选后缀" class="handle-select mr10">
                <el-option key="1" label="txt" value="txt"></el-option>
                <el-option key="2" label="" value=""></el-option>
            </el-select>
            <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
            <el-button type="primary" icon="search" @click="search">搜索</el-button>
        </div>

        <el-dialog
          title="文件内容"
          :visible.sync="dialogVisible"
          width="30%">
         <el-input type="textarea" autosize v-model="fileToBeUpdatedContent"></el-input>
          <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="handleDialogConfirm">更 新</el-button>
          </span>
        </el-dialog>

        <el-dialog
          title="文件内容"
          :visible.sync="dialogReadVisible"
          width="30%">
         <el-input type="textarea" autosize v-model="fileToBeReadContent" :disabled="true"></el-input>
          <span slot="footer" class="dialog-footer">
            <el-button type="primary" @click="handleDialogReadConfirm">确 定</el-button>
          </span>
        </el-dialog>

        <el-table :data="data" border style="width: 100%" ref="multipleTable">
            <!-- <el-table-column type="selection" width="55"></el-table-column> -->
            <el-table-column prop="fileName" label="文件名" >
            </el-table-column>
            <el-table-column prop="fileLength" label="长度(B)" >
            </el-table-column>
            <el-table-column prop="fileReplication" label="复制" >
            </el-table-column>
            <el-table-column label="操作" width="270">
                <template scope="scope">
                   <el-button size="small"
                            @click="handleRead(scope.$index, scope.row)">查看</el-button>
                    <el-button size="small"
                            @click="handleUpdate(scope.$index, scope.row)">更新</el-button>
                    <el-button size="small"
                            @click="handleDownload(scope.$index, scope.row)">下载</el-button>
                    <el-button size="small" type="danger"
                            @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- <div class="pagination">
            <el-pagination
                    @current-change ="handleCurrentChange"
                    layout="prev, pager, next"
                    :total="1000">
            </el-pagination>
        </div> -->
    </div>
</template>

<script>
    import server from '../../../config/index';
    export default {
        data() {
            return {
                tableData: [],
                cur_page: 1,
                multipleSelection: [],
                select_cate: '',
                select_word: '',
                is_search: false,
                dialogVisible: false,
                dialogReadVisible: false,
                fileToBeUpdatedContent: '',
                fileToBeUpdatedName: '',
                fileToBeReadContent: ''
            }
        },
        created(){
            this.getData('/');
        },
        computed: {
            data(){
                const self = this;
                return self.tableData.filter(function(d){
                   
                    if(d.fileName.indexOf(self.select_cate) > -1 && 
                        (d.fileName.indexOf(self.select_word) > -1)
                    ){
                        return d;
                    }
                })
            }
        },
        methods: {
            handleCurrentChange(val){
                this.cur_page = val;
                this.getData();
            },
            getData(path){
                 this.$http.get(server.url + '/webhdfs/v1' + path + '?op=LISTSTATUS&user.name=root').then(response => {

                   var pathList = response.body;
                   if(pathList!=null && pathList.FileStatuses!=null && pathList.FileStatuses.FileStatus!=null){
                       pathList = pathList.FileStatuses.FileStatus;

                       for(var i=0; i<pathList.length; i++)
                       {
                            var temp = pathList[i];
                            // console.log(temp);
                            if(temp.type == 'FILE')
                                this.tableData.push({'fileName' : path + temp.pathSuffix, 'fileLength' : temp.length, 'fileReplication' : temp.replication});
                            if(temp.type == 'DIRECTORY'){
                                var subPath = path + temp.pathSuffix + '/';
                                this.getData(subPath);
                            }

                       } 
                    }
                  


                   }, response => {
                     console.log("error");
                });
            },
            search(){
                this.is_search = true;
            },
            filterTag(value, row) {
                return row.tag === value;
            },
            handleRead(index, row) {
                this.$http.get(server.url + '/webhdfs/v1' + row.fileName + '?op=OPEN&user.name=root').then(response => {

                   console.log(row.fileName + " is read success");
                  
                   this.dialogReadVisible = true;
                   this.fileToBeReadContent = response.bodyText;

                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
               

            },
            handleUpdate(index, row) {
                this.$http.get(server.url + '/webhdfs/v1' + row.fileName + '?op=OPEN&user.name=root').then(response => {

                   console.log(row.fileName + " is read success");
                  
                   this.dialogVisible = true;
                   this.fileToBeUpdatedContent = response.bodyText;
                   this.fileToBeUpdatedName = row.fileName;

                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
               

            },
            handleDelete(index, row) {
                this.$message.error('删除第'+(index+1)+'行');
                this.$http.delete(server.url + '/webhdfs/v1' + row.fileName + '?op=DELETE&user.name=root').then(response => {

                   console.log(row.fileName + " is deleted success");
                   //刷新视图
                   for(var i=0; i<this.tableData.length; i++) {
                    if(this.tableData[i] == row) {
                      this.tableData.splice(i, 1);
                      break;
                    }
                  }

                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
                
            },
            handleDownload(index, row){
                var index1=row.fileName.lastIndexOf(".");
                var index2=row.fileName.length;
                var suffix=row.fileName.substring(index1+1,index2);
                if(!(suffix=='txt' || suffix=='json' || suffix=='csv' || suffix=='html')){
                  this.$message.info('请直接从HDFS下载文件'+(row.fileName));
                  return;
                }

                this.$message.info('下载文件'+(row.fileName));
                this.$http.get(server.url + '/webhdfs/v1' + row.fileName + '?op=OPEN&user.name=root').then(response => {

                   console.log(row.fileName + " is read success");
                   console.log(response);
                   this.downloadFile(row.fileName, response.bodyText)
                   }, response => {
                     
                     console.log("error");
                     console.log(response);
                });
            },
            downloadFile(fileName, content){

                var aLink = document.createElement('a');
                var blob = new Blob([content], {
                    type: 'image/png'
                 });
                var evt = new Event('click');
                aLink.download = fileName;
                aLink.href = URL.createObjectURL(blob);
                aLink.click();
                URL.revokeObjectURL(blob);

            },
            handleDialogReadConfirm(){
              this.dialogReadVisible = false;
            },
            handleDialogConfirm(){
         
                this.$http.put(server.url + '/webhdfs/v1' + this.fileToBeUpdatedName + '?op=CREATE&user.name=root', this.fileToBeUpdatedContent, 
                      {headers: {'content-type':'application/octet-stream'}}).then(response => {

                      this.$message.info('文件'+(this.fileToBeUpdatedName)+'更新成功');

                       this.$http.get(server.url + '/webhdfs/v1' + this.fileToBeUpdatedName + '?op=LISTSTATUS&user.name=root').then(response => {

                         var fileToBeUpdatedLength = response.body.FileStatuses.FileStatus[0].length;

                         //刷新视图
                          for(var i=0; i<this.tableData.length; i++) {
                           if(this.tableData[i].fileName == this.fileToBeUpdatedName) {
                             this.tableData[i].fileLength = fileToBeUpdatedLength;
                             break;
                           }
                         }

                         this.dialogVisible = false;

                         }, response => {
                           console.log("error");
                      });

                      

                     }, response => {
                       
                       console.log("error");
                       console.log(response);
                  });
                 
            }
        }
    }
</script>

<style scoped>
.handle-box{
    margin-bottom: 20px;
}
.handle-select{
    width: 120px;
}
.handle-input{
    width: 300px;
    display: inline-block;
}
</style>