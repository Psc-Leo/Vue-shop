<template>
    <div>
      <!--面包屑导航区域-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图-->
      <el-card>
        <!--提示区-->
        <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
        </el-alert>

        <!-- 步骤条区域 -->
            <!--:active="0" 设置当前激活步骤的索引 这里接收的是数值型的数字，因为要照顾tab栏中得name值，所以这里直接减去0，转化为数值型的就行了-->
            <!--:space="200" 每个 step 的间距，不填写将自适应间距。支持百分比-->
            <!--align-center 进行居中对齐-->
            <!--finish-status="success" 设置结束步骤的状态-->
        <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>

        <!-- tab栏区域 -->
            <!--v-model="activeIndex" 这里的 activeIndex 接收的是字符串类型的数字-->
            <!--需要用el-form包裹住el-tabs  注意：el-tab-pane只能在el-tabs的内部，所以el-form在el-tabs的外面-->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
          <!--:before-leave="beforeTabLeave" 切换标签之前的钩子，若返回 false 或者返回 Promise 且被 reject，则阻止切换-->
          <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave" @tab-click="tabClicked">
            <el-tab-pane label="基本信息" name="0">
              <el-form-item label="商品名称" prop="goods_name">
                <el-input v-model="addForm.goods_name"></el-input>
              </el-form-item>
              <el-form-item label="商品价格" prop="goods_price">
                <el-input v-model="addForm.goods_price" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品重量" prop="goods_weight">
                <el-input v-model="addForm.goods_weight" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品数量" prop="goods_number">
                <el-input v-model="addForm.goods_number" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品分类" prop="goods_cat">
                <el-cascader v-model="addForm.goods_cat" :options="cateList" :props="cateProps" @change="handleChange"></el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品参数" name="1">
              <!-- 渲染表单的Item项 -->
              <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                <!-- 复选框组 -->
                <el-checkbox-group v-model="item.attr_vals">
                  <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品属性" name="2">
              <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品图片" name="3">
                  <!-- action 表示图片要上传到的后台API地址 -->
                  <!--:on-preview="handlePreview" 点击文件列表中已上传的文件时的钩子-->
                  <!--:on-remove="handleRemove" 文件列表移除文件时的钩子-->
                  <!--list-type="picture"文件列表的类型-->
                  <!--:headers="headerObj"设置上传的请求头部  因为upload没有用axios来上传，所以需要手动配置请求头-->
                  <!--on-success 文件上传成功时的钩子-->
              <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :headers="headerObj" :on-success="handleSuccess">
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
            </el-tab-pane>
            <el-tab-pane label="商品内容" name="4">
              <!-- 富文本编辑器组件 -->
              <quill-editor v-model="addForm.goods_introduce"></quill-editor>
              <!-- 添加商品的按钮 -->
              <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
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
          goods_name: '',
          goods_price: '',
          goods_weight: '',
          goods_number: '',
          // 商品所属的分类数组
          goods_cat: [],
          // 图片的数组
          pics: [],
          // 商品的详情描述
          goods_introduce: '',
          attrs: []
        },
        // 添加商品的表单的验证规则
        addFormRules: {
          goods_name: [
            { required: true, message: '请输入商品名称', trigger: 'blur' }
          ],
          goods_price: [
            { required: true, message: '请输入商品价格', trigger: 'blur' }
          ],
          goods_weight: [
            { required: true, message: '请输入商品重量', trigger: 'blur' }
          ],
          goods_number: [
            { required: true, message: '请输入商品数量', trigger: 'blur' }
          ],
          goods_cat: [
            { required: true, message: '请选择商品分类', trigger: 'blur' }
          ]
        },
        // 商品分类列表
        cateList: [],
        cateProps: {
          expandTrigger: 'hover',
          value: 'cat_id',
          label: 'cat_name',
          children: 'children'
        },

        // 动态参数列表数据
        manyTableData: [],
        // 静态属性列表数据
        onlyTableData: [],

        // / 上传图片的URL地址
        uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
        // 手动配置请求头
        headerObj: {
          Authorization: window.sessionStorage.getItem('token')
        },
        // 保存预览图片的url地址
        previewPath: '',
        // 控制预览图片对话框的显示和隐藏
        previewVisible: false
      }
    },
    created() {
      this.getCateList()
    },
    methods: {
      // 获取所有商品分类数据
      async getCateList() {
        const { data: res } = await this.$http.get('categories')
        if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败！')
        this.cateList = res.data
      },
      // 级联选择器选中项变化，会触发这个函数
      handleChange() {
        if (this.addForm.goods_cat.length !== 3) this.addForm.goods_cat = []
      },
      // 阻止切换(切换标签之前的钩子)
      beforeTabLeave(activeName, oldActiveName) {
        // console.log('即将离开的标签页名字是：' + oldActiveName)
        // console.log('即将进入的标签页名字是：' + activeName)
        // return false
        if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
          this.$message.error('请先选择商品分类！')
          return false
        }
      },

      async tabClicked() {
        // console.log(this.activeIndex)
        // 证明访问的是动态参数面板
        if (this.activeIndex === '1') {
          const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
          if (res.meta.status !== 200) return this.$message.error('获取动态参数列表失败！')
          // console.log(res.data)
          res.data.forEach(item => {
            item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
          })
          this.manyTableData = res.data
        } else if (this.activeIndex === '2') {
          const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
          if (res.meta.status !== 200) {
            return this.$message.error('获取静态属性失败！')
          }

          // console.log(res.data)
          this.onlyTableData = res.data
        }
      },
      // 处理图片预览效果
      handlePreview(file) {
        // 形参file就是用户预览的那个文件
        console.log(file)
        this.previewPath = file.response.data.url
        // 显示预览图片对话框
        this.previewVisible = true
      },
      // 处理移除图片的操作
      handleRemove(file) {
        // 形参file就是用户点击删除的文件
        // console.log(file)
        // 1. 获取将要删除的图片的临时路径
        const filePath = file.response.data.tmp_path
        // 2. 从 pics 数组中，找到这个图片对应的索引值
        const i = this.addForm.pics.findIndex(x => x.pic === filePath)
        // 3. 调用数组的 splice 方法，把图片信息对象，从 pics 数组中移除
        this.addForm.pics.splice(i, 1)
        // console.log(this.addForm)
      },
      // 监听图片上传成功的事件
      handleSuccess(response) {
        // 形参response就是上传成功之后服务器返回的结果
        // console.log(response)
        // 1. 拼接得到一个图片信息对象
        const picInfo = { pic: response.data.tmp_path }
        // 2. 将图片信息对象，push 到pics数组中
        this.addForm.pics.push(picInfo)
        // console.log(this.addForm)
      },
      // 添加商品
      add() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return this.$message.error('请填写必要的表单项')
          // 执行添加的业务逻辑
          // lodash   cloneDeep(obj)
          const form = _.cloneDeep(this.addForm)
          form.goods_cat = form.goods_cat.join(',')
          // console.log(form)
          // 处理动态参数
          this.manyTableData.forEach(item => {
            const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
            this.addForm.attrs.push(newInfo)
          })
          // 处理静态属性
          this.onlyTableData.forEach(item => {
            const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
            this.addForm.attrs.push(newInfo)
          })
          form.attrs = this.addForm.attrs
          // console.log(form)
          // 发起请求添加商品
          // 商品的名称，必须是唯一的
          const { data: res } = await this.$http.post('goods', form)
          if (res.meta.status !== 201) {
            return this.$message.error('添加商品失败！')
          }
          this.$message.success('添加商品成功！')
          this.$router.push('/goods')
        })
      }
    },
    computed: {
      cateId() {
        if (this.addForm.goods_cat.length === 3) {
          return this.addForm.goods_cat[2]
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
