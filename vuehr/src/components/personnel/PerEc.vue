<template>
  <div>
    <el-container>
      <el-header style="display: flex;justify-content: space-between;align-items: center;padding-left: 0px">
        <el-button @click="dialogVisible = true" icon="el-icon-plus" type="primary" size="mini">添加话题</el-button>
        <el-button size="mini" type="success" @click="loadSalaryCfg" icon="el-icon-refresh"></el-button>
      </el-header>
      <el-main style="padding-left: 0px;padding-top: 0px">
        <div>
          <el-table
            :data="salaries"
            stripe
            v-loading="tableLoading"
            size="mini"
            @selection-change="handleSelectionChange"
            style="width: 100%">
            <el-table-column
              type="selection"
              width="30">
            </el-table-column>
            <el-table-column
              width="200"
              prop="name"
              label="话题名称">
            </el-table-column>
            <el-table-column
              width="90"
              prop="collectCount"
              label="关注数">
            </el-table-column>
            <el-table-column
              width="90"
              prop="commentCount"
              label="回答数">
            </el-table-column>
            <el-table-column
              width="90"
              prop="isDeleted"
              :formatter = "stateFormat"
              label="当前状态">
            </el-table-column>
            <el-table-column
              prop="sort"
              width="90"
              label="显示顺序">
            </el-table-column>
            <el-table-column
              width="180"
              label="最后操作时间">
              <template slot-scope="scope">{{ scope.row.ut | formatDateTime}}</template>
            </el-table-column>
            <el-table-column
              prop="uP"
              :formatter = "adminFormat"
              width="90"
              label="操作人">
            </el-table-column>
            <el-table-column label="操作" align="center">
              <el-table-column label="编辑" align="center">
                <template slot-scope="scope">
                  <el-button
                    size="mini"
                    @click="handleEdit(scope.$index, scope.row)">编辑
                  </el-button>
                </template>
              </el-table-column>
              <el-table-column label="上下架" align="center">
                <template slot-scope="scope">
                  <el-button
                    size="mini"
                    type="danger"
                    @click="handleDelete(scope.$index, scope.row)">上下架
                  </el-button>
                </template>
              </el-table-column>
            </el-table-column>
          </el-table>
          <div style="text-align: right;margin-top: 10px">
            <el-pagination
              background
              @current-change="currentChange"
              layout="prev, pager, next"
              :total="totalCount">
            </el-pagination>
          </div>
        </div>
        <div style="text-align: left;margin-top: 10px" v-if="salaries!=0">
          <el-button type="danger" round size="mini" :disabled="multipleSelection.length==0" @click="deleteAll">批量上下架
          </el-button>
        </div>
      </el-main>
    </el-container>
    <div style="text-align: left">
      <el-dialog
        title="话题新增"
        :visible.sync="dialogVisible"
        width="50%"
        @close="emptySalary"
        :close-on-click-modal="false">
        <div style="display: flex;justify-content: flex-start;">
          <el-steps :active="index" direction="vertical">
            <el-step title="展示顺序" size="mini"></el-step>
            <el-step title="是否上架"></el-step>
            <el-step title="话题名称"></el-step>
            <el-step title="话题介绍"></el-step>
            <el-step title="图标地址"></el-step>
          </el-steps>
          <div style="margin-left: 30px;display: flex;justify-content: center;align-items: center;width: 80%;">
            <div v-show="index==0">
              展示顺序：
              <el-input
                placeholder="请输入展示顺序..."
                size="mini"
                style="width: 200px"
                type="number"
                v-model="salary.sort">
              </el-input>
            </div>
            <div v-show="index==1">
              是否上架：
              <template>
                <el-radio v-model="salary.isDeleted" label="0">上架</el-radio>
                <el-radio v-model="salary.isDeleted" label="1">下架</el-radio>
              </template>
            </div>
            <div v-show="index==2">
              话题名称：
              <el-input
                placeholder="请输入话题名称..."
                size="mini"
                style="width: 200px"
                type="text"
                v-model="salary.name">
              </el-input>
            </div>
            <div v-show="index==3">
              话题介绍：
              <el-input
                placeholder="请输入话题介绍..."
                style="width: 400px"
                type="textarea"
                v-model="salary.content">
              </el-input>
            </div>
            <div v-show="index==4">
              图标地址：
              <el-input
                placeholder="请输入图标地址..."
                size="mini"
                style="width: 200px"
                type="text"
                v-model="salary.icon">
              </el-input>
              <el-upload
                class="upload-demo"
                :action="actionUrl"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                :on-success="handleSuccess"
                :before-remove="beforeRemove"
                multiple
                :limit="3"
                :on-exceed="handleExceed">
<!--                :file-list="fileList">-->
                <el-button size="small" type="primary">点击上传</el-button>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
              </el-upload>
            </div>
          </div>
        </div>
        <div style="display: flex;align-items: center;justify-content: center;padding: 0px;margin: 0px;">
          <el-button round size="mini" v-if="index!=0" @click="index--">上一步</el-button>
          <el-button type="primary" round size="mini" @click="next" v-text="index==4?'完成':'下一步'"></el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        fileList: [],
        dialogVisible: false,
        tableLoading: false,
        index: 0,
        salaries: [],
        totalCount: -1,
        currentPage: 1,
        multipleSelection: [],
        salary: {
          //默认上架状态
          isDeleted: "0",
          icon:"",
          // id: '',
          // createDate: '',
          // basicSalary: '',
          // trafficSalary: '',
          // lunchSalary: '',
          // bonus: '',
          // pensionBase: '',
          // pensionPer: '',
          // medicalBase: '',
          // medicalPer: '',
          // accumulationFundBase: '',
          // accumulationFundPer: ''
        }
      };
    },
    computed: {
      actionUrl(){
        // let baseUrl = "//localhost:8082";
        let baseUrl = "";
        let resUrl = baseUrl +  "/salary/upload/upload"
        return resUrl
      }
    },
    methods: {
      handleRemove(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview(file) {
        console.log(file);
      },
      handleExceed(files, fileList) {
        this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
      },
      beforeRemove(file, fileList) {
        return this.$confirm(`确定移除 ${ file.name }？`);
      },
      handleSuccess(response, file, fileList){
        console.log(response, file, fileList)
        this.salary.icon = response.url;
      },
      stateFormat(row, column) {
        if (row.isDeleted === 0) {
          return '已上架';
        }
        else if (row.isDeleted === 1) {
          return '已下架'
        }
      },
      adminFormat(row, column) {
        if (row.up === "0") {
          return '无';
        }
        else if (row.up === "1") {
          return '管理员'
        }
      },
      deleteAll(){
        this.$confirm('上下架[' + this.multipleSelection.length + ']条记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = '';
          this.multipleSelection.forEach(row=> {
            ids = ids + row.id + ",";
          })
          this.doDelete(ids);
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消上下架'
          });
        });
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      handleEdit(index, row) {
        this.dialogVisible = true;
        row.createDate = new Date(row.createDate);
        delete row.allSalary;
        this.salary = {...row};
        this.salary.isDeleted = this.salary.isDeleted+"";
      },
      handleDelete(index, row) {
        this.$confirm('上下架[' + row.name + ']话题, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.doDelete(row.id);
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消上下架'
          });
        });
      },
      doDelete(id){
        var _this = this;
        _this.tableLoading = true;
        this.deleteRequest("/salary/sob/salary/" + id).then(resp=>{
          _this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;

            _this.loadSalaryCfg();
          }
        });
      },
      next(){
        var _this = this;
        if (this.index == 4) {
          // if(this.salary.createDate&&this.salary.basicSalary&&this.salary.trafficSalary&&this.salary.lunchSalary&&this.salary.bonus&&this.salary.pensionBase&&this.salary.pensionPer&&this.salary.medicalBase&&this.salary.medicalPer&&this.salary.accumulationFundBase&&this.salary.accumulationFundPer){

          if(this.salary.sort&&(this.salary.isDeleted.length != 0)&&this.salary.name&&this.salary.content){
            if (this.salary.id) {//更新
              _this.tableLoading = true;
              this.putRequest("/salary/sob/salary", this.salary).then(resp=> {
                _this.tableLoading = false;
                if (resp && resp.status == 200) {
                  var data = resp.data;
                  _this.dialogVisible = false;
                  _this.index = 0;
                  _this.loadSalaryCfg();
                }
              });
            } else {
                this.postRequest("/salary/sob/salary", this.salary).then(resp=> {
                  if (resp && resp.status == 200) {
                    var data = resp.data;

                    _this.dialogVisible = false;
                    _this.index = 0;
                    _this.loadSalaryCfg();
                  }
                });
            }
          }else{
            this.$message({type: 'error', message: '字段不能为空!'});
          }
        } else {
          this.index++;
        }
      },

      currentChange(currentPage){
        this.currentPage = currentPage;
        this.loadSalaryCfg();
      },

      loadSalaryCfg(){
        this.tableLoading = true;
        var _this = this;
        this.getRequest("/salary/sob/salary?page=" + this.currentPage + "&size=10").then(resp=> {
          _this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;
            _this.salaries = data.emps;
            _this.totalCount = data.count;
          }
        })
      },
      emptySalary(){
        this.salary = {
          id: '',
          createDate: '',
          basicSalary: '',
          trafficSalary: '',
          lunchSalary: '',
          bonus: '',
          pensionBase: '',
          pensionPer: '',
          medicalBase: '',
          medicalPer: '',
          accumulationFundBase: '',
          accumulationFundPer: ''
        };
        this.index = 0;
      }
    },
    mounted: function () {
      this.loadSalaryCfg();
    }
  };
</script>
<style>
  .el-dialog__body {
    padding-bottom: 10px;
  }
</style>
