<template>
  <div class="ren-9-box-grid">
    <div class="img-list-wrap">
      <div class="img-item-wrap" v-for="(url, index) in urls" :key="url">
        <div class="img-item">
          <el-image :src="url" fit="cover" alt="" :preview-src-list="urls" :initial-index="index"/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Ren9BoxGrid',
  props: {
    urls: {
      type: Array,
      required: true,
    },
  },
}
</script>

<style>
.ren-9-box-grid .img-list-wrap {
  display: flex;
  flex-wrap: wrap;
  margin: 0 auto;
}

.ren-9-box-grid .img-item-wrap {
  box-sizing: border-box;
  width: 33.33%;
  margin-bottom: 10px;
  padding-left: 5px;
  padding-right: 5px;
}

.ren-9-box-grid .img-item {
  box-sizing: border-box;
  width: 100%;
  height: 0;
  padding-bottom: 100%;
  position: relative;
  overflow: hidden;
}

.ren-9-box-grid .el-image {
  width: 100%;
  height: 100%;
  position: absolute;
  display: inline-block;
}
</style>
