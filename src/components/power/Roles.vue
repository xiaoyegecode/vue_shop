<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->

    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children"
              :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限 -->
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop','vcenter' ]" v-for="(item2,i2) in item1.children"
                  :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable
                      @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>

            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClose">
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all
        :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //权限列表
      rolesList: [],
      setRightDialogVisible: false,
      //所有权限的数据
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //默认选中节点Id值数组
      defKeys: [],
      //当前即将分配权限的角色id
      roleId: '',
    }
  },
  created() {
    //获取所有的权限
    this.getRolesList()
  },
  methods: {
    //获取权限列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      //如果返回状态正常，将请求的数据保存在data中
      this.rolesList = res.data
      // console.log(this.rolesList);
    },
    //根据id删除对应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('请问是否要删除该权限', '删除提示', {
        confirmButtonText: '确认删除',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult != "confirm") {
        return this.$message.info("已经取消删除")
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg) 
      role.children = res.data

    },
    //展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.rightsList = res.data
      this.getLeafKeys(role,this.defKeys)
      this.setRightDialogVisible = true
    },
    //获取三级权限的id 保存到defKeys数组中
    getLeafKeys(node,arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => this.getLeafKeys(item,arr))
    },
    //当用户关闭树形权限对话框的时候，清除掉所有选中状态
    setRightDialogClose() {
      this.defKeys = []
    },
    //当用户在树形权限对话框中点击确定，将用户选择的权限发送请求进行更新
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getRolesList();
      this.setRightDialogVisible = false;
    }

  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>