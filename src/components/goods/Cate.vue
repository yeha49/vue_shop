<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分類</el-button>
        </el-col>
      </el-row>

      <tree-table class="treetable" :data="catelist" :columns="columns"
      :selection-type="false"
      :expand-type="false"
      :show-index=true index-text="#" :border="true" :show-row-hover="false"
      >
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="!scope.row.cat_delete" style="color: lightgreen;"></i>
          <i class="el-icon-error" v-if="scope.row.cat_delete"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level==0">一級</el-tag>
          <el-tag type="success" size="mini" v-if="scope.row.cat_level==1">二級</el-tag>
          <el-tag type="warning" size="mini" v-if="scope.row.cat_level==2">三級</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">編輯</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">刪除</el-button>
        </template>
      </tree-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>

    </el-card>

    <el-dialog
      title="添加分類"
      :visible.sync="addCatDialogVisible"
      width="50%" @close="addCateDialogClosed">
      <el-form :model="addCatForm" :rules="addCatFormRules" ref="addCateFormRef" label-width="90px" >
        <el-form-item label="分類名稱:" prop="cat_name">
          <el-input v-model="addCatForm.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父級分類:">
          <el-cascader
          v-model="selectedKeys"
          :options="parentCatList"
          :props="cascaderProps"
          @change="parentCateChanged"
          clearable >
          </el-cascader>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      catelist: [],
      total: 0,
      columns: [
        {
          label: '分類名稱',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCatDialogVisible: false,
      addCatForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCatFormRules: {
        cat_name: [
          { required: true, message: '请输入分類名稱', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
      },
      parentCatList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
        size: 'mini'
      },
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品分類失敗')
      }
      console.log(res.data.result)
      this.total = res.data.total
      this.catelist = res.data.result
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCatDialogVisible = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        this.$message.error('獲取父級分類失敗')
      }
      // console.log(res.data)
      this.parentCatList = res.data
    },
    parentCateChanged () {
      // console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        this.addCatForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCatForm.cat_level = this.selectedKeys.length
      } else {
        this.addCatForm.cat_pid = 0
        this.addCatForm.cat_level = 0
      }
    },
    addCate () {
      // console.log(this.addCatForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCatForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分類失敗')
        }
        this.$message.success('添加分類成功')
        this.getCateList()
        this.addCatDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.$refs.addCatFormRef.resetFields()
      this.selectedKeys = []
      this.addCatForm.cat_level = 0
      this.addCatForm.cat_pid = 0
    }
  }
}
</script>
<style lang="less">
.treetable{
  margin-top: 15px;
}
.el-cascader-panel{
  height: 300px!important;
}

</style>
