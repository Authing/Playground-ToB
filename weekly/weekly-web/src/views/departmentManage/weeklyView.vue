<template>
  <div class="view-weekly">
    <el-row  v-if="userInfo.role == 2 || userInfo.role == 3">
      <div class="title">本周<span v-if="userInfo.role == 2">公司</span><span v-else>部门</span>周报概览</div>
      <p>今天：<span>{{currentDate}}</span>，<span>{{currentWeek}}</span></p>
      <p>公司<span v-if="userInfo.department_name">--部门</span>：<span>{{userInfo.company_name}}<span v-if="userInfo.department_name">--{{userInfo.department_name}}</span></span></p>
      <p>
        <label>
          <span v-if="userInfo.role == 2">公司人员({{departmentMember.length}}人)：</span>
          <span v-else>部门人员({{departmentMember.length}}人)：</span>
          <el-tag v-for="(item, index) in departmentMember" :key="index">{{item.username}}({{item.usernum}})</el-tag>
        </label>
      </p>
      <p>
        <label>未填写周报：<span class="data-style">({{unWeeklyData.length}}人)</span>
          <el-tag v-for="(item, index) in unWeeklyData" :key="index">{{item.username}}({{item.usernum}})</el-tag></label>
      </p>
      <p>
        <label>已填周报：</label>
        <span class="data-style">{{companyWeeklyListNumber}}人</span>
      </p>
      <div class="title">本周已填周报列表</div>
      <div class="search-group">
        <el-col :xs="12" :sm="12" :md="12" :lg="12" :xl="12">
          <el-col :span="16">
            <el-input placeholder="请输入内容" maxlength="20" v-model="searchContent" clearable class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="search()">查询</el-button>
            </el-input>
          </el-col>
        </el-col>
      </div>
      <el-table
        :data="weeklyTableData"
        border
        style="width: 100%">
        <el-table-column
          prop="username"
          label="姓名"
          width="120">
        </el-table-column>
        <el-table-column
          prop="usernum"
          label="工号"
          width="120">
        </el-table-column>
        <el-table-column
          label="职务"
          width="80">
          <template slot-scope="scope">
            {{scope.row.role | roleFilter}}
          </template>
        </el-table-column>
        <el-table-column
          prop="content"
          label="周报内容">
        </el-table-column>
        <el-table-column
          label="最近一次提交日期"
          width="160">
          <template slot-scope="scope">
            {{scope.row.time | dateTimeFormat}}
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="60">
          <template slot-scope="scope">
            <el-button v-if="new Date().getTime()>= scope.row.startDate && new Date().getTime()<=scope.row.endDate" @click="editClick(scope.row)" type="text" size="small">编辑</el-button>
            <span v-else>--</span>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination-box" v-if="weeklyTableData.length>0">
        <el-pagination
          background
          @current-change="handleCurrentChange"
          :current-page.sync="currentPage"
          layout="total, prev, pager, next"
          :total="weeklyListTotal">
        </el-pagination>
      </div>
    </el-row>
    <el-row v-if="userInfo.role == 1">
      <div class="title">所有周报概览</div>
      <p>今天：<span>{{currentDate}}</span>，<span>{{currentWeek}}</span></p>
      <p>
        <label>公司：<span class="data-style">({{companyList.length}}家)</span></label>
        <el-tag v-for="(item, index) in companyList" :key="index">{{item.company_name}}</el-tag>
      </p>
      <template v-for="(item, index) in weeklyDataList">
        <div class="margin-bottom">
          <div class="title">{{item.company_name || item.company_id}}-本周已填周报列表</div>
          <p>
            <label>部门人数：</label>
            <span class="data-style">{{item.companyNumber}}人</span>
          </p>
          <p>
            <label>未填写周报：<span class="data-style">({{item.unWeeklyList.length}}人)</span>
              <el-tag v-for="(item, index) in item.unWeeklyList" :key="index">{{item.username}}({{item.usernum}})</el-tag>
            </label>
          </p>
          <p>
            <label>已填周报：</label>
            <span class="data-style">{{item.companyWeeklyList.length}}人</span>
          </p>
          <div class="search-group">
            <el-col :xs="12" :sm="12" :md="12" :lg="12" :xl="12">
              <el-col :span="16">
                <el-input placeholder="请输入内容" maxlength="20" v-model="searchContentAdmin[index]" clearable class="input-with-select">
                  <el-button slot="append" icon="el-icon-search" @click="searchAdmin(item.company_id)">查询</el-button>
                </el-input>
              </el-col>
            </el-col>
          </div>
          <el-table
            :data="item.children.data"
            border
            style="width: 100%">
            <el-table-column
              prop="username"
              label="姓名"
              width="120">
            </el-table-column>
            <el-table-column
              prop="usernum"
              label="工号"
              width="120">
            </el-table-column>
            <el-table-column
              label="职务"
              width="80">
              <template slot-scope="scope">
                {{scope.row.role | roleFilter}}
              </template>
            </el-table-column>
            <el-table-column
              prop="content"
              label="周报内容">
            </el-table-column>
            <el-table-column
              label="最近一次提交日期"
              width="160">
              <template slot-scope="scope">
                {{scope.row.time | dateTimeFormat}}
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              width="60">
              <template slot-scope="scope">
                <el-button v-if="new Date().getTime()>= scope.row.startDate && new Date().getTime()<=scope.row.endDate" @click="editClick(scope.row)" type="text" size="small">编辑</el-button>
                <span v-else>--</span>
              </template>
            </el-table-column>
          </el-table>
          <div class="pagination-box" v-if="item.children.data.length>0">
            <el-pagination
              background
              @current-change="handleCurrentChange"
              :current-page.sync="currentPage"
              layout="total, prev, pager, next"
              :total="item.children.count">
            </el-pagination>
          </div>
        </div>
      </template>


    </el-row>
    <!--dialog-->
    <el-dialog
      :title="dialogTitle"
      :visible.sync="confirmSubmitVisiable"
      :before-close="handleClose"
      width="600px"
      center>
      <p>周报日期：<span>{{editWeeklyDate}}</span></p>
      <el-input
        type="textarea"
        maxlength="500"
        :autosize="{ minRows: 4, maxRows: 6}"
        placeholder="请输入内容"
        v-model="editWeeklyContent">
      </el-input>
      <span slot="footer" class="dialog-footer">
          <el-button @click="handleClose()">取 消</el-button>
          <el-button type="primary" :loading="loadingFlag" @click="successConfirm()">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  import { mapGetters, mapActions } from 'vuex';
  import { authingClient, tenantId, appId } from '../../authing/index'
  export default {
    data(){
      return {
        weeklyContent: '',
        currentDate: new Date().toLocaleDateString(),
        day: new Date().getDay(),
        weekDay:  ["星期天", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"],
        currentWeek: '',
        weeklyId: '',
        weeklyTableData: [],
        departmentMember: [],
        unWeeklyData: [],
        weeklyListTotal: 0,
        currentPage: 1,
        confirmSubmitVisiable: false,
        editWeeklyDate: '',
        editWeeklyContent: '',
        dialogTitle: '',
        loadingFlag: false,
        searchContent: '',
        roleMap: [],
        companyList: [],
        weeklyDataList: [],
        searchContentAdmin: [],
        companyWeeklyListNumber: 0
      }
    },
    created(){
      this.currentWeek = this.weekDay[this.day];
      if(this.userInfo.role == 1){
        this.allCompanyList();
        this.allCompanyWeekly()
      }else{
        /*获取部门人员列表*/
        this.departmentMemberList();
        /*获取已写周报列表*/
        this.departmentWeeklyList();
        /*获取已填周报人数，未填周报人列表*/
        this.countWeeklyList();
      }
    },
    computed: {
      ...mapGetters([
        "userInfo",
      ])
    },
    methods: {
      ...mapActions([
        "getCurrentWeekly",
        "addWeekly",
        "getDepartmentWeeklyList",
        "getDepartmentMemberListNoPage",
        "getUnDepartmentMemberList",
        "getAllCompanyList",
        "getAllCompanyWeekly"
      ]),
      formatDateTime(item){
        var date = new Date(parseInt(item));
        var y = date.getFullYear();
        var m = date.getMonth() + 1;
        m = m < 10 ? ('0' + m) : m;
        var d = date.getDate();
        d = d < 10 ? ('0' + d) : d;
        var h = date.getHours();
        h=h < 10 ? ('0' + h) : h;
        var minute = date.getMinutes();
        minute = minute < 10 ? ('0' + minute) : minute;
        var second=date.getSeconds();
        second=second < 10 ? ('0' + second) : second;
        return y + '-' + m + '-' + d+' '+h+':'+minute+':'+second;
      },
      dateFormatSpe(item){
        if(!item) return '--';
        var date = new Date(parseInt(item));
        var y = date.getFullYear();
        var m = date.getMonth() + 1;
        m = m < 10 ? ('0' + m) : m;
        var d = date.getDate();
        d = d < 10 ? ('0' + d) : d;
        var h = date.getHours();
        h=h < 10 ? ('0' + h) : h;
        var minute = date.getMinutes();
        minute = minute < 10 ? ('0' + minute) : minute;
        var second=date.getSeconds();
        second=second < 10 ? ('0' + second) : second;
        return y + '.' + m + '.' + d;
      },
      handleCurrentChange(currentPage) {
        this.queryDepartmentWeeklyList(currentPage,10)
      },
      search(){
        this.queryDepartmentWeeklyList(1, 10);
      },
      departmentWeeklyList(){
        this.queryDepartmentWeeklyList(1,10)
      },
      queryDepartmentWeeklyList(currentPage, pageSize){
        /*获取已写周报列表*/
        this.getDepartmentWeeklyList({currentPage, pageSize, searchContent: this.searchContent}).then(res => {
          if(res.errno == 0){
            this.weeklyTableData = res.data.data;
            this.weeklyListTotal = res.data.count;
            // var usernumList = this.weeklyTableData.map( item => {
            //   return {
            //     usernum: item.usernum
            //   }
            // })
            // /*获取未写周报人员列表*/
            // var params = {
            //   usernumList: usernumList
            // }
            // this.getUnDepartmentMemberList(params).then(res => {
            //   if(res.errno == 0){
            //     this.unWeeklyData = res.data.data;
            //   }else{
            //     this.$message.warning('服务器出了小差');
            //   }
            // })
          }else{
            this.$message.error('服务器出了小差');
          }
        })
      },
      departmentMemberList(){
        this.getDepartmentMemberListNoPage().then(res => {
          if(res.errno == 0){
            this.departmentMember = res.data.map( item => {
              return {
                username: item.username,
                usernum: item.usernum
              }
            });
          }else{
            this.$message.error(res.errmsg|| '服务器开小差');
          }
        })
      },
      countWeeklyList(){
        this.getUnDepartmentMemberList().then(res => {
          if (res.errno == 0) {
            this.unWeeklyData = res.data.unWeeklyList;
            this.companyWeeklyListNumber = res.data.companyWeeklyList.length;
          } else {
            this.$message.warning('服务器出了小差');
          }
        })
      },
      submitWeekly(){
        var params = {
          content: this.weeklyContent,
          date: this.currentDate,
          id:  this.weeklyId
        }
        this.addWeekly(params).then(res => {
          if(res.errno == 0){
            this.$message.success(res.errmsg|| '提交成功');
          }else{
            this.$message.error(res.errmsg|| '服务器开小差');
          }
        })
      },
      editClick(row){
        this.editWeeklyContentRow = row;
        this.confirmSubmitVisiable = true;
        this.dialogTitle = '修改周报'
        this.editWeeklyContent = this.editWeeklyContentRow.content;
        this.editWeeklyDate = this.dateFormatSpe(this.editWeeklyContentRow.startDate) + '--' + this.dateFormatSpe(this.editWeeklyContentRow.endDate);
      },
      successConfirm(){
        var params = {
          content: this.editWeeklyContent,
          date: this.currentDate,
          id: this.editWeeklyContentRow.id
        }
        if(this.editWeeklyContent){
          this.loadingFlag = true;
          this.addWeekly(params).then(res => {
            if(res.errno == 0){
              this.$message.success(res.errmsg|| '提交成功');
              this.confirmSubmitVisiable = false;
              this.editWeeklyContentRow = '';
              if(this.userInfo.role == 2 || this.userInfo.role == 3){
                this.departmrntWeeklyList();
              }else if(this.userInfo.role == 1){
                this.allCompanyWeekly()
              }
            }else{
              this.$message.error(res.errmsg|| '服务器开小差');
            }
            this.loadingFlag = false;
          })
        }else{
          this.$message.warning( '输入周报才能提交');
        }
      },
      handleClose(){
        this.confirmSubmitVisiable = false;
        this.editWeeklyContentRow = '';
      },
      /*admin-管理员*/
      allCompanyList(){
        this.getAllCompanyList({userInfo:this.userInfo,tenantId:tenantId}).then(res => {
          if(res.errno == 0){
            this.companyList = res.data;
          }else{
            this.$message.error(res.errmsg|| '服务器开小差');
          }
        })
      },
      allCompanyWeekly(){
        this.queryAdminWeeklyList(1,10)
      },
      queryAdminWeeklyList(currentPage, pageSize){
        /*获取已写周报列表*/
        this.getDepartmentWeeklyList({currentPage, pageSize, searchContent: this.searchContentAdmin,userInfo:this.userInfo}).then(res => {
          if(res.errno == 0){
            this.weeklyDataList = res.data;
          }else{
            this.$message.error('服务器出了小差');
          }
        })
      },
      searchAdmin(){
        this.queryAdminWeeklyList(1, 10);
      }
    }
  }
</script>

<style lang="postcss" scoped>
.view-weekly{
  & .data-style{
      color: #5579ee;
    }
  & .pagination-box{
      text-align: right;
      margin: 10px 0px;
    }
    & .search-group{
        margin-bottom: 10px;
        height: 40px;
      }
    & .margin-bottom{
        margin-bottom: 20px;
      }
}

</style>
