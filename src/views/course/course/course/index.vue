<template>
  <div class="pad20">
    <div class="filter-container">
      <el-form :inline="true" size="mini">
        <el-form-item label="课程名称:">
          <el-input v-model.trim="map.courseName"></el-input>
        </el-form-item>
        <el-form-item label="状态:" >
          <el-select v-model="map.statusId" class="auto-width" clearable filterable placeholder="状态" style="width: 85px">
            <el-option
              v-for="item in opts.statusIdList"
              :key="item.code"
              :label="item.desc"
              :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否免费:">
          <el-select v-model="map.isFree" class="auto-width" clearable filterable placeholder="是否免费" style="width: 100px">
            <el-option
              v-for="item in opts.isFreeList"
              :key="item.code"
              :label="item.desc"
              :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="上下架:">
          <el-select v-model="map.isPutaway" class="auto-width" clearable filterable placeholder="上下架" style="width: 85px">
            <el-option
              v-for="item in opts.isPutawayList"
              :key="item.code"
              :label="item.desc"
              :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button icon='el-icon-search' type="primary" @click="handleCheck">查询</el-button>
          <el-button icon='el-icon-refresh' class="filter-item" @click="handleReset">重置</el-button>
          <el-button icon="el-icon-upload" class="filter-item" @click="handleImport">一键导入</el-button>
        </el-form-item>
      </el-form>
   </div>
    <el-table v-loading="ctrl.load" size="medium" :data="list" stripe border>
      <el-table-column type="index" label="序号" width="50">
      </el-table-column>
      <el-table-column prop="courseLogo" label="课程封面" width="122">
        <template slot-scope="scope">
          <img :src="scope.row.courseLogo" :alt="scope.row.courseName" width="100">
        </template>
      </el-table-column>
      <el-table-column prop="courseName" label="课程名称" width="120">
      </el-table-column>>
      <el-table-column label="课程分类">
        <template slot-scope="scope">
          【 {{ scope.row.categoryName1 }}】>【{{ scope.row.categoryName2 }}】>【{{ scope.row.categoryName3 }}】
        </template>
      </el-table-column>
      <el-table-column label="价格" prop="courseOriginal" width="150">
        <template slot-scope="scope">
          <span v-if="scope.row.isFree === 0">原价：{{scope.row.courseOriginal.toFixed(2)}}<br>优惠价：{{scope.row.courseDiscount.toFixed(2)}}</span>
          <span v-else>免费</span>
        </template>
      </el-table-column>
      <el-table-column
        width="150"
        prop="isPutaway"
        label="上下架"
        align="center">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.isPutaway"
            @change="handleChangeIsPutaway(scope.row, $event)"
            :active-value="0"
            :inactive-value="1"
            active-color="#ff4949"
            inactive-color="#13ce66"
            active-text="下架"
            inactive-text="上架">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        width="150"
        prop="statusId"
        label="状态"
        align="center">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.statusId"
            @change="handleChangeStatusId(scope.row, $event)"
            :active-value="0"
            :inactive-value="1"
            active-color="#ff4949"
            inactive-color="#13ce66"
            active-text="禁用"
            inactive-text="正常">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="sort" label="排序" align="center" width="60">
      </el-table-column>
      <el-table-column label="操作" width="100">
        <template slot-scope="scope">
          <el-button v-has="'/course/pc/course/get'" type="success" @click="handleEdit(scope.row.id)" size="mini">修改</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      style="float: right;margin-top: 20px; margin-bottom: 22px"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :page-size="page.pageSize"
      :page-sizes="[20, 50, 100, 200, 500, 1000]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="page.totalCount">
    </el-pagination>
    <edit :visible="ctrl.editVisible" :formData="formData" :title="ctrl.dialogTitle" @close-callback="closeCllback"></edit>
  </div>
</template>
<script>
import * as api from '@/api/course'
import Edit from './edit'
import { courseEsAdd } from "../../../../api/course";
export default {
  components: { Edit },
  data() {
    return {
      ctrl: {
        load: false,
        dialogVisible: false,
        editVisible: false
      },
      map: {},
      formData: {},
      list: [],
      opts: {
        isFreeList: [],
        statusIdList: [],
        isPutawayList: []
      },
      page: {
        beginPageIndex: 1,
        currentPage: 1,
        endPageIndex: 8,
        pageSize: 20,
        totalCount: 0,
        totalPage: 0
      },
      textIsFree: {
        1: '免费',
        0: '收费'
      },
      textStatusId: {
        1: '正常',
        0: '禁用'
      }
    }
  },
  mounted() {
    this.getList()
    //this.vb = this.visible
    this.$store.dispatch('GetOpts', { enumName: "IsFreeEnum", type: 'arr' }).then(res => {
			this.opts.isFreeList = res
		})
    this.$store.dispatch('GetOpts', { enumName: "StatusIdEnum", type: 'arr' }).then(res => {
      this.opts.statusIdList = res
    })
    this.$store.dispatch('GetOpts', { enumName: "IsPutawayEnum", type: 'arr' }).then(res => {
      this.opts.isPutawayList = res
    })
  },
  methods: {
    getList() {
      this.ctrl.load = true
      api.courseList(this.map, this.page.pageCurrent, this.page.pageSize).then(res => {
        this.ctrl.load = false
        this.list = res.data.list
        this.page.pageSize = res.data.pageSize
        this.page.totalCount = res.data.totalCount
      }).catch(() => {
        this.ctrl.load = false
      })
    },
    handleCheck() {
      this.page.pageCurrent = 1
      this.getList()
    },
    // 重置查询条件
    handleReset() {
      this.reload()
    },
    // 一键导入
    handleImport() {
      courseEsAdd().then(res => {
        if (res.data) {
          this.$message.success(res.data)
          this.getList()
        }
      })
    },
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`)
      this.page.pageSize = val
      this.getList()
    },
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`)
      this.page.pageCurrent = val
      this.getList()
    },
    // 刷新当前页面
    reload() {
      this.map = {}
      this.formData = {}
      this.getList()
    },
    // 关闭编辑弹窗回调
    closeCllback() {
      this.ctrl.dialogVisible = false;
      this.ctrl.editVisible = false;
      this.reload()
    },
    // 修改上下架状态
    handleChangeIsPutaway(row, command) {
      const title = { 0: '下架', 1: '上下架' }
      this.$confirm(`确定要${title[command]}吗?`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.changeIsPutaway(row, command)
        this.reload()
      }).catch(() => {
        this.reload()
      })
    },
    // 请求更新上下架方法
    changeIsPutaway(row, command) {
      this.ctrl.load = true
      api.courseUpdate({ id: row.id, isPutaway: command }).then(res => {
        this.ctrl.load = false
        const msg = { 0: '下架成功', 1: '上架成功' }
        this.$message({
          type: 'success',
          message: msg[command]
        });
          this.reload()
      }).catch(() => {
        this.ctrl.load = false
      })
    },
    // 修改状态
    handleChangeStatusId(row, command) {
      const title = { 0: '正常', 1: '禁用' }
      this.$confirm(`确定要${title[command]}吗?`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.changeStatusId(row, command)
        this.reload()
      }).catch(() => {
        this.reload()
      })
    },
    // 请求更新状态方法
    changeStatusId(row, command) {
      this.ctrl.load = true
      api.courseUpdate({ id: row.id, statusId: command }).then(res => {
        this.ctrl.load = false
        const msg = { 0: '禁用成功', 1: '启用成功' }
        this.$message({
          type: 'success',
          message: msg[command]
        });
          this.reload()
      }).catch(() => {
        this.ctrl.load = false
      })
    },
    // 修改弹窗
    handleEdit(row) {
      this.ctrl.load = true
      api.courseViewForEdit({ id: row }).then(res => {
        this.ctrl.load = false
        this.formData = res.data
        this.ctrl.dialogTitle = "编辑"
        this.ctrl.editVisible = true
      }).catch(() => {
        this.ctrl.load = false
      })
    },
    textClass(isFree) {
      return {
        c_red: isFree === 0,
        c_blue: isFree === 1
      }
    }
  }
}
</script>
