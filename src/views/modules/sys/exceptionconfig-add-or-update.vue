<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="短编码" prop="shortCode">
      <el-input v-model="dataForm.shortCode" placeholder="短编码"></el-input>
    </el-form-item>
    <el-form-item label="完整异常编码" prop="code">
      <el-input v-model="dataForm.code" placeholder="完整异常编码"></el-input>
    </el-form-item>
    <el-form-item label="异常信息" prop="message">
      <el-input v-model="dataForm.message" placeholder="异常信息"></el-input>
    </el-form-item>
    <el-form-item label="归属模块" prop="moduleId">
      <el-input v-model="dataForm.moduleId" placeholder="归属模块"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          shortCode: '',
          code: '',
          message: '',
          moduleId: ''
        },
        dataRule: {
          shortCode: [
            { required: true, message: '短编码不能为空', trigger: 'blur' }
          ],
          code: [
            { required: true, message: '完整异常编码不能为空', trigger: 'blur' }
          ],
          message: [
            { required: true, message: '异常信息不能为空', trigger: 'blur' }
          ],
          moduleId: [
            { required: true, message: '归属模块不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/exceptionconfig/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.shortCode = data.exceptionConfig.shortCode
                this.dataForm.code = data.exceptionConfig.code
                this.dataForm.message = data.exceptionConfig.message
                this.dataForm.moduleId = data.exceptionConfig.moduleId
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/exceptionconfig/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'shortCode': this.dataForm.shortCode,
                'code': this.dataForm.code,
                'message': this.dataForm.message,
                'moduleId': this.dataForm.moduleId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
