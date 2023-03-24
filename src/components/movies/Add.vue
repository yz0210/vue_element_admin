<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>电影管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加电影</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert title="添加电影信息" type="info" center show-icon :closable="false">
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="首映时间"></el-step>
        <el-step title="电影图片"></el-step>
        <el-step title="电影分类"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- tab栏区域 -->

      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" tab-position='left' >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="电影名称" prop="name">
              <el-input v-model="addForm.name"></el-input>
            </el-form-item>
            <el-form-item label="电影导演" prop="director">
              <el-input v-model="addForm.director"></el-input>
            </el-form-item>
            <el-form-item label="电影演员" prop="actor">
              <el-input v-model="addForm.actor" ></el-input>
            </el-form-item>
            <el-form-item label="电影时长" prop="timeLength">
              <el-input v-model="addForm.timeLength" type="number"></el-input>
            </el-form-item>
            <el-form-item label="电影评分" prop="score">
              <el-input v-model="addForm.score" type="number"></el-input>
            </el-form-item>

          </el-tab-pane>
          <el-tab-pane label="首映时间" name="1">
            <el-form-item label="上映日期" prop="dateFormat">
                  <span class="demonstration">请选择首映时间</span>
                  <el-date-picker v-model="addForm.startTime" type="date" value-format=yyyy-MM-dd placeholder="选择日期"></el-date-picker>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="电影图片" name="2">
            <!-- action 表示图片要上传到的后台API地址 -->
            <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :headers="headerObj" :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="电影分类" name="3">
            <el-form-item label="电影类型" prop="movies_cat">
              <el-cascader :show-all-levels="false" filterable  clearable :options="cateList" :props="cateProps" v-model="typeStr" @change="handleChange">
              </el-cascader>
            </el-form-item>
            <el-form-item label="电影地区" prop="area">
              <el-input v-model="addForm.area" ></el-input>
            </el-form-item>
            <el-form-item label="电影语言" prop="language">
              <el-input v-model="addForm.language" ></el-input>
            </el-form-item>
            <el-button type="primary" class="btnAdd" @click="add">添加电影</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>

    </el-card>

    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      activeIndex: '0',
      // 添加商品的表单数据对象
      addForm: {
        name: 'test',
        director: 'test',
        actor: 'test',
        timeLength: 120,
        score: '10',
        area: '中国',
        language: '普通话',
        startTime: '',
        // 商品所属的分类数组
        type: '',
        // 图片的数组
        pics: []
      },
      typeStr: [],
      addFormRules: {
        name: [
          { required: true, message: '请输入电影名称', trigger: 'blur' }
        ],
        director: [
          { required: true, message: '请输入电影导演', trigger: 'blur' }
        ],
        actor: [
          { required: true, message: '请输入电影演员', trigger: 'blur' }
        ],
        timeLength: [
          { required: true, message: '请输入电影时长', trigger: 'blur' }
        ],
        score: [
          { required: true, message: '请输入电影评分', trigger: 'blur' }
        ]
      },
      // 电影类型列表
      cateList: [],
      cateProps: {
        label: 'authName',
        value: 'authName',
        children: 'children',
        multiple: true,
        emitPath: false
      },
      // 上传图片的URL地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories/tree')

      if (res.status !== 200) {
        return this.$message.error('获取电影分类数据失败！')
      }

      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联选择器选中项变化，会触发这个函数
    handleChange() {
      this.addForm.type = this.typeStr.toString()
      // console.log(this.addForm.type)
    },
    // 处理图片预览效果
    handlePreview(file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除图片的操作
    handleRemove(file) {
      // console.log(file)
      // 1. 获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2. 从 pics 数组中，找到这个图片对应的索引值
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      // 3. 调用数组的 splice 方法，把图片信息对象，从 pics 数组中移除
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    // 监听图片上传成功的事件
    handleSuccess(response) {
      console.log(response)
      // 1. 拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2. 将图片信息对象，push 到pics数组中
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    // 添加商品
    add() {
      console.log(this.addForm)
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项！')
        }
        // 执行添加的业务逻辑
        // lodash   cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.type = form.type.join(',')
        console.log(form)

        // 发起请求添加商品
        // 商品的名称，必须是唯一的
        const { data: res } = await this.$http.post('movies', form)

        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }

        this.$message.success('添加商品成功！')
        this.$router.push('/movies')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.type.length === 2) {
        return this.addForm.type[1]
      }
      return null
    }
  }

}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.previewImg {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>
