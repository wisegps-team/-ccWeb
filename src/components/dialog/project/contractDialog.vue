<template>
  <div>
    <a-modal
      :title="title"
      v-model="visibled"
      @ok="handleOk"
      @cancel="handleOk"
      :maskClosable="maskClosable"
      :width="550"
    >
      <a-row>
        <a-col :span="24">
          <a-row>
            <a-col :span="16">
              <div style="height:40px">
                <a-input type="text" placeholder="姓名" style="width: 74%;"/>
                <a-button type="primary" style="width: 22%;float: right;">搜索</a-button>
              </div>
            </a-col>
          </a-row>

          <div class="treeDiv">
            <a-radio-group @change="onChange" v-model="value">
              <a-table
                size="small"
                :bordered="false"
                :columns="columns"
                :dataSource="contractData"
                :pagination="false"
              >
                <div slot="contractId" slot-scope="text,record,index">
                  <a-radio :value="index"></a-radio>
                </div>
                <div
                  slot="client"
                  slot-scope="text,record,index"
                >{{record.name}}</div>
              </a-table>
            </a-radio-group>
          </div>
        </a-col>
      </a-row>
    </a-modal>
  </div>
</template>
<script>
import gql from 'graphql-tag'

export default {
  name: 'ContractDialog',
  props: {
    inititle: {
      type: String,
      default: ''
    },
    inivisible: {
      type: Boolean,
      default: false
    },
    maskClosable: {
      //点击蒙层是否允许关闭
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      title: this.inititle,
      visibled: this.inivisible,
      value: 1,
      contractData: [],
      columns: [
        {
          dataIndex: 'id',
          scopedSlots: { customRender: 'contractId' }
        },
        {
          title: '合同编号',
          dataIndex: 'contractNo',
          width: '350px',
          scopedSlots: { customRender: 'contractNo' }
        },
        {
          title: '合同名称',
          dataIndex: 'name',
          width: '350px',
          scopedSlots: { customRender: 'name' }
        },
        {
          title: '委托人',
          dataIndex: 'client',
          width: '350px',
          scopedSlots: { customRender: 'client' }
        }
      ]
    }
  },
  created() {
    // this.loadData()
  },
  methods: {
    handleOk() {
      var data = this.contractData[this.value]
      this.$emit('changeStatus', { visibled: false, data:data })
    },
    cancel() {
      this.$emit('changeStatus', { visibled: false })
    },
    onChange(e) {
      console.log('radio checked', e.target.value)
    },
    //加载数据
    loadData() {
      var _this = this
      this.$apollo
        .query({
          query: gql`
            query {
              contracts {
                id
                name 
                contractNo
                client {
                  id
                  name
                  address {
                    province
                    city
                    district
                    address
                  }
                }
              }
            }
          `,
          fetchPolicy: 'network-only'
        })
        .then(res => {
          console.log(res,'res')
          _this.contractData = res.data.contracts
        })
        .catch(err => {
          console.log(err)
        })
    }
  },
  watch: {
    inivisible(newT) {
      // console.log(newT)
      this.visibled = newT
      if (newT) {
        this.value = -2
        this.loadData()
      }
    }
  }
}
</script>
