<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.info" placeholder="service name/ description" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" style="margin-left: 10px" @click="handleFilter">     Search
      </el-button>
      <router-link :to="'/app/app_create/'">
        <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit">
          Add App
        </el-button>
      </router-link>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column label="ID" prop="id" align="center" width="50">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="app_id" min-width="50px">
        <template slot-scope="{row}">
          <span>{{ row.app_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Name" min-width="80px">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Secret Key" min-width="120px">
        <template slot-scope="{row}">
          <span>{{ row.secret }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPS" min-width="30px">
        <template slot-scope="{row}">
          <span>{{ row.real_qps }}/{{ row.qps }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPD" min-width="30px">
        <template slot-scope="{row}">
          <span>{{ row.real_qpd }}/{{ row.qpd }}</span>
        </template>
      </el-table-column>

      <el-table-column label="Operation" align="center" min-width="120" class-name="fixed-width">
        <template slot-scope="{row}">
          <router-link :to="'/app/app_stat/'+row.id">
            <el-button type="primary" size="small" style="margin-right: 5px">
              Statistics
            </el-button>
          </router-link>
          <router-link :to="'/app/app_edit/'+row.id">
            <el-button type="primary" size="small" style="margin-right: 5px">
              Update
            </el-button>
          </router-link>
          <el-button size="small" type="danger" @click="handleDelete(row)">
            Delete
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
  </div>
</template>

<script>
import { appList, appDelete } from '@/api/app'
import waves from '@/directive/waves' // waves directive
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

export default {
  name: 'AppList',
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page_no: 1,
        page_size: 20,
        info: ''
      },
      temp: {
        id: undefined
      }
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      appList(this.listQuery).then(response => {
        this.list = response.data.list
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page_no = 1
      this.getList()
    },
    handleDelete(row) {
      this.$confirm('Are you sure to delete the record? This process cannot be undone.', 'Warning', {
        confirmButtonText: 'Delete',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        const deleteQuery = {
          'id': row.id
        }
        appDelete(deleteQuery).then(response => {
          this.$notify({
            title: 'Success',
            message: 'delete success',
            type: 'success',
            duration: 2000
          })
          this.getList()
        })
      }).catch(() => {
        this.$notify({
          title: 'Canceled',
          message: 'delete canceled',
          type: 'info',
          duration: 2000
        })
      })
      // this.list.splice(index, 1)
    }
  }
}
</script>
