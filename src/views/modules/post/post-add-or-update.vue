<template>
  <el-dialog :visible.sync="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false"
             :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()"
             :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
      <el-form-item label="用户ID" prop="userId">
        <user-select v-model="dataForm.userId"/>
      </el-form-item>
      <el-form-item label="图片" prop="imageUrls">
        <el-upload
            class="image-upload"
            :file-list="uploadImage.fileList"
            :action="uploadImage.url"
            :headers="uploadImage.headers"
            list-type="picture-card"
            multiple
            :on-success="handleUploadSuccess"
            accept=".png, .jpg, .jpeg">
          <i slot="default" class="el-icon-plus"></i>
          <div slot="file" slot-scope="{file}">
            <el-image class="el-upload-list__item-thumbnail" :src="file.url" alt="" fit="cover"/>
            <span class="el-upload-list__item-actions">
              <span class="el-upload-list__item-delete" @click="handleRemove(file)">
                <i class="el-icon-delete"></i>
              </span>
            </span>
          </div>
        </el-upload>
      </el-form-item>
      <el-form-item label="地点" prop="address">
        <el-autocomplete
            v-model="dataForm.address"
            :fetch-suggestions="addressTips"
            @select="addressSelect"
            :trigger-on-focus="false"
            placeholder="地点"
        >
          <template slot-scope="{ item }">
            <span>{{ item.label }}</span>
            <span style="color: #909399; float:right;">{{ item.quantity }}</span>
          </template>
        </el-autocomplete>
      </el-form-item>
      <el-form-item label="补充说明" prop="caption">
        <el-input v-model="dataForm.caption" placeholder="补充说明"></el-input>
      </el-form-item>
      <el-form-item label="标签" prop="tags">
        <el-select
            v-model="dataForm.tags"
            multiple
            filterable
            allow-create
            remote
            reserve-keyword
            placeholder="标签"
            :remote-method="tagSearch"
            :loading="tagSelect.loading">
          <el-option
              v-for="tag in tagSelect.options"
              :key="tag.id"
              :value="tag.content ">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="审核状态" prop="auditStatus">
        <ren-radio-group v-model="dataForm.auditStatus" dict-type="audit_status"
                         :disabled="!!dataForm.id"></ren-radio-group>
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
import {generateHeaders} from "@/utils/request";
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
        auditStatus: 2
      },
      tagSelect: {
        loading: false,
        options: []
      },
      uploadImage: {
        headers: generateHeaders(),
        url: window.SITE_CONFIG['apiURL'] + '/post/post/uploadImage',
        fileList: [],
      },
    }
  },
  computed: {
    dataRule() {
      const validateImageUrls = (rule, value, callback) => {
        if (!this.dataForm.imageUrls || this.dataForm.imageUrls.length < 1) {
          return callback(new Error('请至少选择一张图片'))
        }
        if (this.dataForm.imageUrls.length > 9) {
          return callback(new Error('图片数量不能超过9张'))
        }
        callback()
      }
      return {
        userId: [
          {required: true, message: this.$t('validate.required'), trigger: 'blur'}
        ],
        imageUrls: [
          {validator: validateImageUrls, trigger: 'blur'}
        ],
        auditStatus: [
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
        this.uploadImage.fileList = []
        if (this.dataForm.id) {
          this.getInfo()
        }
      })
    },
    // 获取信息
    getInfo() {
      this.$http.get(`/post/post/${this.dataForm.id}`).then(({data: res}) => {
        if (res.code !== 0) {
          return this.$message.error(res.msg)
        }
        this.dataForm = {
          ...this.dataForm,
          ...res.data
        }
        this.uploadImage.fileList = this.dataForm.imageUrls.map(url => {
          return {url}
        })
      }).catch(() => {
      })
    },
    tagSearch(query) {
      if (query !== '') {
        this.userSelect.loading = true
        this.$http.get('/post/tag/page', {
          params: {
            page: 1,
            limit: 20,
            query: query,
          }
        }).then(resp => {
          this.tagSelect.loading = false
          this.tagSelect.options = resp.data.data.list
        })
      } else {
        this.tagSelect.options = [];
      }
    },
    addressTips(keywords, cb) {
      this.$http.get('/post/address/tips', {
        params: {
          keywords,
        }
      }).then(resp => {
        cb(resp.data.data)
      })
    },
    addressSelect(item) {
      this.dataForm.address = item.label
    },
    handleUploadSuccess(resp, file, fileList) {
      file.url = resp.data.src
      this.uploadImage.fileList = fileList
      this.dataForm.imageUrls = fileList.map(item => item.url)
    },
    handleRemove(file) {
      this.uploadImage.fileList = this.uploadImage.fileList.filter(item => item.url !== file.url)
      this.dataForm.imageUrls = this.uploadImage.fileList.map(item => item.url)
    },
    // 表单提交
    dataFormSubmitHandle: debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return false
        }
        this.$http[!this.dataForm.id ? 'post' : 'put']('/post/post/', this.dataForm).then(({data: res}) => {
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
