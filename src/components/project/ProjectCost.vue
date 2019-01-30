<template>
  <div>
    <div class="cost-title">项目指标</div>
    <a-row class="cost-rowLine">
      <a-col :span="6">
        <span class="cost-span">项目负责人：</span>
        <span>陈康然</span>
      </a-col>
      <a-col :span="6">
        <span class="cost-span">业务类型：</span>
        <span>审核施工图预算</span>
      </a-col>
      <a-col :span="6">
        <span class="cost-span">工程规模：</span>
        <span>0平方米</span>
      </a-col>
    </a-row>
    <a-row class="cost-rowLine">
      <a-col :span="6">
        <span class="cost-span">送审金额：</span>
        <a-input class="ui-w100" maxlength="12"/>
        <span>元</span>
      </a-col>
      <a-col :span="6">
        <span class="cost-span">审定金额：</span>
        <a-input class="ui-w100" maxlength="12"/>
        <span>元</span>
      </a-col>
      <a-col :span="6">
        <span class="cost-span">核增核减额：</span>
        <a-input class="ui-w100" maxlength="12"/>
        <span>元</span>
      </a-col>
      <a-col :span="6">
        <span class="cost-span">单方造价：</span>
        <a-input class="ui-w100" maxlength="12"/>
        <span>元/平方米</span>
      </a-col>
    </a-row>

    <!-- 任务表格 -->
    <div style="margin-top:20px" class="costTable">
      <a-table
        :columns="columns"
        :dataSource="tasksData"
        :pagination="false"
        size="small"
        :bordered="true"
        :scroll="{x:scrollX}"
      >
        <!-- <a-icon type="plus-circl" slot="title" scopedSlots="text,record,index">新增</a-icon> -->
        <span slot="ok">
          {{ costName }}
          <a-popconfirm placement="top" okText="确定" cancelText="取消" @confirm="addDraft">
            <template slot="title">
              <p>是否添加{{ costName }}，添加后无法删除</p>
            </template>
            <a-icon type="plus-circle" style="color:#5873c9;"/>
          </a-popconfirm>
        </span>

        <!-- </template> -->
        <!-- 自定义新增行 -->
        <template v-for="(col, i) in costNameArr" :slot="col" slot-scope="text, record, index">
          <div :key="col">
            <div v-if="enEditCost.indexOf(col)>-1">{{text[enEditCost.indexOf(col)].submitAmount}}</div>
            <div v-else>
              <a-input
                class="inputC"
                v-if="index <= tasksData.length-2"
                style="text-align:center;max-width:120px"
                :value="text"
                :placeholder="placeholer[i]"
                @change="e => handleChange(e.target.value, record.key, col)"
              />
              <!-- 最后一行 -->
              <div
                class="inputC"
                v-else
                style="text-align:center;line-height: 30px;font-weight:600;"
              >{{ text }}</div>
            </div>
          </div>
        </template>

        <!-- 负责人 -->
        <template slot="handler" slot-scope="text, record, index">
          <a-input
            class="inputC"
            readonly="readonly"
            style="text-align:center"
            :value="text"
            placeholder="负责人"
            v-if="index <= tasksData.length-2"
            @click="showEmployee(record)"
          />
        </template>
        <!-- 自定义操作 -->
        <template slot="operation" slot-scope="text, record, index">
          <span v-if="index <= tasksData.length-2">
            <a @click="saveRow(record)">
              <a-icon type="save" title="保存"/>
            </a>
            <a-divider type="vertical"/>
            <a @click="cancel(record.key)">
              <a-icon type="delete" title="删除"/>
            </a>
            <a-divider type="vertical"/>
            <a @click="addChild(record.key)">
              <a-icon type="share-alt" title="创建子任务"/>
            </a>
          </span>
        </template>

        <!-- <template slot="footer" slot-scope="currentPageData">{{ currentPageData }}</template> -->
      </a-table>

      <div class="addCostTask">
        <a href="javascript:void(0)" @click="addCost">
          <a-icon type="plus"></a-icon>添加任务
        </a>
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
import gql from 'graphql-tag'
import employeeTree from '@/components/same/employeeTree'

export default {
  name: 'ProjectCost',
  props: {
    projectId: {
      type: String,
      defalut: ''
    }
  },
  data() {
    return {
      costName: '初稿',
      scrollX: 1200,
      emTitle: '', //人员弹框标题
      visibled: false, //人员弹框是否显示
      isOne: true, //人员是否多选
      employeeType: 1, //部门人员弹框类型
      tasks: [], //原始数据
      tasksData: [], //处理过的数据
      docNum: 1, //稿数

      costNameArr: ['name', 'submitAmount', 'deviationRemark', 'numTasks1'],
      enEditCost: ['numTasks1'],
      // costNameArr: ['name', 'submitAmount', 'deviationRemark'],

      placeholer: ['任务名称', '送审金额', '偏差原因'],
      columns: [
        {
          title: '任务名称',
          dataIndex: 'name',
          key: 'name',
          width: '300',
          align: 'center',
          scopedSlots: {
            customRender: 'name'
          }
        },
        {
          title: '负责人',
          dataIndex: 'handler',
          key: 'handler',
          width: '100',
          align: 'center',
          scopedSlots: { customRender: 'handler' }
        },
        // {
        //   title: '复核人',
        //   dataIndex: 'department',
        //   key: 'department',
        //   width: '60',
        //   scopedSlots: { customRender: 'department' }
        // },
        {
          title: '送审金额',
          dataIndex: 'submitAmount',
          key: 'submitAmount',
          width: '100',
          align: 'center',
          scopedSlots: { customRender: 'submitAmount' }
        },
        {
          title: '造价金额',
          dataIndex: 'numTasks',
          key: 'numTasks',
          align: 'center',
          // width: '300',
          children: [
            {
              // title: '初稿',
              dataIndex: 'numTasks',
              key: 'numTasks1',
              width: '100',
              align: 'center',
              slots: {
                title: 'ok'
              },
              scopedSlots: { customRender: 'numTasks1' }
            }
          ]
        },
        {
          title: '偏差率',
          dataIndex: 'deviationRate',
          key: 'deviationRate',
          width: '120',
          align: 'center',
          scopedSlots: { customRender: 'deviationRate' }
        },
        {
          title: '偏差原因',
          dataIndex: 'deviationReason',
          key: 'deviationReason',
          width: '120',
          align: 'center',
          scopedSlots: { customRender: 'deviationReason' }
        },
        {
          title: '操作',
          key: 'action',
          // fixed: 'right',
          align: 'center',
          width: '120',
          scopedSlots: { customRender: 'operation' }
        }
      ],
      data: [
        {
          key: '1',
          name: '小明',
          workId: '001',
          editable: false,
          department: '行政部'
        },
        {
          key: '2',
          name: '李莉',
          workId: '002',
          editable: false,
          department: 'IT部'
        },
        {
          key: '3',
          name: '汇总',
          workId: '003',
          editable: false,
          department: '财务部'
        }
      ]
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
                businessType
                projectScale
                projectScaleUnit
                submitAmount
                docNum
                approvedAmount
                tasks {
                  id
                  name
                  type
                  parentNode{
                    id
                    name
                    type
                  }
                  submitAmount
                  childTasks {
                    id
                    name
                    remark
                    type
                    submitAmount
                    approvedAmount
                    handler{
                      name
                      id
                    }
                    numTasks {
                      id
                      name
                      remark
                      submitAmount
                      approvedAmount
                      handler {
                        name
                        id
                      }
                      childTasks {
                        id
                        name
                        remark
                        submitAmount
                        approvedAmount
                        handler {
                          name
                          id
                        }
                      }
                    }
                  }
                  numTasks{
                    id
                    name
                    remark
                    submitAmount
                    approvedAmount
                    handler {
                      name
                      id
                    }
                  }
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
      console.log(data)
      this.tasks = data[0].tasks
      this.beforeTable(this.tasks)
      // this.principal.name = data[0].principal
      // this.steps = data[0].steps
      // this.summarys = data[0].summarys
      // this.plans = data[0].plans
      // this.status = data[0].status
      // var statusData = this.statusArr.filter(ele => ele.value == status)
      // this.nowStatus = statusData[0].name
      // this.nowStatusColor = statusData[0].background
      // this.summarysData = {};
      // this.summarys.forEach(_sum => {
      //   var time = moment(_sum.createdAt).format('YYYY-MM-DD')
      //   var obj = Object.assign({},_sum);
      //   obj.time = moment(_sum.createdAt).format('HH:mm')
      //   if(this.summarysData[time]){
      //     this.summarysData[time].push(obj)
      //   }else {
      //     this.summarysData[time] = [obj]
      //   }
      // });
      // console.log(this.summarysData)
    },
    saveRow(record) {
      const target = this.data.filter(item => item.key === key)[0]
      target.editable = false
      target.isNew = false
    },

    cancel(key) {
      const target = this.data.filter(item => item.key === key)[0]
      target.editable = false
    },
    addChild(key) {},
    handleChange(value, key, column) {
      const newData = [...this.data]
      const target = newData.filter(item => key === item.key)[0]
      if (target) {
        target[column] = value
        this.data = newData
      }
    },
    addDraft() {
      var draftObj = {}
      var draftIndex = 0
      this.columns.forEach((ele, index) => {
        if (ele.title == '造价金额') {
          draftObj = ele
          draftIndex = index
        }
      })
      // for(var o in draftObj){
      var draftCL = draftObj['children'] //造价金额子表头
      var draftCLL = draftCL.length //稿数长度

      if (draftCLL > 6) {
        return
      }
      var obj = { 0: '初稿', 1: '二稿', 2: '三稿', 3: '四稿', 4: '五稿', 5: '六稿', 6: '终稿' }

      var lastDraft = Object.assign({}, draftCL[draftCLL - 1]) //复制

      lastDraft.scopedSlots.customRender = 'numTasks' + (draftCLL + 1) //属性唯一
      lastDraft.key = 'numTasks' + (draftCLL + 1) //属性唯一

      this.costNameArr.push('numTasks' + (draftCLL + 1))
      this.enEditCost.push('numTasks' + (draftCLL + 1))

      draftCL[draftCLL - 1].title = this.costName //有添加按钮的就不能设置title
      delete draftCL[draftCLL - 1].slots //删除自定义表头

      this.costName = obj[draftCLL] //稿名下次使用

      draftCL.push(lastDraft)
      draftObj['children'] = draftCL
      this.columns.splice(draftIndex, 1, draftObj)
      this.scrollX += draftCLL * 20

      this.$apollo.mutate({
        mutation: gql`
          mutation {
            updateProject(data: { docNum: 2 }, where: { id: "${this.projectId}" }) {
              docNum
              id
            }
          }
        `
      }).then(res => {
        console.log(res)


      }).catch(err => {
        console.log(err)
      })


    },
    showEmployee(record) {
      this.visibled = true
    },
    changeEMStatus(obj) {
      this.visibled = obj.visibled
    },

    addCost() {
      var obj = {
        name: '',
        handler: {},
        project: { id: this.projectId },
        type: 0,
        submitAmount: '',
        numTasks: [
          {
            name: '',
            handler: {},
            project: { id: this.projectId },
            type: 1,
            submitAmount: ''
          }
        ]
      }
      this.tasks.push(obj)
      this.beforeTable(this.tasks)
    },

    beforeTable(data) {
      var obj = {
        name: '汇总',
        submitAmount: 0,
        numTasks: [
          {
            submitAmount: 0
          }
        ]
      }
      this.tasksData = data.concat([obj])
    }
  }
}
</script>

<style>
.ui-w100 {
  width: 112px;
}
.cost-span {
  display: inline-block;
  width: 90px;
  text-align: right;
}
.cost-title {
  color: #5873c9;
  line-height: 32px;
  margin-bottom: 10px;
  font-size: 16px;
}
.cost-rowLine {
  /* height: 28px; */
  /* line-height: 28px; */
  margin-bottom: 10px;
}

.addCostTask {
  margin-top: 10px;
  font-size: 12px;
}
.addCostTask a {
  color: #5873c9;
}

.costTable .ant-table-small > .ant-table-content > .ant-table-body {
  margin: 0;
}
.costTable .ant-table-thead tr th {
  background-color: #eee !important;
}
.costTable .ant-table-small .ant-table-content .ant-table-body .ant-table-row td {
  padding: 0;
  line-height: 0px;
}
.costTable .ant-table-row-hover td {
  background: #fff !important;
}
.inputC {
  margin: -5px 0;
  text-align: 'center';
  border: 0;
}
.inputC:focus {
  box-shadow: unset;
}
</style>

