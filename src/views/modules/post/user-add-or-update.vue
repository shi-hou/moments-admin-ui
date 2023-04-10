<template>
  <el-dialog :visible.sync="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false"
             :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()"
             :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="dataForm.username" placeholder="用户名"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
        <el-input v-model="dataForm.mobile" placeholder="手机号"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="个性签名" prop="bio">
        <el-input v-model="dataForm.bio" placeholder="个性签名"></el-input>
      </el-form-item>
      <el-form-item label="性别" prop="gender">
        <ren-radio-group v-model="dataForm.gender" dict-type="gender"></ren-radio-group>
      </el-form-item>
      <el-form-item label="头像" prop="avatar">
        <el-upload
            class="avatar-uploader"
            :action="uploadAvatar.url"
            :headers="uploadAvatar.headers"
            :show-file-list="false"
            :on-success="handleAvatarUpload"
            accept=".png, .jpg, .jpeg">
          <el-image v-if="dataForm.avatar" :src="dataForm.avatar" class="avatar" fit="cover"/>
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
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
import {isMobile} from "@/utils/validate"
import {generateHeaders} from "@/utils/request"

export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: '',
        username: '',
        mobile: '',
        password: '',
        createDate: '',
        bio: '',
        gender: '2',
        avatar: '',
      },
      uploadAvatar: {
        headers: generateHeaders(),
        url: window.SITE_CONFIG['apiURL'] + '/post/user/uploadAvatar'
      }
    }
  },
  computed: {
    dataRule() {
      const validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          return callback(new Error(this.$t('validate.required')))
        }
        callback()
      };
      const validateMobile = (rule, value, callback) => {
        if (value && !isMobile(value)) {
          return callback(new Error(this.$t('validate.format', {'attr': this.$t('user.mobile')})))
        }
        callback()
      };
      return {
        username: [
          {required: true, message: this.$t('validate.required'), trigger: 'blur'}
        ],
        mobile: [
          {required: true, validator: validateMobile, trigger: 'blur'}
        ],
        password: [
          {validator: validatePassword, trigger: 'blur'}
        ],
        gender: [
          {required: true, message: this.$t('validate.required'), trigger: 'blur'}
        ],
      }
    }
  },
  methods: {
    init() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.getInfo()
        }
      })
    },
    // 获取信息
    getInfo() {
      this.$http.get(`/post/user/${this.dataForm.id}`).then(({data: res}) => {
        if (res.code !== 0) {
          return this.$message.error(res.msg)
        }
        this.dataForm = {
          ...this.dataForm,
          ...res.data
        }
      }).catch(() => {
      })
    },
    // 上传头像成功
    handleAvatarUpload(resp) {
      this.dataForm.avatar = resp.data.src
    },
    // 表单提交
    dataFormSubmitHandle: debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return false
        }
        this.$http[!this.dataForm.id ? 'post' : 'put']('/post/user/', this.dataForm).then(({data: res}) => {
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
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 80px;
  height: 80px;
  line-height: 80px;
  text-align: center;
}

.avatar {
  width: 80px;
  height: 80px;
  display: block;
}
</style>
