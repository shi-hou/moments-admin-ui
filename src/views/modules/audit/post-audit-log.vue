<template>
  <el-card shadow="never" class="aui-card--fill">
    <div class="mod-audit__postaudit}">
      <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item>
          <el-input v-model="dataForm.id" placeholder="id" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="getDataList()">{{ $t('query') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="info" @click="exportHandle()">{{ $t('export') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('audit:postaudit:save')" type="primary" @click="addOrUpdateHandle()">{{ $t('add') }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="$hasPermission('audit:postaudit:delete')" type="danger" @click="deleteHandle()">{{ $t('deleteBatch') }}</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="dataListLoading" :data="dataList" border @selection-change="dataListSelectionChangeHandle" style="width: 100%;">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="id" label="id" header-align="center" align="center"></el-table-column>
        <el-table-column prop="postId" label="贴子ID" header-align="center" align="center"></el-table-column>
        <el-table-column prop="sysUserId" label="审核员ID" header-align="center" align="center">
          <template slot-scope="scope">
            {{ scope.row.sysUserId || '机器审核' }}
          </template>
        </el-table-column>
        <el-table-column prop="isPassed" label="审核通过与否" header-align="center" align="center">
          <template slot-scope="scope">
            {{ scope.row.isPassed === 1 ? '是' : '否' }}
          </template>
        </el-table-column>
        <el-table-column prop="resultMessage" label="结果信息" header-align="center" align="center"></el-table-column>
        <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
<!--        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">-->
<!--          <template slot-scope="scope">-->
<!--            <el-button v-if="$hasPermission('audit:postaudit:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">{{ $t('update') }}</el-button>-->
<!--            <el-button v-if="$hasPermission('audit:postaudit:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">{{ $t('delete') }}</el-button>-->
<!--          </template>-->
<!--        </el-table-column>-->
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
      <log-check v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></log-check>
    </div>
  </el-card>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import LogCheck from './post-audit-log-check'
export default {
  mixins: [mixinViewModule],
  data () {
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/audit/audit-log/page',
        getDataListIsPage: true,
        exportURL: '/audit/audit-log/export',
        deleteURL: '/audit/audit-log',
        deleteIsBatch: true
      },
      dataForm: {
        id: ''
      }
    }
  },
  components: {
    LogCheck
  }
}
</script>
