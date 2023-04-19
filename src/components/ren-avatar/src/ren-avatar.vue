<template>
  <div class="ren-avatar">
    <el-avatar
        :size="size"
    >
      <el-image fit="cover" :src="src ? src : 'http://moments-image.oss-cn-shenzhen.aliyuncs.com/20230316/b7e357f25f62493185763d5266aafc42.png'"
                style="height: 100%; width: 100%"/>
    </el-avatar>
  </div>
</template>
<script>
export default {
  name: 'RenAvatar',
  props: {
    size: {
      type: Number,
      default: 50,
    },
    src: String,
  },
}
</script>

<style>
.ren-avatar {
  display: inline-block;
}

.ren-avatar .el-avatar {
  vertical-align: middle;
}
</style>
