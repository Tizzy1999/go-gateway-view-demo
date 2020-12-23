<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit===false">Create GRPC Service</span>
          <span v-if="isEdit===true">Update GRPC Service</span>
        </div>
        <div style="margin-bottom:50px;">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="Service Name" class="is-required">
              <el-input v-model="form.service_name" placeholder="6-128 characters" :disabled="isEdit===true" />
            </el-form-item>
            <el-form-item label=" Description" class="is-required">
              <el-input v-model="form.service_desc" placeholder="Maximum 255 characters" />
            </el-form-item>
            <el-form-item label=" Port" class="is-required">
              <el-input v-model="form.port" placeholder="8001-8999" />
            </el-form-item>
            <el-form-item label="Metadata Transfer">
              <el-input v-model="form.header_transfer" type="textarea" autosize placeholder="add/del/edit format:add headerName headValue" />
            </el-form-item>
            <el-form-item label="Authentication">
              <el-switch
                v-model="form.open_auth"
                :active-value="1"
                :inactive-value="0"
              />
            </el-form-item>
            <el-form-item label="IP Whitelist">
              <el-input v-model="form.white_list" type="textarea" autosize placeholder="格式：127.0.0.1 多条换行，白名单优先于黑名单" />
            </el-form-item>
            <el-form-item label="IP Blacklist">
              <el-input v-model="form.black_list" type="textarea" autosize placeholder="格式：127.0.0.1 多条换行，白名单优先于黑名单" />
            </el-form-item>

            <el-form-item label="Client limit">
              <el-input v-model="form.clientip_flow_limit" placeholder="0表示无限制" />
            </el-form-item>

            <el-form-item label="Server limit">
              <el-input v-model="form.service_flow_limit" placeholder="0表示无限制" />
            </el-form-item>

            <el-form-item label="Round type">
              <el-radio-group v-model="form.round_type">
                <el-radio v-model="form.round_type" label="0">random</el-radio>
                <el-radio v-model="form.round_type" label="1">round-robin</el-radio>
                <el-radio v-model="form.round_type" label="2">weight_round-robin</el-radio>
                <el-radio v-model="form.round_type" label="3">ip_hash</el-radio>
              </el-radio-group>
            </el-form-item>

            <el-form-item label="IP list" class="is-required">
              <el-input v-model="form.ip_list" type="textarea" autosize placeholder="格式：127.0.0.1:80 多条换行" />
            </el-form-item>

            <el-form-item label="Weight list" class="is-required">
              <el-input v-model="form.weight_list" type="textarea" autosize placeholder="格式：50 多条换行" />
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
import { serviceAddGrpc, serviceDetail, serviceUpdateGrpc } from '@/api/service'
export default {
  name: 'ServiceCreateGRPC',
  data() {
    return {
      isEdit: false,
      submitButDisabled: false,
      form: {
        service_name: '',
        service_desc: '',
        header_transfer: '',
        round_type: 2,
        ip_list: '',
        weight_list: '',
        open_auth: 0,
        black_list: '',
        white_list: '',
        clientip_flow_limit: '',
        service_flow_limit: ''
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
      serviceDetail(query).then(response => {
        this.form.id = response.data.info.id
        this.form.service_name = response.data.info.service_name
        this.form.service_desc = response.data.info.service_desc
        this.form.port = response.data.grpc_rule.port.toString()
        this.form.open_auth = response.data.access_control.open_auth

        this.form.black_list = response.data.access_control.black_list.replace(/,/g, '\n')
        this.form.white_list = response.data.access_control.white_list.replace(/,/g, '\n')
        this.form.clientip_flow_limit = response.data.access_control.clientip_flow_limit
        this.form.service_flow_limit = response.data.access_control.service_flow_limit
        this.form.round_type = response.data.load_balance.round_type.toString()
        this.form.ip_list = response.data.load_balance.ip_list.replace(/,/g, '\n')
        this.form.weight_list = response.data.load_balance.weight_list.replace(/,/g, '\n')
        console.log(response)
      }).catch((err) => {
        console.log('fetch data error: ' + err)
      })
    },
    handleSubmit() {
      this.submitButDisabled = true
      const query = Object.assign({}, this.form)
      console.log(query)
      query.ip_list = query.ip_list.replace(/\n/g, ',')
      query.weight_list = query.weight_list.replace(/\n/g, ',')
      query.black_list = query.black_list.replace(/\n/g, ',')
      query.white_list = query.white_list.replace(/\n/g, ',')
      query.round_type = Number(query.round_type)
      query.port = Number(query.port)
      query.service_flow_limit = Number(query.service_flow_limit)
      query.clientip_flow_limit = Number(query.clientip_flow_limit)
      if (this.isEdit) {
        serviceUpdateGrpc(query).then(response => {
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
        serviceAddGrpc(query).then(response => {
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
