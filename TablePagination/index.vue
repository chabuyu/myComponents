<template>
    <div class="table-pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :page-sizes="pageSizes"
          :page-size="pageSize"
          :current-page="currentPage"
          layout="total, sizes, prev, pager, next, jumper"
          :total="dataTotal">
        </el-pagination>
    </div>
</template>
<script>
    export default {
      props: {
        dataTotal: {
          type: Number,
          default: 0
        },
        pageSize: {
          type: Number,
          default: 10
        },
        pageSizes: {
          type: Array,
          default: function () {
            return [10, 15, 20, 30]
          }
        },
        currentPage: {
          type: Number,
          default: 1
        }
      },
      methods: {
        handleSizeChange (val) {
          this.$emit('size-change', val)
        },
        handleCurrentChange (val) {
          this.$emit('current-change', val)
        }
      }
    }
</script>
<style scoped>
    .table-pagination{
        margin-top: 10px;
    }
</style>
