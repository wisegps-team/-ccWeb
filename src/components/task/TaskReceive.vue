<template>
  <div class="taskReceive">
    <a-row>
      <a-col :span="24" class="taskReceiveTable">
        <a-table :columns="columns" :dataSource="current" bordered :pagination="pagination">
          <span slot="name" slot-scope="text, record,index">
            <a class="font-blue">{{ text }}</a>
          </span>
          <div slot="checkFile" slot-scope="text, record,index">
            <a class="font-blue" style="margin-right:5px">查看</a>
          </div>
          <div slot="action" slot-scope="text,record,index">
            <a v-if="(record.type===0||record.type===1)&&record.status===1" class="font-blue" style="margin-right:5px" @click="acceptTask(record)">接受</a>
            <a v-if="(record.type===0||record.type===1)&&record.status===1" class="font-blue" style="margin-right:5px" @click="rejectTask(record)">拒绝</a>
            <a v-if="record.status===2&&record.status===4" class="font-blue" style="margin-right:5px" @click="commentTask(record)">讨论</a>
            <a v-if="(record.type===0||record.type===1)&&(record.status===2||record.status===5)" class="font-blue" style="margin-right:5px" @click="submitTask(record)">提交</a>
            <a v-if="(record.type===0||record.type===1)&&record.status > 2" class="font-blue" style="margin-right:5px" @click="finishTask(record)">完成</a>
            <a v-if="(record.type===2||record.type===3)&&record.status===4" class="font-blue" style="margin-right:5px" @click="reviewTask(record)">审核</a>
            <a v-if="(record.type===2||record.type===3)&&record.status===4" class="font-blue" style="margin-right:5px" @click="taskRemark(record)">纪要</a>
            <a class="font-blue" style="margin-right:5px" @click="goProject(record)">项目</a>
          </div>
        </a-table>
      </a-col>
    </a-row>
    <TaskSubmit inititle="提交审核" :initvisibled="openSubmit" @submitSave2="submitSave" @submitCancel="submitCancel"/>
    <TaskReview inititle="审核" :initvisibled="openReview" @reviewSave="reviewSave" @reviewCancel="reviewCancel" :currentTask="currentTask"/>
  </div>
</template>
<script>
import gql from 'graphql-tag'
import TaskSubmit from '@/components/dialog/task/TaskSubmitDialog'
import TaskReview from '@/components/dialog/task/TaskReviewDialog'
import moment from 'moment'

const columns = [
  {
    title: '任务名称',
    width: 200,
    align: 'center',
    key: 'name',
    dataIndex: 'name'
  },
  {
    title: '发起人',
    width: 80,
    align: 'center',
    key: 'createdBy',
    dataIndex: 'createdBy.name'
  },
  {
    title: '负责人',
    width: 80,
    align: 'center',
    key: 'handler',
    dataIndex: 'handler.name'
  },
  {
    title: '文件',
    width: 80,
    align: 'center',
    key: 'id',
    dataIndex: 'id',
    scopedSlots: { customRender: 'checkFile' }
  },
  {
    title: '截止日期',
    width: 80,
    align: 'center',
    key: 'endDate',
    dataIndex: 'endDate',
    customRender: (value) => {
      var d = ''
      if(value){
        d = moment(value).format('YYYY-MM-DD')
      }
      return  {
        children: d,
        attrs: {}
      }
    }
  },
  {
    title: '任务状态',
    width: 80,
    align: 'center',
    key: 'status',
    dataIndex: 'status',
    customRender: (value, row, index) => {
        var desc = ''
        console.log(value)
        switch (value) {
          case 0:
            desc = '未分配'
            break
          case 1:
            desc = '已分配'
            break
          case 2:
            desc = '已接受'
            break
          case 3:
            desc = '已拒绝'
            break
          case 4:
            desc = '审核中'
            break
          case 5:
            desc = '审核未通过'
            break
          case 6:
            desc = '审核通过'
            break
          case 7:
            desc = '已完成'
            break
        }
        return  {
          children: desc,
          attrs: {},
        }
    }
  },
  {
    title: '操作',
    width: 180,
    align: 'center',
    key: 'action',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' }
  }
]

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
  name: 'TaskReceive',
  components: {
    TaskSubmit,
    TaskReview
  },
  props: {
    tapType: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      openSubmit: false,
      openReview: false,
      columns,
      taskType: this.tapType,
      //   pagination: this.pager(),
      current: [],
      pagination: {},
      pageSizeOptions: ['10', '20', '30'],
      currentPageSize: 10,
      defaultCurrent: 1,
      total: 20,
      user: {},
      currentTask: {}
    }
  },
  created() {
    console.log(this.taskType, 'type')
    this.loading = true
    this.loadList()
  },
  methods: {
    init() {
      console.log(this.taskType)
    },
    //接口获取数据
    loadList() {
      const _this = this
      const skip = (this.defaultCurrent -1) * this.currentPageSize
      this.$apollo
        .query({
          query: gql`
            query getTasks($skip: Int!, $pageSize: Int!)  {
              tasksConnection (where: { handler: { id: "cjr66m91v00to07359nnkrjmu" }, type_in: [0, 1, 2, 3], status_gt: 0 }) {
                aggregate {
                  count
                }
              }

              tasks(
                where: { handler: { id: "cjr66m91v00to07359nnkrjmu" }, type_in: [0, 1, 2, 3], status_gt: 0 }, skip: $skip, first: $pageSize
              ) {
                id
                handler {
                  id
                  name
                }
                type
                step
                name
                remark
                submitAmount
                approvedAmount
                status
                endDate
                createdBy {
                  id
                  name
                }
                createdAt
                project {
                  id
                  name
                }
                parentNode {
                  id
                  name
                }
                numParentNode {
                  id
                  name
                }
              }
            }
          `,
          variables: {
            skip: parseInt(skip),
            pageSize: parseInt(this.currentPageSize),
          },
          fetchPolicy: 'network-only'
        })
        .then(res => {
          console.log(res,'res')
          _this.total = res.data.tasksConnection.aggregate.count
          _this.current = res.data.tasks
          _this.pagination = _this.pager()
          _this.loading = res.loading
        })
        .catch(err => {
          console.log(err)
          _this.total = 0
          _this.defaultCurrent = 1
          _this.current = []
          _this.pagination = _this.pager()
          _this.loading = false
        })
    },
    onPagerChange(page, pageSize) {
      this.defaultCurrent = page
      this.currentPageSize = pageSize
      this.loading = true
      this.loadList()
    },
    onPagerSizeChange(current, size) {
      this.defaultCurrent = current
      this.currentPageSize = size
      this.loading = true
      this.loadList()
    },
    pager() {
      return {
        total: this.total,
        showTotal: total => `共有 ${total} 条`,
        showSizeChanger: true,
        pageSizeOptions: this.pageSizeOptions,
        pageSize: this.currentPageSize,
        defaultCurrent: this.defaultCurrent,
        onChange: this.onPagerChange,
        onShowSizeChange: this.onPagerSizeChange
      }
    },
    acceptTask(record) {
      const _this = this
      this.$confirm({
        title: '接受任务',
        content: `你确定接受任务【${record.name}】吗？`,
        onOk() {
          console.log('接受任务', record)
          _this.updateTaskStatus(record, 2)
        },
        onCancel() {
        }
      })
    },
    rejectTask(record) {
      const _this = this
      this.$confirm({
        title: '拒绝任务',
        content: `你确定拒绝任务【${record.name}】吗？`,
        onOk() {
          console.log('拒绝任务', record)
          _this.updateTaskStatus(record, 3)
        },
        onCancel() {
        }
      })
    },
    finishTask(record) {
      const _this = this
      this.$confirm({
        title: '完成任务',
        content: `你确定完成任务【${record.name}】吗？`,
        onOk() {
          console.log('完成任务', record)
          _this.updateTaskStatus(record, 7)
        },
        onCancel() {
        }
      })
    },
    commentTask(record) {
      console.log(record.comments)
    },
    submitTask(record) {
      console.log(record)
      this.currentTask = record
      this.openSubmit = true
    },
    reviewTask(record) {
      console.log(record)
      this.currentTask = record
      this.openReview = true
    },
    goProject(record) {
      console.log(record)
      alert(`${record.project.id}, ${record.project.name}`)
    },
    taskRemark(record){
      alert(record.remark)
    },
    submitSave(data) {
      console.log(data)
      if(data){
        const _this = this
        this.$apollo
          .mutate({
            mutation: gql`
              mutation {
                updateTask(data: { status: 4 }, where: { id: "${_this.currentTask.id}" }) {
                  id
                }

                createTask(
                  data: {
                    project: { connect: { id: "${_this.currentTask.project.id}" } }
                    parentNode: { connect: { id: "${_this.currentTask.id}" } }
                    type: ${data.type}
                    handler: { connect: { id: "${data.handler.id}" } }
                    name: "【${_this.currentTask.type == 0 ?_this.currentTask.parentNode.name: _this.currentTask.numParentNode.name}】${_this.currentTask.name}"
                    step: ${data.step}
                    submitAmount: ${data.submitAmount}
                    approvedAmount: ${data.approvedAmount}
                    unitPrice: ${data.unitPrice}
                    remark: "${data.remark}"
                    status: 4
                    reviewResult: -1
                  }
                ) {
                  id
                }
              }
            `
          })
          .then(res => {
            console.log(res,'updateTaskStatus')
            _this.currentTask.status = 4
            _this.openSubmit = false
          }).catch(err => {
          console.log(err, 'err')
        })
      }
    },
    submitCancel(){
      this.openSubmit = false
    },
    reviewSave(data) {
      if(data){
        const _this = this
        if(data.isShowNextReview){ //发到下一级审核
          const reviewStatus = data.reviewResult === 1 ? 6: 5
          const status = data.reviewResult === 1 ? 4: 5
          this.$apollo
            .mutate({
              mutation: gql`
                mutation {
                  # 更新审核任务的审核意见为同意或者不同意，状态为审核通过/不通过
                  updateTask(
                    data: {
                      status: ${reviewStatus},
                      reviewResult: ${data.reviewResult},
                      reviewComment: "${data.reviewComment}",
                      reviewDate: "${data.reviewDate}"
                    },
                    where: {
                      id: "${_this.currentTask.id}"
                    }
                  ) {
                    id
                  }

                  # 更新父任务的状态为审核通过/不通过
                  ${_this.currentTask.parentNode ?`udpateTask2:updateTask(
                    data: {
                      status: ${status}
                    }, where: {
                      id: "${_this.currentTask.parentNode.id}"
                    }
                  ) {
                    id
                  }`:''}

                  # 新增下一级审核任务
                  createTask(
                    data: {
                      project: { connect: { id: "${_this.currentTask.project.id}" } }
                      ${_this.currentTask.parentNode?`parentNode: { connect: { id: "${_this.currentTask.parentNode.id}" } }`:''}
                      type: ${data.type}
                      handler: { connect: { id: "${data.nextHandler.id}" } }
                      name: "${_this.currentTask.name}"
                      step: ${data.nextStep}
                      submitAmount: ${data.submitAmount}
                      approvedAmount: ${data.approvedAmount}
                      unitPrice: ${data.unitPrice || 0}
                      remark: "${data.remark}"
                      status: 4
                      reviewResult: -1
                    }
                  ) {
                    id
                  }
                }
              `
            })
            .then(res => {
              console.log(res,'updateTaskStatus')
              _this.currentTask.status = reviewStatus
              this.openReview = false
            }).catch(err => {
            console.log(err, 'err')
          })
        }else{ //审核结束
          const status = data.reviewResult === 1 ? 6: 5
          this.$apollo
            .mutate({
              mutation: gql`
                mutation {
                  # 更新审核任务的审核意见为同意或者不同意，状态为审核通过/不通过
                  updateTask(
                    data: {
                      status: ${status},
                      reviewResult: ${data.reviewResult},
                      reviewComment: "${data.reviewComment}",
                      reviewDate: "${data.reviewDate}"
                    },
                    where: {
                      id: "${_this.currentTask.id}"
                    }
                  ) {
                    id
                  }

                  # 更新父任务的状态为审核通过/不通过
                 ${_this.currentTask.parentNode ? `updateTask2:updateTask(data: { status: ${status} }, where: { id: "${_this.currentTask.parentNode.id}" }) {
                    id
                  }`:''} 
                }
              `
            })
            .then(res => {
              console.log(res,'updateTaskStatus')
              _this.currentTask.status = status
              this.openReview = false
            }).catch(err => {
            console.log(err, 'err')
          })
        }
      }
    },
    reviewCancel(){
      this.openReview = false
    },
    updateTaskStatus(record, status){
      record.status = status
      let notification = ''
      let recipients = []
      switch (status){
        case 2: //接受
          notification = `【${record.handler.name}】接受了【${record.project.name}】项目中的【${record.name}】任务`
          recipients = [record.createdBy.id]
          break
        case 3: //拒绝
          notification = `【${record.handler.name}】拒绝了【${record.project.name}】项目中的【${record.name}】任务`
          recipients = [record.createdBy.id]
          break
        case 5: //审核通过
          if(record.type === 2){ //任务审核
            notification = `【${record.handler.name}】审核通过了【${record.project.name}】项目中的【${record.name}】任务，审核意见为【${record.reviewComment}】`
            recipients = [record.createdBy.id]
          }else{ //项目多级复核
            notification = `【${record.handler.name}】拒绝了【${record.project.name}】项目中的【[多]${record.name}】任务`
            recipients = [record.createdBy.id]
          }
          break
        case 6: //审核不通过
          if(record.type === 2){ //任务审核
            notification = `【${record.handler.name}】审核未通过了【${record.project.name}】项目中的【${record.name}】任务，审核意见为【${record.reviewComment}】`
            recipients = [record.createdBy.id]
          }else{ //项目多级复核
            notification = `【${record.handler.name}】拒绝了【${record.project.name}】项目中的【[多]${record.name}】任务`
            recipients = [record.createdBy.id]
          }
          break
      }

      this.$apollo
        .mutate({
          mutation: gql`
            mutation {
              updateTask(
                data: {
                  status: ${status}
                }
                where: { id: "${record.id}" }
              ) {
                id
              }

              createNotification(
                data: {
                  type: 3
                  name: "${notification}"
                  status: 0
                  recipients: { connect: ${ArrayToString(recipients)} }
                }
              ) {
                id
              }
            }
          `
        })
        .then(res => {
          console.log(res,'updateTaskStatus')
        }).catch(err => {
          console.log(err, 'err')
      })
    }
  },
  watch: {
    tapType(newT, oldT) {
      // var that = this
      // this.
      // getData()
      // console.log(this.$apollo, 'use')
      console.log(this)
      this.taskType = newT
      this.init()
    }
  }
}
</script>
<style>
.taskReceiveTable .ant-table-thead > tr > th,
.taskReceiveTable .ant-table-tbody > tr > td {
  padding: 8px 16px;
}
</style>

