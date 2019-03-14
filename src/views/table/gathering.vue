<template>
  <div>
    <br/>
    <el-form :inline="true">
      <el-form-item label="活动名称">
        <el-input v-model="searchMap.name"></el-input>
      </el-form-item>
      <el-form-item label="活动日期">
        <el-date-picker
          v-model="searchMap.startTime_1"
          type="date"
          placeholder="选择开始日期">
        </el-date-picker>
        <el-date-picker
          v-model="searchMap.startTime_2"
          type="date"
          placeholder="选择结束日期">
        </el-date-picker>
      </el-form-item>
      <el-button @click="loadData" type="primary">查询</el-button>
      <el-button @click="handleEdit('')" type="primary">新增</el-button>
    </el-form>

    <el-table
      :data="list"
      border
      style="width: 100%">
      <el-table-column
        prop="id"
        label="活动id"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="活动名称"
        width="180">
      </el-table-column>
      <el-table-column
        prop="sponsor"
        label="主办方">
      </el-table-column>
      <el-table-column
        prop="address"
        label="活动地址">
      </el-table-column>
      <el-table-column
        prop="starttime"
        label="开始日期">
      </el-table-column>
      <el-table-column
        prop="endtime"
        label="结束日期">
      </el-table-column>
      <el-table-column
        fixed="right"
        label="操作"
        width="100">
        <template slot-scope="scope">
          <el-button @click="handleEdit(scope.row.id)" type="text" size="small">修改</el-button>
          <el-button @click="handleDelete(scope.row.id)" type="text" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="loadData"
      @current-change="loadData"
      :current-page="currentPage"
      :page-sizes="[5, 10, 20]"
      :page-size="10"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <el-dialog title="编辑" :visible.sync="dialogFormVisible" center>
      <span solt="body">
        <el-form label-width="80px" :model="pojo">
          <el-form-item label="活动名称">
            <el-input v-model="pojo.name" placeholder="活动名称"></el-input>
          </el-form-item>
          <el-form-item label="基本地址">
            <el-input v-model="pojo.address" placeholder="基本地址"></el-input>
          </el-form-item>
          <el-form-item label="开始日期">
            <el-date-picker v-model="pojo.starttime" type="date" placeholder="开始日期"
                            style="width: 100%;"></el-date-picker>
          </el-form-item>
          <el-form-item label="截止日期">
            <el-date-picker v-model="pojo.endtime" type="date" placeholder="截止日期" style="width: 100%;"></el-date-picker>
          </el-form-item>
          <el-form-item label="报名截止">
            <el-date-picker v-model="pojo.enrolltime" type="date" placeholder="报名截止日期"
                            style="width: 100%;"></el-date-picker>
          </el-form-item>
          <el-form-item label="活动详情">
            <el-input v-model="pojo.detail" type="textarea" placeholder="活动详情" :rows="2"></el-input>
          </el-form-item>
          <el-form-item label="选择城市">
            <el-select v-model="pojo.city" placeholder="请选择">
              <el-option
                v-for="item in cityList"
                :key="item.name"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="是否可见">
              <el-switch
                v-model="pojo.state"
                active-color="#13ce66"
                inactive-color="#ff4949"
                active-value="1"
                inactive-value="0">
              </el-switch>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="handleSave">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import gatheringAPI from '@/api/gathering'
  import cityAPI from '@/api/city'
  import message from "../../utils/message"
  import {Message} from 'element-ui'

  export default {
    created() {
      this.loadData()
      cityAPI.getList().then(request => {
        this.cityList = request.data
      })
    },
    data() {
      return {
        list: [],
        currentPage: 1, // 当前页
        pageSize: 10, //每页大小
        searchMap: {}, //查询条件
        total: 0,
        dialogFormVisible: false, //编辑窗口是否可见
        pojo: {}, //表单实体对象
        cityList: [],
        id:''//当前用户修改的id
      }
    },
    methods: {
      loadData() {
        gatheringAPI.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
          this.list = response.data.rows
          this.total = response.data.total
        }).catch(function (reason) {
          console.log('catch:', reason);
        })
        /*gatheringAPI.getList().then(rsp => {
          this.list = rsp.data.rows
        })*/
      },
      submitForm() {
        console.log(this.pojo)
        message.handlerShowMessage(cityAPI.getList(), this)
      },
      handleSave() {
          gatheringAPI.update(this.id,this.pojo).then(response => {
            this.$message({
              message: response.message,
              type: (response.flag?'success':'error')
            })
            if (response.flag) {
              this.loadData()
              this.dialogFormVisible = false
            }
          })
      },
      handleEdit(id) {
        this.dialogFormVisible = true
        this.id = id
        if (id != '') {
          gatheringAPI.findById(id).then(resp => {
            if (resp.flag) {
              this.pojo = resp.data
            }
          })
        } else {
          this.pojo = {}
        }
      },
      handleDelete(id){
        this.$confirm('确认删除？','提示',{
          confirmButtonText:'确定',
          cancelButtonText:'取消',
          type:'warning'
        }).then(() => {
          gatheringAPI.deleteById(id).then(response => {
            if (response.flag){
              Message.success(response.message)
              this.loadData()
            }else{
              message.error(response.message)
            }
          })
        })
      }
    }
  }
</script>

<style></style>
