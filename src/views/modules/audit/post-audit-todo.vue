<template>
  <el-card shadow="never" class="aui-card--fill">
    <div class="mod-post__post}">
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
        <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template slot-scope="scope">
            <el-button v-if="$hasPermission('audit:audit-todo:do')" type="text" size="small"
                       @click="addOrUpdateHandle(scope.row.id)">{{ $t('manage') }}
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
      <!-- 弹窗, 进行审核 -->
      <audit-do v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></audit-do>
    </div>
  </el-card>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import AuditDo from './post-audit-do'

export default {
  mixins: [mixinViewModule],
  data() {
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/audit/audit-todo/page',
        getDataListIsPage: true,
      },
    }
  },
  components: {
    AuditDo
  }
}
</script>
