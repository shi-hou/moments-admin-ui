<template>
  <el-card shadow="never" class="aui-card--fill">
    <div class="mod-post__user}">
      <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item>
          <el-input v-model="dataForm.query" placeholder="搜索" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="getDataList()">{{ $t('query') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="info" @click="exportHandle()">{{ $t('export') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('post:user:save')" type="primary" @click="addOrUpdateHandle()">{{ $t('add') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('post:user:delete')" type="danger" @click="deleteHandle()">{{ $t('deleteBatch') }}</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="dataListLoading" :data="dataList" border @selection-change="dataListSelectionChangeHandle" style="width: 100%;">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="username" label="用户名" header-align="center" align="center"></el-table-column>
        <el-table-column prop="mobile" label="手机号" header-align="center" align="center"></el-table-column>
        <el-table-column prop="avatar" label="头像" header-align="center" align="center">
          <template slot-scope="scope">
            <ren-avatar :src="scope.row.avatar"></ren-avatar>
          </template>
        </el-table-column>
        <el-table-column prop="gender" label="性别" header-align="center" align="center">
          <template slot-scope="scope">
            {{ $getDictLabel("gender", scope.row.gender) }}
          </template>
        </el-table-column>
        <el-table-column prop="fans" label="粉丝数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="followers" label="关注数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template slot-scope="scope">
            <el-button v-if="$hasPermission('post:user:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">{{ $t('update') }}</el-button>
            <el-button v-if="$hasPermission('post:user:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">{{ $t('delete') }}</el-button>
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
import AddOrUpdate from './user-add-or-update'
export default {
  mixins: [mixinViewModule],
  data () {
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/post/user/page',
        getDataListIsPage: true,
        exportURL: '/post/user/export',
        deleteURL: '/post/user',
        deleteIsBatch: true
      },
      dataForm: {
        query: ''
      }
    }
  },
  components: {
    AddOrUpdate
  }
}
</script>
