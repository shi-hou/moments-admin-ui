<template>
  <el-card shadow="never" class="aui-card--fill">
    <div class="mod-post__post}">
      <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item label="用户ID">
          <user-select v-model="dataForm.userId"></user-select>
        </el-form-item>
        <el-form-item label="审核状态">
          <ren-select v-model="dataForm.auditStatus" dict-type="audit_status"></ren-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="getDataList()">{{ $t('query') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="info" @click="exportHandle()">{{ $t('export') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('post:post:save')" type="primary" @click="addOrUpdateHandle()">
            {{ $t('add') }}
          </el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('post:post:delete')" type="danger" @click="deleteHandle()">
            {{ $t('deleteBatch') }}
          </el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="dataListLoading" :data="dataList" border @selection-change="dataListSelectionChangeHandle"
                style="width: 100%;">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="userId" label="用户ID" header-align="center" align="center"></el-table-column>
        <el-table-column prop="previewImageUrl" label="头图" header-align="center" align="center">
          <template slot-scope="scope">
            <el-image :src="scope.row.previewImageUrl"/>
          </template>
        </el-table-column>
        <el-table-column prop="images" label="图片数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="address" label="地点" header-align="center" align="center"></el-table-column>
        <el-table-column prop="auditStatus" label="审核状态" header-align="center" align="center">
          <template slot-scope="scope">
            <el-tag size="small" :type="auditStatusTagTypes[scope.row.auditStatus]">
              {{ $getDictLabel("audit_status", scope.row.auditStatus) }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
        <el-table-column prop="views" label="浏览量" header-align="center" align="center"></el-table-column>
        <el-table-column prop="likes" label="点赞数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="stars" label="收藏数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="comments" label="评论数" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template slot-scope="scope">
            <el-button v-if="$hasPermission('post:post:update')" type="text" size="small"
                       @click="addOrUpdateHandle(scope.row.id)">{{ $t('update') }}
            </el-button>
            <el-button v-if="$hasPermission('post:post:delete')" type="text" size="small"
                       @click="deleteHandle(scope.row.id)">{{ $t('delete') }}
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
          :current-page="page"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="limit"
          :total="total"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="pageSizeChangeHandle"
          @current-change="pageCurrentChangeHandle">
      </el-pagination>
      <!-- 弹窗, 新增 / 修改 -->
      <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    </div>
  </el-card>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import AddOrUpdate from './post-add-or-update'
import UserSelect from "@/components/user-select";

export default {
  mixins: [mixinViewModule],
  data() {
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/post/post/page',
        getDataListIsPage: true,
        exportURL: '/post/post/export',
        deleteURL: '/post/post',
        deleteIsBatch: true
      },
      dataForm: {
        userId: '',
        auditStatus: ''
      },
      auditStatusTagTypes: ['primary', 'primary', 'success', 'danger']
    }
  },
  components: {
    UserSelect,
    AddOrUpdate
  }
}
</script>
