e<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="權限名稱" prop="authName"></el-table-column>
        <el-table-column label="路徑" prop="path"></el-table-column>
        <el-table-column label="權限等級" prop="level">
          <template slot-scope="scope">
            <el-tag type="primary" v-if="scope.row.level =='0'">一級</el-tag>
            <el-tag type="success" v-if="scope.row.level =='1'">二級</el-tag>
            <el-tag type="warning" v-if="scope.row.level =='2'">三級</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取權限列表失敗')
      }
      this.rightsList = res.data
      console.log(res.data)
    }
  }
}
</script>
<style lang="lass" scoped>
</style>
