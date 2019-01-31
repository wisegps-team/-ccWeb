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
            <div v-if="enEditCost.indexOf(col)>-1">{{text[enEditCost.indexOf(col)].approvedAmount}}</div>
            <div v-else>
              <a-input
                class="inputC"
                v-if="index <= tasksData.length-2"
                style="text-align:center;max-width:120px"
                v-model="record.name"
                @input="changeValue(record,index,col)"
                :placeholder="placeholer[i]"
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

        <!-- 任务名称 -->
        <div slot="name" slot-scope="text,record,index" style="display: inline-block;">
          <div v-if="record.parentNode">
            <a-input
              class="inputC"
              style="text-align:center;max-width:120px"
              v-model="record.name"
              :placeholder="placeholer[0]"
            />
          </div>
          <div v-else>
            <a-input
              class="inputC"
              v-if="index <= tasksData.length-2"
              style="text-align:center;max-width:120px"
              v-model="record.name"
              :placeholder="placeholer[0]"
            />
            <div
              class="inputC"
              v-else
              style="text-align:center;line-height: 30px;font-weight:600;"
            >{{ text }}</div>
          </div>
        </div>
        <!-- 送审金额 -->
        <div slot="submitAmount" slot-scope="text,record,index">
          <div v-if="record.parentNode">
            <a-input
              class="inputC"
              style="text-align:center;max-width:120px"
              v-model="record.submitAmount"
              :placeholder="placeholer[1]"
            />
          </div>
          <div v-else>
            <a-input
              class="inputC"
              v-if="index <= tasksData.length-2"
              style="text-align:center;max-width:120px"
              v-model="record.submitAmount"
              :placeholder="placeholer[1]"
            />
            <div
              class="inputC"
              v-else
              style="text-align:center;line-height: 30px;font-weight:600;"
            >{{ text }}</div>
          </div>
        </div>
        <!-- 偏差原因 -->
        <div slot="deviationReason" slot-scope="text,record,index">
          <div v-if="record.parentNode">
            <a-input
              class="inputC"
              style="text-align:center;max-width:120px"
              v-model="record.deviationReason"
              :placeholder="placeholer[2]"
            />
          </div>
          <div v-else>
            <a-input
              class="inputC"
              v-if="index <= tasksData.length-2"
              style="text-align:center;max-width:120px"
              v-model="record.deviationReason"
              :placeholder="placeholer[2]"
            />
            <div
              class="inputC"
              v-else
              style="text-align:center;line-height: 30px;font-weight:600;"
            >{{ text }}</div>
          </div>
        </div>

        <!-- 负责人 -->
        <template slot="handler" slot-scope="text, record, index">
          <a-input
            class="inputC"
            readonly="readonly"
            style="text-align:center"
            :value="text? text.name:''"
            placeholder="负责人"
            v-if="index <= tasksData.length-2"
            @click="showEmployee(record,index)"
          />
          <a-input
            class="inputC"
            readonly="readonly"
            style="text-align:center"
            :value="text? text.name:''"
            placeholder="负责人"
            v-else-if="record.parentNode"
            @click="showEmployee(record,index)"
          />
        </template>
        <!-- 自定义操作 -->
        <template slot="operation" slot-scope="text, record, index">
          <span v-if="index <= tasksData.length-2">
            <a @click="saveRow(record)" style="padding:5px">
              <a-icon type="save" title="保存"/>
            </a>
            <!-- <a-divider type="vertical"/> -->
            <a @click="deleteTask(record,index)" style="padding:5px">
              <a-icon type="delete" title="删除"/>
            </a>
            <!-- <a-divider type="vertical" v-if="!record.parentNode"/> -->
            <a @click="addChild(record,index)" v-if="!record.parentNode" style="padding:5px">
              <a-icon type="down-circle" title="创建子任务"/>
            </a>
            <!-- <a-divider type="vertical"/> -->
            <a @click="shareTask(record,index)" v-if="record.handler" style="padding:5px">
              <a-icon type="share-alt" title="分配"/>
            </a>
          </span>
          <span v-else-if="record.parentNode">
            <a @click="saveRow(record)" style="padding:5px">
              <a-icon type="save" title="保存"/>
            </a>
            <!-- <a-divider type="vertical"/> -->
            <a @click="deleteTask(record,index)" style="padding:5px">
              <a-icon type="delete" title="删除"/>
            </a>
            <!-- <a-divider type="vertical"/> -->
            <!-- <a @click="addChild(record,index)">
              <a-icon type="down-circle" title="创建子任务"/>
            </a>-->
            <!-- <a-divider type="vertical"/> -->
            <a
              @click="shareTask(record,index)"
              v-if="record.handler && record.status == 0"
              style="padding:5px"
            >
              <a-icon type="share-alt" title="分配"/>
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

import { mapState, mapMutations } from 'vuex'

//对象转字符串
function ObjectToString(obj) {
  var type = Object.prototype.toString.call(obj)
  var result = ''
  if (type === '[object Object]') {
    result += '{'
    for (var o in obj) {
      if (Object.prototype.toString.call(obj[o]) === '[object String]') {
        result += o + ': "' + obj[o] + '",'
      } else if (Object.prototype.toString.call(obj[o]) === '[object Object]') {
        result += o + ': ' + ObjectToString(obj[o]) + ','
      } else if (Object.prototype.toString.call(obj[o]) === '[object Array]') {
        result += o + ': ' + ArrayToString(obj[o]) + ','
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
      projectName: '',
      scrollX: 1200,
      emTitle: '', //人员弹框标题
      visibled: false, //人员弹框是否显示
      isOne: true, //人员是否多选
      employeeType: 1, //部门人员弹框类型
      tasks: [], //原始数据
      tasksData: [], //处理过的数据
      docNum: 1, //稿数
      handler: {}, //负责人
      selectPId: '',
      selectIndex: 0,
      selectRecord: {},

      // costNameArr: ['name', 'submitAmount', 'deviationReason', 'numTasks1'],
      costNameArr: ['numTasks1'],
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
    console.log(this.userInfo.id, 'info')
    // console.log(this.$apollo)
  },
  computed: {
    ...mapState({
      userInfo: state => state.user.info
    })
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
                tasks(where:{type:0}) {
                  id
                  name
                  type
                  handler{
                    id
                    name
                  }
                  parentNode{
                    id
                    name
                    type
                  }
                  submitAmount
                  status
                  childTasks {
                    id
                    name
                    remark
                    type
                    submitAmount
                    approvedAmount
                    status
                    parentNode{
                      id
                      name
                      type
                    }
                    handler{
                      name
                      id
                    }
                    numTasks {
                      id
                      name
                      remark
                      status
                      submitAmount
                      approvedAmount
                      type
                      handler {
                        name
                        id
                      }
                      childTasks {
                        id
                        name
                        remark
                        status
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
                    status
                    type
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
                      status
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

    //初始化数据
    initInfo(data) {
      console.log(data)
      if (!data.length) {
        return
      }
      this.projectName = data[0].name
      this.tasks = data[0].tasks.filter(ele => ele.type == 0 && !ele.parentNode)
      this.tasksData = []
      this.costNameArr = ['numTasks1']
      this.enEditCost = ['numTasks1']
      // debugger
      this.docNum = data[0].docNum || 1
      if (this.docNum > 1) {
        var draftObj = {}
        var draftIndex = 0
        this.columns.forEach((ele, index) => {
          if (ele.title == '造价金额') {
            draftObj = ele
            draftIndex = index
            ele.children = [
              {
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
          }
        })
        var obj = { 1: '初稿', 2: '二稿', 3: '三稿', 4: '四稿', 5: '五稿', 6: '六稿', 7: '终稿' }
        var draftCL = draftObj['children'] //造价金额子表头

        draftCL[0].title = obj[1] //默认初稿添加

        if (this.docNum == this.enEditCost.length) {
          //稿数相同不用添加
          return
        }

        // debugger
        for (var i = 0; i < this.docNum - 1; i++) {
          var lastDraft = Object.assign({}, draftCL[i]) //复制
          lastDraft.scopedSlots.customRender = 'numTasks' + (i + 2) //属性唯一
          lastDraft.key = 'numTasks' + (i + 2) //属性唯一
          lastDraft.title = obj[i + 2]
          this.costNameArr.push('numTasks' + (i + 2))
          this.enEditCost.push('numTasks' + (i + 2))

          if (i + 2 == this.docNum) {
            //最后一稿不能设置title
            this.costName = lastDraft.title
            delete lastDraft.title
          }

          // delete lastDraft.slots //删除自定义表头
          draftCL.push(lastDraft)
        }

        draftObj['children'] = draftCL
        this.columns.splice(draftIndex, 1, draftObj)
        this.scrollX = 1200
        this.scrollX += this.docNum * 60

        console.log(this.columns, 'this.columns')

        // var lastDraft = Object.assign({}, draftCL[draftCLL - 1]) //复制

        // lastDraft.scopedSlots.customRender = 'numTasks' + (draftCLL + 1) //属性唯一
        // lastDraft.key = 'numTasks' + (draftCLL + 1) //属性唯一

        // this.costNameArr.push('numTasks' + (draftCLL + 1))
        // this.enEditCost.push('numTasks' + (draftCLL + 1))

        // draftCL[draftCLL - 1].title = this.costName //有添加按钮的就不能设置title
        // delete draftCL[draftCLL - 1].slots //删除自定义表头

        // this.costName = obj[draftCLL] //稿名下次使用

        // draftCL.push(lastDraft)
        // draftObj['children'] = draftCL
        // this.columns.splice(draftIndex, 1, draftObj)
        // this.scrollX += draftCLL * 20
      }

      this.beforeTable(this.tasks)
    },
    //保存
    saveRow(record) {
      console.log(record)
      var numTasks = []
      record.numTasks.forEach(ele => {
        var obj = {
          name: ele.name,
          status: 0,
          type: 1,
          project: { connect: { id: this.projectId } },
          submitAmount: parseFloat(ele.submitAmount || 0),
          createdBy: { connect: { id: this.userInfo.id } }
        }
        numTasks.push(obj)
      })

      var _this = this
      if (record.id) {
        //已经存在列表中需要更新数据库中的数据

        var needNum = record.numTasks.filter(ele => ele.status == 0)
        console.log()

        this.$apollo
          .mutate({
            mutation: gql`
            mutation {
              updateTask(data:{
                handler:{connect:{id:"${record.handler.id}"}}
                name:"${record.name}"
                submitAmount:${record.submitAmount}
              },where:{id:"${record.id}"})
              {id}
            }
          `
          })
          .then(res => {
            console.log(res, 'update handler')
            if (needNum.length) {
              needNum.forEach((ele, numI) => {
                this.$apollo
                  .mutate({
                    mutation: gql`
                      mutation {
                        updateTask(data:{
                          handler:{connect:{id:"${record.handler.id}"}}
                          name:"${ele.name}"
                          submitAmount:${ele.submitAmount}
                        },where:{id:"${ele.id}"})
                        {id}
                      }
                    `
                  })
                  .then(res => {
                    console.log(res, 'update childdler')
                    if (_this.needNum.length == numI + 1) {
                      _this.loadData(_this.projectId)
                    }
                  })
                  .catch(err => {})
              })
            } else {
              _this.loadData(_this.projectId)
            }
          })
          .catch(err => {})
      } else {
        if (record.parentNode.id) {
          //如果是子任务
          // console.log(`
          //     mutation {
          //       createTask(data: {
          //         name:"${record.name}"
          //         parentNode:{connect:{id:"${record.parentNode.id}"}}
          //         project:{connect:{id:"${_this.projectId}"}}
          //         submitAmount:${record.submitAmount || 0}
          //         type:0,
          //         numTasks: {
          //           create:${ArrayToString(numTasks)}
          //         }
          //         createBy: {connect: {id: "${_this.userInfo.id}"}}
          //       }){id,numTasks}
          //     }
          //   `)
          this.$apollo
            .mutate({
              mutation: gql`
              mutation {
                createTask(data: {
                  name:"${record.name}"
                  parentNode:{connect:{id:"${record.parentNode.id}"}}
                  project:{connect:{id:"${_this.projectId}"}}
                  submitAmount:${record.submitAmount || 0}
                  type:0,
                  status:0,
                  numTasks: {
                    create:${ArrayToString(numTasks)}
                  }
                  createdBy: {connect: {id: "${_this.userInfo.id}"}}
                }){id}
              }
            `
            })
            .then(res => {
              console.log(res)
              _this.loadData(_this.projectId)
            })
            .catch(err => {
              console.log(err, 'saveRow')
            })
        } else {
          //夫任务
          console.log(`
              mutation {
                createTask(data: {
                  name:"${record.name}"
                  project:{connect:{id:"${_this.projectId}"}}
                  submitAmount:${record.submitAmount}
                  type:0,
                  numTasks: {
                    create:${ArrayToString(numTasks)}
                  },
                  createdBy: {connect: {id:"${_this.userInfo.id}"}}
                }){id}
              }
            `)
          this.$apollo
            .mutate({
              mutation: gql`
              mutation {
                createTask(data: {
                  name:"${record.name}"
                  project:{connect:{id:"${_this.projectId}"}}
                  submitAmount:${record.submitAmount || 0}
                  type:0,
                  status:0,
                  numTasks: {
                    create:${ArrayToString(numTasks)}
                  },
                  createdBy:{connect: {id: "${_this.userInfo.id}"}}
                }){id}
              }
              
              `
            })
            .then(res => {
              _this.loadData(_this.projectId)
            })
            .catch(err => {
              console.log(err, 'saveRow')
            })
        }

        //不存在则添加到数据库中
      }
    },

    //删除任务
    deleteTask(record, index) {
      console.log(record)
      if (record.id) {
        //已经存在列表中需要删除数据库中的数据
      } else {
        //不存在则直接删除数据
        this.tasks.splice(index, 1)
        // this.tasks.push(obj)
        this.beforeTable(this.tasks)
      }
      // const target = this.data.filter(item => item.key === key)[0]
      // target.editable = false
    },

    //添加子任务
    addChild(record, index) {
      var obj = {
        name: '',
        handler: {},
        project: { id: this.projectId },
        parentNode: { id: record.id },
        type: 0,
        submitAmount: record.submitAmount,
        numTasks: []
      }
      if (this.docNum == 1) {
        obj.numTasks = [
          {
            name: '初稿',
            handler: {},
            project: { id: this.projectId },
            type: 1,
            submitAmount: ''
          }
        ]
      } else {
        for (var i = 0; i < this.docNum; i++) {
          var objName = { 1: '初稿', 2: '二稿', 3: '三稿', 4: '四稿', 5: '五稿', 6: '六稿', 7: '终稿' }
          var numTobj = {
            name: objName[i + 1],
            handler: {},
            project: { id: this.projectId },
            type: 1,
            status: 0,
            submitAmount: ''
          }
          // debugger
          obj.numTasks.push(numTobj)
        }
      }
      record.childTasks.push(obj)
      // console.log(obj, 'ob--------------')
      this.tasks.splice(index, 1, record)
      this.beforeTable(this.tasks)
    },

    //分配任务
    shareTask(record) {
      this.$confirm({
        title: '分配任务',
        content: `你确定分配任务【${record.name}】吗？`,
        onOk() {
          var numT = record.numTasks.filter(ele => ele.status == 0)
          var _this = this

          console.log(record, 'shareTask', numT)

          numT.forEach((ele, i) => {
            this.$apollo
              .mutate({
                mutation: gql`
            mutation {
              updateTask(data:{
                status:1
              },where:{id:"${ele.id}"})
              {id}
              createNotification(data: {
                type: 2
                name: "【郑会龙】分配了【${_this.projectName}】项目中的【${ele.name}】任务给您，任务截止日期 2019-01-23"
                status: 0
                recipients: {
                  connect: {id:"${record.handler.id}"}
                }
              }){
              id 
              }
            }
          `
              })
              .then(res => {
                // console.log(res,'update fepei')
                if (i == numT.length - 1) {
                  _this.loadData(_this.projectId)
                }
              })
              .catch(err => {})
          })
        },
        onCancel() {}
      })
    },

    //添加稿数
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

      this.docNum = draftCLL + 1

      var costObj = { 1: '初稿', 2: '二稿', 3: '三稿', 4: '四稿', 5: '五稿', 6: '六稿', 7: '终稿' }
      var _costName = costObj[this.docNum]

      var _this = this

      this.$apollo
        .mutate({
          mutation: gql`
          mutation {
            updateProject(data: { docNum: ${this.docNum} }, where: { id: "${this.projectId}" }) {
              docNum
              id
            }
          }
        `
        })
        .then(res => {
          console.log(res, 'update docNum')
          //父级任务创建稿数
          _this.tasks.forEach((ele, index) => {
            _this.$apollo
              .mutate({
                mutation: gql`
                  mutation {
                    createTask (data:{
                      name:"${_costName}"
                      submitAmount:${parseFloat(0)}
                      project:{connect:{id:"${_this.projectId}"}}
                      numParentNode:{connect:{id:"${ele.id}"}}
                      ${ele.handler ? `handler:{connect:{id:"${ele.handler.id}"}}` : ''}
                      type:1
                      status:0,
                      createdBy: {connect: {id: "${_this.userInfo.id}"}}
                    }){id}
                  }
                `
              })
              .then(res => {
                console.log('create parent')
                //子任务创建稿数
                if (ele.childTasks.length) {
                  ele.childTasks.forEach((e, i) => {
                    _this.$apollo
                      .mutate({
                        mutation: gql`
                            mutation {
                              createTask (data:{
                                name:"${_costName}"
                                submitAmount:${parseFloat(e.submitAmount)}
                                project:{connect:{id:"${_this.projectId}"}}
                                numParentNode:{connect:{id:"${e.id}"}}
                                ${e.handler ? `handler:{connect:{id:"${e.handler.id}"}}` : ''}
                                type:1
                                status:0
                                createdBy: {connect: {id: "${_this.userInfo.id}"}}
                              }){id}
                            }
                          `
                      })
                      .then(res => {
                        console.log(res, 'create child')
                        if (_this.tasks.length == index + 1 && ele.childTasks.length == i + 1) {
                          _this.loadData(_this.projectId)
                        }
                      })
                      .catch(err => {
                        console.log(err)
                      })
                  })
                } else {
                  if (_this.tasks.length == index + 1) {
                    _this.loadData(_this.projectId)
                  }
                }
              })
              .catch(err => {
                console.log(err)
              })
          })
          // this.tasks.childTasks
        })
        .catch(err => {
          console.log(err)
        })
    },

    showEmployee(record, index) {
      console.log(record)
      if (record.parentNode) {
        this.selectPId = record.parentNode.id
      }

      ;(this.selectIndex = index), (this.selectRecord = record)
      // selectRecord:{},
      this.visibled = true
    },
    changeEMStatus(obj) {
      this.visibled = obj.visibled
      if (obj.data.length) {
        this.handler = {
          id: obj.data[0].key,
          name: obj.data[0].title
        }
      } else {
        return
      }
      this.selectRecord.handler = this.handler
      if (this.selectPId) {
        var _i = 0
        this.tasks.forEach((e, i) => {
          if (e.id == this.selectPId) {
            _i = i
          }
        })
        var Arr = this.tasks[_i]
        Arr[this.selectIndex] = this.selectRecord
        this.tasks.splice(_i, 1, Arr)
      } else {
        this.tasks.splice(this.selectIndex, 1, this.selectRecord)
      }

      console.log(obj)
    },

    changeValue(record, col, index) {
      console.log(record, col, index)
    },

    //添加父任务
    addCost() {
      var obj = {
        name: '',
        handler: {},
        project: { id: this.projectId },
        parentNode: {},
        type: 0,
        submitAmount: '',
        numTasks: []
      }
      if (this.docNum == 1) {
        obj.numTasks = [
          {
            name: '初稿',
            handler: {},
            project: { id: this.projectId },
            type: 1,
            submitAmount: ''
          }
        ]
      } else {
        for (var i = 0; i < this.docNum; i++) {
          var objName = { 1: '初稿', 2: '二稿', 3: '三稿', 4: '四稿', 5: '五稿', 6: '六稿', 7: '终稿' }
          var numTobj = {
            name: objName[i + 1],
            handler: {},
            project: { id: this.projectId },
            type: 1,
            status: 0,
            submitAmount: ''
          }
          // debugger
          obj.numTasks.push(numTobj)
        }
      }
      console.log(obj, 'ob--------------')
      this.tasks.push(obj)
      this.beforeTable(this.tasks)
    },

    beforeTable(data) {
      var obj = {
        name: '汇总',
        submitAmount: 0,
        deviationReason: '',
        deviationRate: '',
        numTasks: []
      }

      for (var i = 0; i < this.docNum + 1; i++) {
        obj.numTasks.push({ submitAmount: 0 })
      }

      //复制数组
      this.tasksData = []
      data.forEach(ele => {
        this.tasksData.push(ele)
      })

      //获取子任务
      this.tasksData.forEach((ele, index) => {
        if (ele.childTasks) {
          if (ele.childTasks.length) {
            ele.children = ele.childTasks
            ele.key = index
            if (ele.children.length) {
              ele.children.forEach((e, i) => {
                e.key = index + '' + i
              })
            }
          }
        }
      })

      this.tasksData.forEach(ele => {
        if (ele.childTasks.length) {
          //存在子任务
          ele.childTasks.forEach((tE, ti) => {
            tE.numTasks.forEach((tcE, tci) => {
              tcE.approvedAmount = 0
              if (tcE.childTasks.length) {
                //获取审核后的最后一个审定金额
                tcE.approvedAmount += tcE.childTasks[tcE.childTasks.length - 1].approvedAmount
              }
            })
          })

          ele.numTasks.forEach((nE, ni) => {
            //父任务稿件审定金额由子任务稿件审定金额的叠加
            nE.approvedAmount = 0
            ele.childTasks.forEach((tE, ti) => {
              nE.approvedAmount += tE.numTasks[ni].approvedAmount
            })
          })
        } else {
          //不存在子任务
          ele.numTasks.forEach((nE, ni) => {
            nE.approvedAmount = 0
            if (nE.childTasks.length) {
              //获取审核后的最后一个审定金额
              nE.approvedAmount += nE.childTasks[nE.childTasks.length - 1].approvedAmount
            }
          })
        }
      })

      //汇总
      this.tasksData.push(obj)
      console.log(this.tasksData, this.tasks, 'tasksssssssss')
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

