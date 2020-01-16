/**
 * 基础菜单 商品管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>日记管理</el-breadcrumb-item>
      <el-breadcrumb-item>日记列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索筛选 -->
    <el-form :inline="true" :model="formInline" class="user-search">
      <el-form-item label="搜索：">
        <el-input size="small" v-model="formInline.keyWords" placeholder="请输入日记名称/日记内容"></el-input>
      </el-form-item>
      <el-form-item label="">
        <el-select v-model="formInline.onPublic" size="small" placeholder="请选择公开状态">
          <el-option label="全部" value=" "></el-option>
          <el-option label="公开" value="1"></el-option>
          <el-option label="私密" value="0"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">添加</el-button>
      </el-form-item>
    </el-form>
    <!--列表-->
    <el-table size="small" :data="listData" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%;">
      <el-table-column align="center" type="selection" width="60">
      </el-table-column>
      <el-table-column  prop="userName" label="用户姓名 " width="150">
      </el-table-column>
      <el-table-column  prop="diaryName" label="日记名称" width="150">
      </el-table-column>
      <el-table-column  prop="content" label="日记内容" width="200">
      </el-table-column>
      <el-table-column  prop="tagName" label="标签名称" width="200">
      </el-table-column>
      <el-table-column  prop="onPublic" label="是否公开" width="200">
      </el-table-column>
      <el-table-column  prop="createTime" label="创建时间" width="200">
      </el-table-column>

      <el-table-column align="center" label="操作" min-width="200">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="deleteUser(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>
    <!-- 编辑 或者添加 界面 -->
    <el-dialog :title="title" :visible.sync="editFormVisible" width="70%"  align="center" @click="closeDialog">
      <el-form label-width="120px" :model="editForm" :rules="rules" ref="editForm">
        <el-form-item label="日记名称" prop="deptName">
          <el-input size="small" v-model="editForm.deptName" auto-complete="off" placeholder="日记名称"></el-input>
        </el-form-item>
        <el-form-item label="日记内容" prop="deptNo">
          <el-input size="small" v-model="editForm.deptNo" auto-complete="off" placeholder="日记内容"></el-input>
        </el-form-item>
        <el-select v-model="formInline.onPublic" size="small" placeholder="请选择公开状态">
          <el-option label="全部" value=" "></el-option>
          <el-option label="公开" value="1"></el-option>
          <el-option label="私密" value="0"></el-option>
        </el-select>
        <el-form-item label="部门代码" prop="deptNo">
          <el-input size="small" v-model="editForm.deptNo" auto-complete="off" placeholder="请输入部门代码"></el-input>
        </el-form-item>
        <el-form-item label="部门名称" prop="deptName">
          <el-input size="small" v-model="editForm.deptName" auto-complete="off" placeholder="请输入部门名称"></el-input>
        </el-form-item>
        <el-form-item label="部门代码" prop="deptNo">
          <el-input size="small" v-model="editForm.deptNo" auto-complete="off" placeholder="请输入部门代码"></el-input>
        </el-form-item>
        <el-form-item label="部门名称" prop="deptName">
          <el-input size="small" v-model="editForm.deptName" auto-complete="off" placeholder="请输入部门名称"></el-input>
        </el-form-item>
        <el-form-item label="部门代码" prop="deptNo">
          <el-input size="small" v-model="editForm.deptNo" auto-complete="off" placeholder="请输入部门代码"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="closeDialog">取消</el-button>
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { diaryList, diaryEdit, diaryDelete } from '../../api/userMG'
import Pagination from '../../components/Pagination'
export default {
  data() {
    return {
      nshow: true, //switch开启
      fshow: false, //switch关闭
      loading: false, //是显示加载
      editFormVisible: false, //控制编辑页面显示与隐藏
      title: '添加',
      editForm: {
        userId: '',
        userName: '',
        content: '',
        diaryName: '',
        enabled: 1,
        imgUrl: '',
        onPublic: '',
        tagId: '',
        tagName: '',
        wxImgUrl: ''
        // token: localStorage.getItem('logintoken')
      },

      formInline: {
        pageNo: 1,
        pageSize: 10,
        keyWords: '',
        onPublic: ''
      },
      // 删除部门
      seletedata: {
        ids: '',
        token: localStorage.getItem('logintoken')
      },
      userparm: [], //搜索权限
      listData: [
      ], //用户数据
      // 分页参数
      pageparm: {
        currentPage: 1,
        pageNo: '',
        pageSize: 10,
        total: ''
      }
    }
  },
  // 注册组件
  components: {
    Pagination
  },
  /**
   * 数据发生改变
   */

  /**
   * 创建完毕
   */
  created() {
    this.getdata(this.formInline)
  },

  /**
   * 里面的方法只有被调用才会执行
   */
  methods: {
    // 获取公司列表
    getdata(parameter) {
      this.loading = true
      console.log(parameter)

      diaryList(parameter)
        .then(res => {
          this.loading = false
          if (res.code === 200) {
            this.listData = res.data.records
            // 分页赋值
            this.pageparm.pageNo = this.formInline.pageNo
            this.pageparm.pageSize = this.formInline.pageSize
            this.pageparm.total = res.data.total
          } else {
            this.$message({
              type: 'info',
              message: res.msg
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('菜单加载失败，请稍后再试！')
        })


    },
    // 分页插件事件
    callFather(parm) {
      this.formInline.pageNo= parm.currentPage
      this.formInline.pageSize = parm.pageSize
      this.getdata(this.formInline)
    },
    // 搜索事件
    search() {
      this.getdata(this.formInline)
    },
    //显示编辑界面
    handleEdit: function(index, row) {
      this.editFormVisible = true
      if (row != undefined && row != 'undefined') {
        this.title = '修改'
        this.editForm.deptId = row.deptId
        this.editForm.deptName = row.deptName
        this.editForm.deptNo = row.deptNo
      } else {
        this.title = '添加'
        this.editForm.deptId = ''
        this.editForm.deptName = ''
        this.editForm.deptNo = ''
      }
    },
    // 编辑、增加页面保存方法
    submitForm(editData) {
      this.$refs[editData].validate(valid => {
        if (valid) {
          deptSave(this.editForm)
            .then(res => {
              this.editFormVisible = false
              this.loading = false
              if (res.success) {
                this.getdata(this.formInline)
                this.$message({
                  type: 'success',
                  message: '公司保存成功！'
                })
              } else {
                this.$message({
                  type: 'info',
                  message: res.msg
                })
              }
            })
            .catch(err => {
              this.editFormVisible = false
              this.loading = false
              this.$message.error('公司保存失败，请稍后再试！')
            })
        } else {
          return false
        }
      })
    },
    // 删除公司
    deleteUser(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          deptDelete(row.deptId)
            .then(res => {
              if (res.success) {
                this.$message({
                  type: 'success',
                  message: '公司已删除!'
                })
                this.getdata(this.formInline)
              } else {
                this.$message({
                  type: 'info',
                  message: res.msg
                })
              }
            })
            .catch(err => {
              this.loading = false
              this.$message.error('公司删除失败，请稍后再试！')
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 关闭编辑、增加弹出框
    closeDialog() {
      this.editFormVisible = false
    }
  }
}
</script>

<style scoped>
.user-search {
  margin-top: 20px;
}
.userRole {
  width: 100%;
}
</style>


