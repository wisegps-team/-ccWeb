<template>
  <div class="projectMember">
    <div>
      <a-row>
        <a-col :span="12">
          <h3>项目成员</h3>
        </a-col>
        <a-col :span="12" style="text-align:right">
          <a-button type="default" icon="plus" @click="showEmployee">添加项目成员</a-button>
        </a-col>
      </a-row>
      <div :style="{marginTop:'10px'}" class="projectMemberTable">
        <a-table :columns="columns" :dataSource="data" :pagination="false">
          <a slot="name" slot-scope="text" href="javascript:;">{{ text }}</a>
          <a-avatar
            slot="img"
            slot-scope="text, record"
            src="https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
          />

          <div slot="remark" slot-scope="text, record" class="remark">
            <a-input :value="text? text:'备注'" type="text"/>
          </div>
          <div slot="icon" slot-scope="text, record" :style="{textAlign:'right'}">
            <span :style="{padding:'2px',cursor:'pointer'}">
              <a-icon type="delete"/>
            </span>
          </div>
        </a-table>
      </div>
    </div>
    <div :style="{marginTop:'20px'}">
      <a-row>
        <a-col :span="12">
          <h3>外部联系人</h3>
        </a-col>
        <a-col :span="12" style="text-align:right">
          <a-button type="default" icon="plus">添加外部联系人</a-button>
        </a-col>
      </a-row>
      <div :style="{marginTop:'10px'}" class="projectMemberTable">
        <a-table :columns="columns" :dataSource="data2" :pagination="false" v-if="data2.length">
          <a slot="name" slot-scope="text" href="javascript:;">{{ text }}</a>
          <a-avatar
            slot="img"
            slot-scope="text, record"
            src="https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
          />

          <div slot="remark" slot-scope="text, record" class="remark">
            <a-input :value="text? text:'备注'" type="text"/>
          </div>
          <div slot="icon" slot-scope="text, record" :style="{textAlign:'right'}">
            <span :style="{padding:'2px',cursor:'pointer'}">
              <a-icon type="delete"/>
            </span>
          </div>
        </a-table>
        <div v-else class="empty-tip">
          <div :style="{fontSize:'50px'}">
            <a-icon type="project"/>
          </div>
          <div>暂无数据</div>
        </div>
      </div>
    </div>

    <!-- 人员弹框 -->
    <employee-tree
      :inititle="emTitle"
      :inivisible="visibled"
      :type="employeeType"
      :isOne="isOne"
      @changeStatus="changeEMStatus"
    />
  </div>
</template>
<script>
import employeeTree from '@/components/same/employeeTree'
import gql from 'graphql-tag'

const columns = [
  {
    dataIndex: 'img',
    key: 'img',
    width: 60,
    //   slots: { title: 'customTitle' },
    scopedSlots: { customRender: 'img' }
  },
  {
    title: '姓名',
    width: 120,
    dataIndex: 'name',
    key: 'name'
  },
  {
    title: '职务',
    width: 120,
    dataIndex: 'job',
    key: 'job'
  },
  {
    title: '电话',
    width: 120,
    key: 'mobile',
    dataIndex: 'mobile'
  },
  {
    title: '备注',
    width: 120,
    key: 'remark',
    dataIndex: 'remark',
    scopedSlots: { customRender: 'remark' }
  },
  {
    key: 'icon',
    scopedSlots: { customRender: 'icon' }
  }
]

// const data = [
//   {
//     key: '1',
//     name: 'John Brown',
//     work: 32,
//     phone: '13088987654',
//     remark: 'nice'
//   },
//   {
//     key: '2',
//     name: 'Jim Green',
//     work: 42,
//     phone: '13088987654',
//     remark: 'loser'
//   },
//   {
//     key: '3',
//     name: 'Joe Black',
//     work: 32,
//     phone: '13088987654'
//     // remark: 'cool'
//   }
// ]

//对象转字符串
function ObjectToString(obj) {
  var type = Object.prototype.toString.call(obj)
  var result = ''
  if (type === '[object Object]') {
    result += '{'
    for (var o in obj) {
      if (Object.prototype.toString.call(obj[o]) === '[object String]') {
        result += o + ': "' + obj[o] + '",'
      } else {
        result += o + ': ' + obj[o] + ','
      }
    }
    result += '}'
  }
  return result
}
//数组转字符串
function ArrayToString(obj) {
  var type = Object.prototype.toString.call(obj)
  var result = ''
  if (type === '[object Array]') {
    result = '['
    obj.forEach(ele => {
      result += ObjectToString(ele)
    })
    result += ']'
  } else if (type === '[object Object]') {
    result += ObjectToString(obj)
  }
  return result
}

export default {
  name: 'ProjectMember',
  props: {
    projectId: {
      type: String,
      defalut: ''
    }
  },
  data() {
    return {
      columns,
      data: [], //项目成员
      data2: [], //外部人员
      emTitle: '添加人员',
      isOne: false,
      visibled: false,
      employeeType: 1,
      projectName:''
    }
  },
  components: {
    employeeTree
  },
  created() {
    this.loadData(this.projectId)
  },
  methods: {
    loadData(id) {
      var _this = this
      this.$apollo
        .query({
          query: gql`
            query {
              projects(where: {id:"${id}"}) {
                id
                name
                members {
                  id
                  sex
                  name
                  mobile
                  job
                }
              }
            }
          `,
          fetchPolicy: 'network-only'
        })
        .then(res => {
          console.log(res, 'res')
          _this.initInfo(res.data.projects)
        })
        .catch(err => {
          console.log(err)
        })
    },

    initInfo(data) {
      this.data = data[0].members
      this.projectName = data[0].name
      console.log(this.data.length)
    },

    showEmployee() {
      console.log('e')
      this.visibled = true
    },
    //添加项目成员
    changeEMStatus(obj) {
      console.log(obj)

      this.visibled = obj.visibled
      var data = []
      if (!obj.data.length) {
        return
      }
      obj.data.forEach(ele => {
        var obj = { id: ele.key }
        data.push(obj)
      })
      this.addMembers(data)
    },

    addMembers(data) {
      this.$apollo
        .mutate({
          mutation: gql`
            mutation {
              updateProject(
                data: {
                  members: { connect: ${ArrayToString(data)} }
                }
                where: { id: "${this.projectId}" }
              ) {
                id
                name
                members {
                  id
                  sex
                  name
                  mobile
                  job
                }
              }

              createNotification(
                data: {
                  type: 3
                  name: "【郑东鸣】将您添加为【${this.projectName}】的项目组成员"
                  status: 0
                  recipients: { connect: ${ArrayToString(data)} }
                }
              ) {
                id
                
              }
            }
          `
        })
        .then(res => {
          console.log(res,'update')
          this.data = res.data.updateProject.members
        }).catch(err => {

        })
    }
  }
}
</script>
<style>
.projectMemberTable .ant-table-tbody > tr > td {
  padding: 4px 16px;
}
.projectMemberTable .ant-table-tbody > tr > td .remark input {
  border: 0px;
}
.projectMemberTable .ant-table-tbody > tr > td .remark input:focus {
  box-shadow: unset;
}
.projectMember .empty-tip {
  text-align: center;
  padding: 50px 0;
  color: #cbcbcb;
}
</style>


