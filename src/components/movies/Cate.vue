<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>电影管理</el-breadcrumb-item>
      <el-breadcrumb-item>电影分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isOk" >
          <i class="el-icon-success"  style="color: lightgreen;"></i>
<!--          <i class="el-icon-error" v-else style="color: red;"></i>-->
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope" >
          <el-tag size="mini" v-if="scope.row.level===0" >一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.level===1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else-if="scope.row.level===2">三级</el-tag>
          <el-tag type="danger" size="mini" v-else-if="scope.row.level===3">四级</el-tag>
          <el-tag type="info" size="mini" v-else>五级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name" >
          <el-input v-model="addCateForm.label"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <!-- options 用来指定数据源 -->
          <!-- props 用来指定配置对象 -->
          <el-cascader  :options="parentCateList" :props="cateProps" v-model="selectedIdArray" @change="parentCateChanged" clearable >
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类的数据列表，默认为空
      cateList: [],
      parentCateList: [],
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'label'
        },
        {
          label: '是否有效',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isOk'
        },
        {
          label: '排序',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类的名称
        label: '',
        parentId: 0
      },
      cateId: 0, // 分类的Id
      selectedIdArray: [],
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        label: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      },
      // 指定级联选择器的配置对象
      cateProps: {
        value: 'id',
        label: 'label',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories/tree')

      if (res.status !== 200) {
        return this.$message.error('获取电影分类失败！')
      }
      // console.log(res.data)
      // 把数据列表，赋值给 cateList
      this.cateList = res.data
      this.parentCateList = res.data
      // 为总数据条数赋值
    },
    // 点击按钮，展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      // 再展示出对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    // 选择项发生变化触发这个函数
    parentCateChanged() {
      // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
      // 反之，就说明没有选中任何父级分类
      this.cateId = this.selectedIdArray[0]
      if (this.selectedIdArray.length > 0) {
        this.addCateForm.parentId = this.selectedIdArray[this.selectedIdArray.length - 1]
        if (this.selectedIdArray.length === 1) {
          this.addCateForm.parentId = 0
        }
      } else {
        this.cateId = 0
      }
    },
    // 点击按钮，添加新的分类
    addCate() {
      this.parentCateChanged()
      /*  console.log(this.selectedIdArray)
      console.log(this.addCateForm.label)
      console.log(this.cateId)
      console.log(this.addCateForm.parentId) */
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // eslint-disable-next-line no-template-curly-in-string
        const { data: res } = await this.$http.post(`categories/${this.cateId}/add`, this.addCateForm)
        if (res.status !== 200) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        await this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedIdArray = []
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}

</style>
