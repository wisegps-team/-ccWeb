<template>
  <div>
    <a-modal
      :title="title"
      v-model="visibled"
      @ok="handleOk"
      @cancel="cancel"
      :maskClosable="maskClosable"
      :width="550"
    >
      <a-row>
        <a-col :span="24">
          <!-- <div style="height:40px">
            <a-input type="text" placeholder="姓名" style="width: 74%;"/>
            <a-button type="primary" style="width: 22%;float: right;">搜索</a-button>
          </div> -->
          <div class="treeDiv">
            <a-radio-group @change="onChange" v-model="value">
              <a-table
                size="small"
                :bordered="false"
                :columns="columns"
                :dataSource="clientData"
                :pagination="false"
              >
                <div slot="clientId" slot-scope="text,record,index">
                  <a-radio :value="index"></a-radio>
                </div>
                <div
                  slot="address"
                  slot-scope="text,record,index"
                >{{record.address.province + record.address.city + record.address.district + record.address.address}}</div>
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
  name: 'ClientDialog',
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
      clientData: [],
      columns: [
        {
          dataIndex: 'id',
          scopedSlots: { customRender: 'clientId' }
        },
        {
          title: '委托人名称',
          dataIndex: 'name',
          width: '350px',
          scopedSlots: { customRender: 'name' }
        },
        {
          title: '委托方地址',
          dataIndex: 'address',
          width: '350px',
          scopedSlots: { customRender: 'address' }
        }
      ]
    }
  },
  created() {
    // this.loadData()
  },
  methods: {
    handleOk() {
      var data = this.clientData[this.value]
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
              clients {
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
          `,
          fetchPolicy: 'network-only'
        })
        .then(res => {
          _this.clientData = res.data.clients
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

