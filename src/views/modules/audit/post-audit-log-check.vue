<template>
  <el-dialog :visible.sync="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()" :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
          <el-form-item label="贴子ID" prop="postId">
          <el-input v-model="dataForm.postId" placeholder="贴子ID"></el-input>
      </el-form-item>
          <el-form-item label="审核员ID" prop="sysUserId">
          <el-input v-model="dataForm.sysUserId" placeholder="审核员ID"></el-input>
      </el-form-item>
          <el-form-item label="审核通过与否" prop="isPassed">
          <el-input v-model="dataForm.isPassed" placeholder="审核通过与否"></el-input>
      </el-form-item>
          <el-form-item label="结果信息" prop="resultMessage">
          <el-input v-model="dataForm.resultMessage" placeholder="结果信息"></el-input>
      </el-form-item>
        </el-form>
    <template slot="footer">
      <el-button @click="visible = false">{{ $t('cancel') }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t('confirm') }}</el-button>
    </template>
  </el-dialog>
</template>

<script>
import debounce from 'lodash/debounce'
export default {
  data () {
    return {
      visible: false,
      dataForm: {
        id: '',
        postId: '',
        sysUserId: '',
        isPassed: '',
        resultMessage: '',
        createDate: ''
      }
    }
  },
  computed: {
    dataRule () {
      return {
        postId: [
          { required: true, message: this.$t('validate.required'), trigger: 'blur' }
        ],
        sysUserId: [
          { required: true, message: this.$t('validate.required'), trigger: 'blur' }
        ],
        isPassed: [
          { required: true, message: this.$t('validate.required'), trigger: 'blur' }
        ],
        resultMessage: [
          { required: true, message: this.$t('validate.required'), trigger: 'blur' }
        ],
      }
    }
  },
  methods: {
    init () {
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.getInfo()
        }
      })
    },
    // 获取信息
    getInfo () {
      this.$http.get(`/audit/postaudit/${this.dataForm.id}`).then(({ data: res }) => {
        if (res.code !== 0) {
          return this.$message.error(res.msg)
        }
        this.dataForm = {
          ...this.dataForm,
          ...res.data
        }
      }).catch(() => {})
    },
    // 表单提交
    dataFormSubmitHandle: debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return false
        }
        this.$http[!this.dataForm.id ? 'post' : 'put']('/audit/postaudit/', this.dataForm).then(({ data: res }) => {
          if (res.code !== 0) {
            return this.$message.error(res.msg)
          }
          this.$message({
            message: this.$t('prompt.success'),
            type: 'success',
            duration: 500,
            onClose: () => {
              this.visible = false
              this.$emit('refreshDataList')
            }
          })
        }).catch(() => {})
      })
    }, 1000, { 'leading': true, 'trailing': false })
  }
}
</script>
