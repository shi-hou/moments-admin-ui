<template>
  <el-dialog :visible.sync="visible" title="审核" :close-on-click-modal="false"
             :close-on-press-escape="false">
    <el-divider>内容</el-divider>
    <ren9-box-grid :urls="dataForm.imageUrls"/>
    <el-descriptions :column="1">
      <el-descriptions-item label="地点">{{ dataForm.address }}</el-descriptions-item>
      <el-descriptions-item label="补充说明">{{ dataForm.caption }}</el-descriptions-item>
      <el-descriptions-item label="标签">
        <el-tag size="small" v-for="tag in dataForm.tags" :key="tag">{{ tag }}</el-tag>
      </el-descriptions-item>
    </el-descriptions>
    <el-divider>审核</el-divider>
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()"
             :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
      <el-form-item label="审核结果">
        <el-radio-group v-model="auditForm.isPassed">
          <el-radio :label="0">审核不通过</el-radio>
          <el-radio :label="1">审核通过</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="理由" v-if="auditForm.isPassed === 0">
        <el-input
            type="textarea"
            placeholder=""
            v-model="auditForm.resultMessage"
            maxlength="100"
            show-word-limit
        >
        </el-input>
      </el-form-item>
    </el-form>
    <template slot="footer">
      <el-button @click="visible = false">{{ $t('cancel') }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">提交</el-button>
    </template>
  </el-dialog>
</template>

<script>
import debounce from 'lodash/debounce'
import UserSelect from "@/components/user-select";

export default {
  components: {UserSelect},
  data() {
    return {
      visible: false,
      dataForm: {
        id: '',
        userId: '',
        imageUrls: [],
        address: '',
        caption: '',
        tags: [],
      },
      auditForm: {
        postId: null,
        isPassed: 1,
        resultMessage: ''
      },
    }
  },
  computed: {
    dataRule() {
      return {
        isPassed: [
          {required: true, message: this.$t('validate.required'), trigger: 'blur'}
        ],
      }
    },
  },
  methods: {
    init() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.getInfo()
      })
    },
    // 获取信息
    getInfo() {
      this.$http.get(`/audit/audit-todo/${this.dataForm.id}`).then(({data: res}) => {
        if (res.code !== 0) {
          return this.$message.error(res.msg)
        }
        this.dataForm = {
          ...this.dataForm,
          ...res.data
        }
        this.auditForm.postId = this.dataForm.id
      }).catch(() => {
      })
    },
    // 表单提交
    dataFormSubmitHandle: debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return false
        }
        this.$http.post('/audit/audit-todo/doAudit', this.auditForm).then(({data: res}) => {
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
        }).catch(() => {
        })
      })
    }, 1000, {'leading': true, 'trailing': false})
  }
}
</script>

<style>
.image-upload .el-upload-list__item {
  transition: none !important;
}
</style>
