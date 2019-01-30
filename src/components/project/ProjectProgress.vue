<template>
  <div style="padding:10px">
    <div style="margin-bottom:20px">
      <table class="form-table" width="100%" cellspacing="0" cellpadding="0">
        <tbody>
          <tr>
            <!--ms-if-->
            <td width="200">
              负责人：
              <span class="font16">
                <img
                  class="project-leader"
                  style="cursor: pointer;"
                  src="http://tvips.glodon.com/api/org/emp/downHeadImgByEmployeeIdNew/2586"
                >
                <span class="font16">
                  <input
                    class="font-blue nofocus"
                    type="text"
                    readonly="readonly"
                    :value="principal.name"
                    style="border-style: none; width: 90px; cursor: pointer;"
                    @click="showEmployee(1)"
                  >
                </span>
              </span>
            </td>

            <td width="210">
              开始日期：
              <span class="ui-date">
                <a-date-picker style="width:120px"/>
              </span>
            </td>
            <td width="210">
              截止日期：
              <span class="ui-date">
                <a-date-picker style="width:120px"/>
              </span>
            </td>
            <td width="80">
              <div class="ui-dropdown">
                <a-select value="2">
                  <a-select-option value="2">工作日</a-select-option>
                  <a-select-option value="1">自然日</a-select-option>
                </a-select>
              </div>
            </td>
            <td width="240" class="font-center">
              <div class="project-date">
                <b class="current-date font-red">4</b>
                <span>/</span>
                <b class="all-date">4</b>
                <i>天</i>
              </div>
            </td>
            <!-- <a-icon type="check-circle" />已完成 -->
            <!-- <a-icon type="clock-circle" /> 进行中-->
            <!-- <a-icon type="minus-circle" /> 未开始 -->
            <!-- <a-icon type="exclamation-circle" /> 延期/延误 -->
            <!-- <a-icon type="pause-circle" /> 暂停 -->
            <!-- <a-icon type="save" /> 已归档 -->
            <td width="240">
              <div
                id="select-project_status"
                class="project-complete"
                :style="{backgroundColor:nowStatusColor}"
              >
                <div class="select-box">
                  <div class="select-content" @click="showStatusList">
                    <span class="select-contentL">
                      <a-icon type="check-circle"/>
                      <span style="position: relative; top: -1px;">{{ nowStatus }}</span>
                    </span>
                    <a-icon type="down"/>
                  </div>
                </div>
                <!--ms-if-->
                <div class="select-list" ref="statusDiv" v-if="statusListShow">
                  <a-row>
                    <a-col
                      :span="25"
                      v-for="(statusItem,index) in statusArr"
                      :key="index"
                      :style="{display: statusItem.name != nowStatus ? 'block':'none'}"
                    >
                      <a
                        :value="statusItem.value"
                        @click="StatusListClick(statusItem)"
                        style="color:#fff;font-size:18px;display: block;padding: 10px;"
                      >
                        <a-icon :type="statusItem.icon" style="margin-right: 5px;"/>
                        {{ statusItem.name }}
                      </a>
                    </a-col>
                  </a-row>
                </div>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div style="margin-bottom:20px" id="steps">
      <h3>
        <span style="margin-right:20px">项目进展</span>
        <span style="font-size:14px;color:#5873c9;cursor:pointer">
          <a-icon type="edit"/>编辑节点
        </span>
      </h3>
      <div class="project-flow" ref="flowDiv">
        <a-row v-if="steps.length < 6" class="project-flow_l">
          <a-col class="flow-item" :span="4" v-for="(items,index) in steps" :key="index" @mouseenter="flowMove(index)" @mouseleave="removeMove">
              <div class="flow-status" :style="{visibility: move == index ? 'unset':'hidden'}" >
                <a title="已完成" class="font-blue" :style="{display: items.status == 3 ? 'none':'inline-block'}">
                  <a-icon type="check-circle" style="width: 18px;"/>
                </a>
                <a title="进行中" class="font-green" :style="{display: items.status == 1 ? 'none':'inline-block'}">
                  <a-icon type="clock-circle" style="width: 18px;"/>
                </a>
                <a title="暂停" class="font-red" :style="{display: items.status == 2 ? 'none':'inline-block'}">
                  <a-icon type="pause-circle" style="width: 18px;"/>
                </a>
              </div>
              <dl class="flow-box">
                <dt title="审核类">
                  <i>{{ items.order }}</i>{{ items.name }}
                </dt>
                <dd>
                  <i>计</i>
                  <input type="text" readonly="readonly" placeholder="点击选择日期" class="flow-time">
                </dd>
              </dl>
              <a class="flow-arrow" style="font-size:20px" v-if="index < steps.length-1">
                <a-icon type="left"/>
              </a>
            </a-col>
        </a-row>
        <div v-else>
          <!-- 单双行 -->
          <a-row v-for="item in Math.ceil(steps.length/6)" :key="item" :class="[item%2 == 0 ?'project-flow_r':'project-flow_l']">
            <!-- 循环遍历项目进展在范围内 -->
            <a-col class="flow-item" :span="4" v-for="(items,index) in steps" :key="index" v-if="item*6 >index && (item-1)*6 <= index" @mouseenter="flowMove(index)" @mouseleave="removeMove">
              <div class="flow-status" :style="{visibility: move == index ? 'unset':'hidden'}" >
                <a title="已完成" class="font-blue" :style="{display: items.status == 3 ? 'none':'inline-block'}">
                  <a-icon type="check-circle" style="width: 18px;"/>
                </a>
                <a title="进行中" class="font-green" :style="{display: items.status == 1 ? 'none':'inline-block'}">
                  <a-icon type="clock-circle" style="width: 18px;"/>
                </a>
                <a title="暂停" class="font-red" :style="{display: items.status == 2 ? 'none':'inline-block'}">
                  <a-icon type="pause-circle" style="width: 18px;"/>
                </a>
              </div>
              <dl class="flow-box">
                <dt title="审核类">
                  <i>{{ items.order }}</i>{{ items.name }}
                </dt>
                <dd>
                  <i>计</i>
                  <input type="text" readonly="readonly" placeholder="点击选择日期" class="flow-time">
                </dd>
              </dl>
              <a class="flow-arrow" style="font-size:20px" v-if="index < steps.length-1">
                <a-icon v-if="(index+1)%6 == 0" type="down"/>
                <a-icon v-else :type="item%2 == 0 ? 'left':'right'"/>
              </a>
            </a-col>
          </a-row>
        </div>
      </div>
    </div>
    <div style="margin-bottom:20px">
      <h3>项目计划</h3>
      <a-table :columns="columns" :dataSource="data" bordered size="small" :pagination="false">
        <!-- <template slot="name" slot-scope="text">
          <a href="javascript:;">{{ text }}</a>
        </template>-->
        <div slot="name" slot-scope="text,record,index">
          <a-input v-model="record.name"/>
        </div>
        <div slot="startDate" slot-scope="text,record,index">
          <a-date-picker/>
        </div>
        <div slot="endDate" slot-scope="text,record,index">
          <a-date-picker/>
        </div>
        <div slot="principal" slot-scope="text,record,index">
          <a-input v-model="record.principal" readonly="readonly"/>
        </div>
        <div slot="members" slot-scope="text,record,index">
          <a-input v-model="record.members" readonly="readonly"/>
        </div>
        <div slot="status" slot-scope="text,record,index">
          <a-select v-model="record.status" style="width:100px">
            <a-select-option key="0" :value="0">未开始</a-select-option>
            <a-select-option key="1" :value="1">进行中</a-select-option>
            <a-select-option key="2" :value="2">已完成</a-select-option>
            <a-select-option key="3" :value="3">延期</a-select-option>
          </a-select>
        </div>
        <div slot="remark" slot-scope="text,record,index">
          <a-input v-model="record.remark"/>
        </div>
        <div slot="action" slot-scope="text,record,index">
          <a-icon
            type="save"
            title="保存计划"
            class="font-blue"
            style="width: 25px;line-height: 25px;cursor:pointer;color:#999"
          />
          <a-icon
            type="close-circle"
            title="取消编辑"
            style="width: 25px;line-height: 25px;cursor:pointer;color:#999"
          />
          <a-icon
            type="delete"
            title="删除计划"
            style="width: 25px;line-height: 25px;cursor:pointer;color:#999"
          />
        </div>
      </a-table>
      <div style="font-size:10px;color:#5873c9;cursor:pointer" @click="addPlans">
        <a-icon type="plus"></a-icon>添加计划项
      </div>
    </div>
    <div class="progress">
      <h3>项目纪要</h3>
      <a-timeline mode="left" :pending="true">
        <a-icon slot="pendingDot" type="clock-circle-o" style="font-size: 0px;"/>

        <a-timeline-item v-for="(value, key, index) in summarysData" :key="index">
          <p style="font-size: 15px;color: #000;">{{ key }}</p>
          <p v-for="val in value" :key="val.id" style="font-size: 12px;">
            <span style="color: #999;margin-right: 10px;">{{ val.time }}</span>
            <span style="color: #000;font-weight:600 ;margin-right: 5px;">{{ val.createdBy }}</span>
            <span>{{ val.content }}</span>
          </p>
          <!-- <p>这个一项事项</p>
          <p>这是二项事项</p>
          <p>这是二项事项</p> -->
        </a-timeline-item>
        <!-- <a-timeline-item>
          <p>2015-09-01</p>
          <p>这个一项事项</p>
        </a-timeline-item> -->
      </a-timeline>
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
// id,name,status,remark,endDate,startDate,principal,members

import gql from 'graphql-tag'
import moment from 'moment'

import employeeTree from '@/components/same/employeeTree'

const columns = [
  {
    title: '计划项',
    dataIndex: 'name',
    width: 150,
    scopedSlots: { customRender: 'name' }
  },
  {
    title: '开始时间',
    dataIndex: 'startDate',
    width: 150,
    scopedSlots: { customRender: 'startDate' }
  },
  {
    title: '截止时间',
    dataIndex: 'endDate',
    width: 150,
    scopedSlots: { customRender: 'endDate' }
  },
  {
    title: '负责人',
    dataIndex: 'principal',
    width: 150,
    scopedSlots: { customRender: 'principal' }
  },
  {
    title: '参与人',
    dataIndex: 'members',
    width: 150,
    scopedSlots: { customRender: 'members' }
  },
  {
    title: '完成情况',
    dataIndex: 'status',
    width: 150,
    scopedSlots: { customRender: 'status' }
  },
  {
    title: '备注',
    width: 150,
    dataIndex: 'remark',
    scopedSlots: { customRender: 'remark' }
  },
  {
    title: '',
    dataIndex: 'action',
    width: 100,
    scopedSlots: { customRender: 'action' }
  }
]

export default {
  name: 'ProjectProgress',
  props: {
    projectId: {
      type: String,
      defalut: ''
    }
  },
  data() {
    return {
      data: [],
      columns,
      nowStatus: '已完成',
      statusListShow: false,
      nowStatusColor: '#96ca85',
      emTitle: '', //人员弹框标题
      visibled: false, //人员弹框是否显示
      isOne: true, //人员是否多选
      employeeType: 1, //部门人员弹框类型
      type: 1, //结果类型

      principal: {}, //负责人
      steps: [], //项目进展
      summarys: [], //项目纪要
      summarysData: {},//处理过的数据
      plans: [], //项目计划
      endData: '', //开始时间
      startData: '', //截止时间
      status: '',
      move:-1,
      statusArr: [
        {
          name: '未开始',
          value: 0,
          background: '#78bb61',
          icon: 'minus-circle'
        },
        {
          name: '进行中',
          value: 2,
          background: '#78bb61',
          icon: 'clock-circle'
        },
        {
          name: '已完成',
          value: 3,
          background: '#5d9cec',
          icon: 'check-circle'
        },
        {
          name: '已归档',
          value: 4,
          background: '#78bb61',
          icon: 'minus-circle'
        },
        {
          name: '延期',
          value: 5,
          background: '#78bb61',
          icon: 'exclamation-circle'
        },
        {
          name: '暂停',
          value: 6,
          background: '#78bb61',
          icon: 'pause-circle'
        },
        {
          name: '延误',
          value: 7,
          background: '#78bb61',
          icon: 'exclamation-circle'
        }
      ]
    }
  },
  components: {
    employeeTree
  },
  created() {
    this.loadData(this.projectId)
    // console.log(moment("2019-01-28T10:35:48.791Z").format('YYYY-MM-DD'))
  },
  mounted() {
    document.removeEventListener('click', this.handleDocumentClick)
    document.addEventListener('click', this.handleDocumentClick)
    // document.getElementById('steps').removeEventListener('mousemove',this.removeMove)
    // document.getElementById('steps').addEventListener('mousemove', this.removeMove)

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
                principal
                principalId
                projectNo
                endDate
                startDate
                status
                steps {
                  id
                  name
                  order
                  status
                  planTime
                  finishTime
                }
                plans{
                  id
                  name
                  status
                  remark
                  endDate
                  startDate
                  principal
                  members
                }
                summarys {
                  id
                  content
                  createdBy
                  createdAt
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
      this.principal.name = data[0].principal
      this.steps = data[0].steps
      this.summarys = data[0].summarys
      this.plans = data[0].plans
      this.status = data[0].status
      var statusData = this.statusArr.filter(ele => ele.value == status)
      this.nowStatus = statusData[0].name
      this.nowStatusColor = statusData[0].background

      this.summarysData = {};
      this.summarys.forEach(_sum => {
        var time = moment(_sum.createdAt).format('YYYY-MM-DD')
        var obj = Object.assign({},_sum);
        obj.time = moment(_sum.createdAt).format('HH:mm')
        if(this.summarysData[time]){
          this.summarysData[time].push(obj)
        }else {
          this.summarysData[time] = [obj]
        }
      });
      console.log(this.summarysData)

    },
    //项目状态改变
    handleDocumentClick(e) {
      if (this.$refs.statusDiv) {
        this.statusListShow = false
      }
    },

    //显示项目状态列表
    showStatusList() {
      this.statusListShow = true
    },
    //点击列表并隐藏
    StatusListClick(record) {
      this.nowStatus = record.name
      this.nowStatusColor = record.background
      this.statusListShow = false
    },
    //人员弹框返回结果
    changeEMStatus(obj) {
      this.visibled = obj.visibled
      if (this.type == 1) {
        //修改项目负责人
        this.principal.id = obj.data[0].key
        this.principal.name = obj.data[0].title
        this.updatePrincipal()
      }
    },
    //人员弹框显示
    showEmployee(type) {
      this.visibled = true
      this.type = type
    },
    addPlans() {
      this.data.push({
        name: '',
        status: 0,
        remark: '',
        endDate: '',
        startDate: '',
        principal: '',
        members: ''
      })
    },

    //修改项目负责人
    updatePrincipal() {
      var _this = this
      this.$apollo
        .mutate({
          mutation: gql`
          mutation {
            updateProject(
              data: {
                principal: "${_this.principal.name}"
                summarys: { create: { content: "修改了项目负责人为【${_this.principal.name}】", createdBy: "吴立新" } }
              }
              where: { id: "${_this.projectId}" }
            ) {
              id
            }
          }
        `
        })
        .then(res => {
          _this.loadData(_this.projectId)
        })
        .catch(err => {
          console.log(err)
        })
    },
    //进展节点移入事件
    flowMove(i) {
      this.move = i
    },
    //进展节点移出事件
    removeMove(){
      // console.log('mousemove')
      this.move = -1
    }
  }
}
</script>

<style>
.project-leader {
  width: 28px;
  height: 28px;
  border-radius: 100%;
  margin-right: 5px;
}
div.project-complete {
  background-color: #5d9cec;
}
.select-box {
  padding: 5px;
  border-radius: 2px;
}
div.project-complete .select-content {
  border-color: #8bb8f1;
}
.select-content {
  height: 43px;
  border: 1px solid;
  color: #fff;
  border-radius: 2px;
  position: relative;
  cursor: pointer;
}
.select-content span.select-contentL {
  line-height: 43px;
  font-size: 18px;
  margin-left: 10px;
}
.select-content .anticon-down {
  color: #fff;
  position: absolute;
  top: 12px;
  right: 10px;
  width: 20px;
  height: 20px;
  text-align: center;
  line-height: 20px;
}
.select-content span i {
  font-size: 20px;
  margin-right: 10px;
}

.select-list {
  position: absolute;
  width: 237px;
  background: inherit;
  z-index: 2;
}
.font-blue {
  color: #5873c9 !important;
}
.font-red {
  color: #fa8564 !important;
}
.font-green {
  color: #7fb144 !important;
}
.font-center {
  text-align: center !important;
}

.project-date b {
  font-size: 30px;
}
.project-date span {
  font-size: 36px;
  margin: 0 5px;
}
.progress .ant-timeline-item {
  padding-bottom: 0px;
}
.progress .ant-timeline-item-tail {
  /* position: absolute;
    left: 4px;
    top: 0.75em;
    height: 100%; */
  border-left: 2px solid #7abb65;
}
.progress .ant-timeline.ant-timeline-pending .ant-timeline-item-last .ant-timeline-item-tail {
  border-left: 2px solid #7abb65;
  display: block;
}
.nofocus:focus {
  outline: none;
  box-shadow: none;
}

.project-flow .project-flow_l {
  *zoom: 1;
  padding: 0 0 40px 25px;
}
.project-flow .project-flow_l .flow-item {
  padding-right: 30px;
  float: left;
  display: inline;
}
.project-flow .flow-item {
  position: relative;
  /* width: 190px; */
}
.project-flow .flow-status {
  visibility: hidden;
  margin-bottom: 5px;
}
.project-flow .flow-box dt {
  display: inline-block;
  color: #555;
  width: 150px;
  font-size: 16px;
  padding: 10px;
  border: 2px solid #aaa;
  border-radius: 50px;
  margin-bottom: 5px;
  cursor: pointer;
}
.project-flow .flow-box dt i {
  margin-right: 10px;
  font-style: italic;
}
.project-flow .flow-box dd {
  font-size: 9px;
}
.project-flow .flow-box dd i {
  font-style: normal;
  padding: 0 3px;
  color: #fff;
  border-radius: 2px;
  margin-right: 5px;
  background-color: #7fb144;
}
.project-flow .flow-time {
  vertical-align: top;
  border: 0;
}
.project-flow .project-flow_l .flow-arrow {
  position: absolute;
  right: 5px;
  top: 37px;
}
.project-flow .flow-arrow {
  font-size: 16px;
  color: #aaa;
}
.project-flow .project-flow_l .flow-item:nth-child(6) .flow-arrow {
  right: 100px;
  top: 105px;
}

.project-flow .project-flow_r {
    *zoom: 1;
    padding: 0 28px 40px 0;
}

.project-flow .project-flow_r .flow-item {
    padding-left: 30px;
    float: right;
    display: inline;
}
.project-flow .project-flow_r .flow-arrow {
    position: absolute;
    left: 4px;
    top: 37px;
}
.project-flow .project-flow_r .flow-item {
    padding-left: 30px;
    float: right;
    display: inline;
}
</style>

