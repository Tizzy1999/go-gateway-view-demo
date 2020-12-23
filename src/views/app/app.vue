<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit===false">Create App</span>
          <span v-if="isEdit===true">Update App</span>
        </div>
        <div style="margin-bottom:50px;">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="app_id" class="is-required">
              <el-input v-model="form.app_id" placeholder="6-128 characters" :disabled="isEdit===true" />
            </el-form-item>
            <el-form-item label="Name" class="is-required">
              <el-input v-model="form.name" placeholder="Maximum 255 characters" />
            </el-form-item>
            <el-form-item label="Secret Key">
              <el-input v-model="form.secret" placeholder="32-bit auto-generated" />
            </el-form-item>

            <el-form-item label="QPS limit">
              <el-input v-model="form.qps" placeholder="0表示无限制" />
            </el-form-item>

            <el-form-item label="QPD limit">
              <el-input v-model="form.qpd" placeholder="0表示无限制" />
            </el-form-item>

            <el-form-item>
              <el-button type="primary" :disabled="submitButDisabled" @click="handleSubmit">Submit</el-button>
            </el-form-item>
          </el-form>
        </div>
      </el-card>
    </el-row>
  </div>
</template>

<script>
import { appAdd, appDetail, appUpdate } from '@/api/app'
export default {
  name: 'ServiceAddApp',
  data() {
    return {
      isEdit: false,
      submitButDisabled: false,
      form: {
        id: '',
        app_id: '',
        name: '',
        secret: '',
        white_ips: '',
        qps: '',
        qpd: ''
      }
    }
  },
  created() {
    const id = this.$route.params && this.$route.params.id
    if (id > 0) {
      this.isEdit = true
      this.fetchData(id)
    }
  },
  methods: {
    fetchData(id) {
      console.log('fetch by ID: ' + id)
      const query = { 'id': id }
      appDetail(query).then(response => {
        this.form.id = response.data.id
        this.form.app_id = response.data.app_id
        this.form.name = response.data.name
        this.form.id = response.data.id
        this.form.secret = response.data.secret
        this.form.qps = response.data.qps
        this.form.qpd = response.data.qpd
        console.log(response)
      }).catch((err) => {
        console.log('fetch data error: ' + err)
      })
    },
    handleSubmit() {
      this.submitButDisabled = true
      const query = Object.assign({}, this.form)
      console.log(query)
      query.id = Number(query.id)
      query.qpd = Number(query.qpd)
      query.qps = Number(query.qps)
      if (this.isEdit) {
        appUpdate(query).then(response => {
          this.submitButDisabled = false
          this.$notify({
            title: 'Success',
            message: 'Updated',
            type: 'success',
            duration: 2000
          })
        }).catch(() => {
          this.submitButDisabled = false
        })
      } else {
        appAdd(query).then(response => {
          this.submitButDisabled = false
          this.$notify({
            title: 'Success',
            message: 'Added',
            type: 'success',
            duration: 2000
          })
        }).catch(() => {
          this.submitButDisabled = false
        })
      }
    }
  }
}
</script>

<style scoped>
.mixin-components-container {
  background-color: #f0f2f5;
  padding: 30px;
  min-height: calc(100vh - 84px);
}
.component-item{
  min-height: 100px;
}
</style>
