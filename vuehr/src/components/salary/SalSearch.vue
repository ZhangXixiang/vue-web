<!--评论列表-->
<template>
  <div>
    <el-container>
      <el-header style="display: flex;justify-content: space-between;align-items: center;padding-left: 0px">
        <!--        <el-button @click="dialogVisible = true" icon="el-icon-plus" type="primary" size="mini">添加账套</el-button>-->
        <el-form :inline="true" :model="formInline" class="demo-form-inline">
          <el-form-item label="评论内容">
            <el-input v-model="formInline.name" placeholder="评论内容" size="mini"  width="30"></el-input>
          </el-form-item>
          <el-form-item label="所属回答">
            <el-input v-model="formInline.answer" placeholder="所属回答" size="mini"  width="30"></el-input>
          </el-form-item>
          <el-form-item label="所属话题">
            <el-input v-model="formInline.topic" placeholder="所属话题" size="mini"  width="30"></el-input>
          </el-form-item>
        <el-button size="mini" type="success" @click="loadSalaryCfg" icon="el-icon-refresh"></el-button>
        </el-form>
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
              prop="content"
              label="评论内容">
            </el-table-column>
            <el-table-column
              width="140"
              prop="answerTitle"
              label="所属回答">
            </el-table-column>
            <el-table-column
              width="140"
              prop="topicName"
              label="所属话题">
            </el-table-column>
            <el-table-column
              width="140"
              label="发布时间">
              <template slot-scope="scope">{{ scope.row.createTime | formatDateTime}}</template>
            </el-table-column>
            <el-table-column
              width="140"
              prop="userName"
              label="发布人">
            </el-table-column>
            <el-table-column
              prop="isDeleted"
              :formatter = "stateFormat"
              width="70"
              label="当前状态">
            </el-table-column>
            <el-table-column
              width="140"
              label="最后操作时间">
              <template slot-scope="scope">{{ scope.row.ut | formatDateTime}}</template>
            </el-table-column>
            <el-table-column
              prop="up"
              :formatter = "adminFormat"
              width="70"
              label="操作人">
            </el-table-column>

            <el-table-column label="操作" align="center">
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
<!--    <div style="text-align: left">-->
<!--      <el-dialog-->
<!--        title="添加工资账套"-->
<!--        :visible.sync="dialogVisible"-->
<!--        width="50%"-->
<!--        @close="emptySalary"-->
<!--        :close-on-click-modal="false">-->
<!--        <div style="display: flex;justify-content: flex-start;">-->
<!--          <el-steps :active="index" direction="vertical">-->
<!--            <el-step title="基本工资" size="mini"></el-step>-->
<!--            <el-step title="交通补助"></el-step>-->
<!--            <el-step title="午餐补助"></el-step>-->
<!--            <el-step title="奖金"></el-step>-->
<!--            <el-step title="养老金"></el-step>-->
<!--            <el-step title="医疗保险"></el-step>-->
<!--            <el-step title="公积金"></el-step>-->
<!--            <el-step title="启用时间"></el-step>-->
<!--          </el-steps>-->
<!--          <div style="margin-left: 30px;display: flex;justify-content: center;align-items: center;width: 80%;">-->
<!--            <div v-show="index==0">-->
<!--              基本工资：-->
<!--              <el-input-->
<!--                placeholder="请输入基本工资..."-->
<!--                size="mini"-->
<!--                style="width: 200px"-->
<!--                type="number"-->
<!--                @keyup.enter.native="next"-->
<!--                v-model="salary.basicSalary">-->
<!--              </el-input>-->
<!--            </div>-->
<!--            <div v-show="index==1">-->
<!--              交通补助：-->
<!--              <el-input-->
<!--                placeholder="请输入交通补助..."-->
<!--                @keyup.enter.native="next"-->
<!--                size="mini"-->
<!--                style="width: 200px"-->
<!--                type="number"-->
<!--                v-model="salary.trafficSalary">-->
<!--              </el-input>-->
<!--            </div>-->
<!--            <div v-show="index==2">-->
<!--              午餐补助：-->
<!--              <el-input-->
<!--                placeholder="请输入午餐补助..."-->
<!--                @keyup.enter.native="next"-->
<!--                size="mini"-->
<!--                style="width: 200px"-->
<!--                type="number"-->
<!--                v-model="salary.lunchSalary">-->
<!--              </el-input>-->
<!--            </div>-->
<!--            <div v-show="index==3">-->
<!--              奖金：-->
<!--              <el-input-->
<!--                placeholder="请输入奖金..."-->
<!--                size="mini"-->
<!--                @keyup.enter.native="next"-->
<!--                style="width: 200px"-->
<!--                type="number"-->
<!--                v-model="salary.bonus">-->
<!--              </el-input>-->
<!--            </div>-->
<!--            <div v-show="index==4">-->
<!--              <div>-->
<!--                养老金比率：-->
<!--                <el-input-->
<!--                  placeholder="请输入养老金比率..."-->
<!--                  size="mini"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.pensionPer">-->
<!--                </el-input>-->
<!--              </div>-->
<!--              <div style="margin-top: 10px">-->
<!--                养老金基数：-->
<!--                <el-input-->
<!--                  placeholder="请输入养老金基数..."-->
<!--                  size="mini"-->
<!--                  @keyup.enter.native="next"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.pensionBase">-->
<!--                </el-input>-->
<!--              </div>-->
<!--            </div>-->
<!--            <div v-show="index==5">-->
<!--              <div>-->
<!--                医疗保险比率：-->
<!--                <el-input-->
<!--                  placeholder="请输入医疗保险比率..."-->
<!--                  size="mini"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.medicalPer">-->
<!--                </el-input>-->
<!--              </div>-->
<!--              <div style="margin-top: 10px">-->
<!--                医疗保险基数：-->
<!--                <el-input-->
<!--                  placeholder="请输入医疗保险基数..."-->
<!--                  size="mini"-->
<!--                  @keyup.enter.native="next"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.medicalBase">-->
<!--                </el-input>-->
<!--              </div>-->
<!--            </div>-->
<!--            <div v-show="index==6">-->
<!--              <div>-->
<!--                公积金比率：-->
<!--                <el-input-->
<!--                  placeholder="请输入公积金比率..."-->
<!--                  size="mini"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.accumulationFundPer">-->
<!--                </el-input>-->
<!--              </div>-->
<!--              <div style="margin-top: 10px">-->
<!--                公积金基数：-->
<!--                <el-input-->
<!--                  placeholder="请输入公积金基数..."-->
<!--                  @keyup.enter.native="next"-->
<!--                  size="mini"-->
<!--                  style="width: 200px"-->
<!--                  type="number"-->
<!--                  v-model="salary.accumulationFundBase">-->
<!--                </el-input>-->
<!--              </div>-->
<!--            </div>-->
<!--            <div v-show="index==7">-->
<!--              启用时间：-->
<!--              <el-date-picker-->
<!--                v-model="salary.createDate"-->
<!--                type="date"-->
<!--                size="mini"-->
<!--                value-format="yyyy-MM-dd"-->
<!--                style="width: 200px;"-->
<!--                placeholder="选择日期">-->
<!--              </el-date-picker>-->
<!--            </div>-->
<!--          </div>-->
<!--        </div>-->
<!--        <div style="display: flex;align-items: center;justify-content: center;padding: 0px;margin: 0px;">-->
<!--          <el-button round size="mini" v-if="index!=0" @click="index&#45;&#45;">上一步</el-button>-->
<!--          <el-button type="primary" round size="mini" @click="next" v-text="index==7?'完成':'下一步'"></el-button>-->
<!--        </div>-->
<!--      </el-dialog>-->
<!--    </div>-->
  </div>
</template>
<script>
  export default {
    data() {
      return {
        dialogVisible: false,
        tableLoading: false,
        index: 0,
        salaries: [],
        multipleSelection: [],
        totalCount: -1,
        currentPage: 1,
        formInline: {
          name: '',
          answer: '',
          topic: ''
        },
        // salary: {
        //   id: '',
        //   createDate: '',
        //   basicSalary: '',
        //   trafficSalary: '',
        //   lunchSalary: '',
        //   bonus: '',
        //   pensionBase: '',
        //   pensionPer: '',
        //   medicalBase: '',
        //   medicalPer: '',
        //   accumulationFundBase: '',
        //   accumulationFundPer: ''
        // }
      };
    },
    methods: {
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
            ids = ids + row.commentId + ",";
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
        this.salary = row;
      },
      handleDelete(index, row) {
        this.$confirm('上下架[' + row.content + ']回答, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.doDelete(row.commentId);
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
        this.deleteRequest("/salary/sob/comment/" + id).then(resp=>{
          _this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;

            _this.loadSalaryCfg();
          }
        });
      },
      // next(){
      //   var _this = this;
      //   if (this.index == 7) {
      //     if(this.salary.createDate&&this.salary.basicSalary&&this.salary.trafficSalary&&this.salary.lunchSalary&&this.salary.bonus&&this.salary.pensionBase&&this.salary.pensionPer&&this.salary.medicalBase&&this.salary.medicalPer&&this.salary.accumulationFundBase&&this.salary.accumulationFundPer){
      //       if (this.salary.id) {//更新
      //         _this.tableLoading = true;
      //         this.putRequest("/salary/sob/salary", this.salary).then(resp=> {
      //           _this.tableLoading = false;
      //           if (resp && resp.status == 200) {
      //             var data = resp.data;
      //             _
      //             _this.dialogVisible = false;
      //             _this.index = 0;
      //             _this.loadSalaryCfg();
      //           }
      //         });
      //       } else {//添加
      //         this.$prompt('请输入账套名称', '提示', {
      //           confirmButtonText: '确定',
      //           cancelButtonText: '取消'
      //         }).then(({value}) => {
      //           this.salary.name = value;
      //           this.postRequest("/salary/sob/salary", this.salary).then(resp=> {
      //             if (resp && resp.status == 200) {
      //               var data = resp.data;
      //
      //               _this.dialogVisible = false;
      //               _this.index = 0;
      //               _this.loadSalaryCfg();
      //             }
      //           });
      //         }).catch(() => {
      //         });
      //       }
      //     }else{
      //       this.$message({type: 'error', message: '字段不能为空!'});
      //     }
      //   } else {
      //     this.index++;
      //   }
      // },
      currentChange(currentPage){
        this.currentPage = currentPage;
        this.loadSalaryCfg();
      },
      loadSalaryCfg(){
        this.tableLoading = true;
        var _this = this;
        this.getRequest("/salary/sob/comment?page=" + this.currentPage + "&size=10" + "&name="+this.formInline.name+ "&topic="+this.formInline.topic+ "&answer="+this.formInline.answer).then(resp=> {
          _this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;
            _this.salaries = data.emps;
            _this.totalCount = data.count;
            // _this.salaries = resp.data;
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
