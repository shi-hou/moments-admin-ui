<template>
  <el-select
      :value="value"
      @input="$emit('input', $event)"
      filterable
      remote
      reserve-keyword
      clearable
      placeholder="输入用户名或手机号搜索"
      :remote-method="userSearch"
      :loading="loading">
    <el-option
        v-for="user in options"
        :key="user.id"
        :value="user.id"
    >
      <ren-avatar :src="user.avatar" :size="20"/>
      {{ user.username }}({{ user.mobile }})
    </el-option>
  </el-select>
</template>
<script>
export default {
  name: 'UserSelect',
  data() {
    return {
      loading: false,
      options: []
    }
  },
  props: ['value'],
  methods: {
    userSearch(query) {
      if (query !== '') {
        this.loading = true
        this.$http.get('/post/user/page', {
          params: {
            page: 1,
            limit: 20,
            query: query,
          }
        }).then(resp => {
          this.loading = false
          this.options = resp.data.data.list
        })
      } else {
        this.options = [];
      }
    },
  }

}
</script>
