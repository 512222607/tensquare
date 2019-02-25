<template>
  <div>
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
    </el-table>
    <el-pagination
      @size-change="fetchData"
      @current-change="fetchData"
      :current-page="currentPage"
      :page-sizes="[5, 10, 20]"
      :page-size="10"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
  import gatheringAPI from '@/api/gathering'

  export default {
    created() {
      this.fetchData()
    },
    data() {
      return {
        list: [],
        currentPage: 1, // 当前页
        pageSize: 10, //每页大小
        searchMap: {}, //查询条件
        total: 0
      }
    },
    methods: {
      fetchData() {
        gatheringAPI.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
          this.list = response.data.rows
          this.total = response.data.total
        })
        /*gatheringAPI.getList().then(rsp => {
          this.list = rsp.data.rows
        })*/
      }
    }
  }
</script>

