<template>
  <div>
    <a-row>
      <a-col :span="24">
        <a-button type="primary" icon="file-done" style="margin-right:15px" @click="showReview">提交审核</a-button>
        <a-button type="default" icon="file-done">整理成果文件</a-button>
      </a-col>
    </a-row>
    <a-row class="ProjectReview">
      <a-col :span="24" v-for="(item,i) in data" :key="i" style="margin-top:10px">
        <a-card
          :title="item.name"
          :headStyle="{height:'48px',lineHeight:'48px',background:'#eee',padding:'0 10px'}"
        >
          <a-row>
            <a-col :span="16">
              <a-steps>
                <a-step status="finish">
                  <span slot="title" style="font-size:14px">发起审核</span>
                  <!-- <a-icon type="user" slot="icon"/> -->
                  <a-icon type="eye" slot="icon"/>
                  <template slot="description">
                    <div style="margin-top: 10px;font-size:13px;color: #555;">
                      <!-- <p>
                        <span>任务名称：</span>{{item.name}}
                      </p> -->
                      <p>
                        <span>发起人：</span>{{item.parentNode?item.parentNode.createdBy?item.parentNode.createdBy.name:'':''}}
                      </p>
                      <p>
                        <span>发起日期：</span>2019-01-31
                      </p>
                      <p>
                        <span>总造价：</span>0
                      </p>
                      <p>
                        <span>附件：</span>
                      </p>
                      <p>
                        <span>说明：</span>
                      </p>
                      <p>
                        <a>
                          <a-icon type="audit"/>审核纪要
                        </a>
                      </p>
                    </div>
                  </template>
                </a-step>
                <a-step  v-for="num in 3" :key="num" :status="item.parentNode.childTasks[num-1] ?'finish':'wait'">
                  <span slot="title" style="font-size:14px" v-if="num == 1">一级审核</span>
                  <span slot="title" style="font-size:14px" v-else-if="num == 2">二级审核</span>
                  <span slot="title" style="font-size:14px" v-else-if="num == 3">三级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                  <template slot="description" v-if="item.parentNode.childTasks[num-1]">
                    <div style="margin-top: 10px;font-size:13px;color: #555;">
                      <!-- <p>
                        <span>任务名称：</span>装修工程
                      </p> -->
                      <p>
                        <span>审核人：</span>{{ item.parentNode.childTasks[num-1].handler?item.parentNode.childTasks[num-1].handler.name:'' }}
                      </p>
                      <p>
                        <span>审核日期：</span>2019-01-31
                      </p>
                      <p>
                        <span>总造价：</span>{{ item.parentNode.childTasks[num-1].approvedAmount }}
                      </p>
                      <p>
                        <span>附件：</span>
                      </p>
                      <p>
                        <span>说明：</span>
                      </p>
                      <!-- <p>
                        <a>
                          <a-icon type="audit"/>审核纪要
                        </a>
                      </p> -->
                    </div>
                  </template>
                </a-step>
                <!-- <a-step status="process">
                  <span slot="title" style="font-size:14px">二级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                </a-step>
                <a-step status="wait">
                  <span slot="title" style="font-size:14px">三级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                </a-step> -->
              </a-steps>
            </a-col>
            <a-col :span="8" style="height: 248px;line-height: 248px;position:relative">
              <div style="display:flex;align-items:center;justify-content: center;width:100%;height:100%;">
                <MyIcon type="icon-shenpitongyi" style="font-size:100px"/>
              </div>
              
            </a-col>
          </a-row>
        </a-card>
      </a-col>
      <a-col :span="24" v-for="(value, key, d) in reviewData" :key="key" style="margin-top:10px">
        <a-card
          :title="key"
          :headStyle="{height:'48px',lineHeight:'48px',background:'#eee',padding:'0 10px'}"
        >
          <a-row>
            <a-col :span="16">
              <a-steps>
                <a-step status="finish">
                  <span slot="title" style="font-size:14px">发起审核</span>
                  <!-- <a-icon type="user" slot="icon"/> -->
                  <a-icon type="eye" slot="icon"/>
                  <template slot="description">
                    <div style="margin-top: 10px;font-size:13px;color: #555;">
                      <!-- <p>
                        <span>任务名称：</span>装修工程
                      </p> -->
                      <p>
                        <span>发起人：</span>{{value.createdBy?value.createdBy.name:''}}
                      </p>
                      <p>
                        <span>发起日期：</span>2019-01-31
                      </p>
                      <p>
                        <span>总造价：</span>0
                      </p>
                      <p>
                        <span>附件：</span>
                      </p>
                      <p>
                        <span>说明：</span>
                      </p>
                      <p>
                        <a>
                          <a-icon type="audit"/>审核纪要
                        </a>
                      </p>
                    </div>
                  </template>
                </a-step>
                <a-step  v-for="num in 3" :key="num" :status="value[num-1] ?'finish':'wait'">
                  <span slot="title" style="font-size:14px" v-if="num == 1">一级审核</span>
                  <span slot="title" style="font-size:14px" v-else-if="num == 2">二级审核</span>
                  <span slot="title" style="font-size:14px" v-else-if="num == 3">三级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                  <template slot="description" v-if="value[num-1]">
                    <div style="margin-top: 10px;font-size:13px;color: #555;">
                      <!-- <p>
                        <span>任务名称：</span>装修工程
                      </p> -->
                      <p>
                        <span>审核人：</span>{{value[num-1].handler?value[num-1].handler.name:''}}
                      </p>
                      <p>
                        <span>审核日期：</span>2019-01-31
                      </p>
                      <p>
                        <span>总造价：</span>{{value[num-1].approvedAmount}}
                      </p>
                      <p>
                        <span>附件：</span>
                      </p>
                      <p>
                        <span>说明：</span>{{value[num-1].reviewComment}}
                      </p>
                      <!-- <p>
                        <a>
                          <a-icon type="audit"/>审核纪要
                        </a>
                      </p> -->
                    </div>
                  </template>
                </a-step>
                <!-- <a-step status="process">
                  <span slot="title" style="font-size:14px">二级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                </a-step>
                <a-step status="wait">
                  <span slot="title" style="font-size:14px">三级审核</span>
                  <a-icon type="check-circle" slot="icon"/>
                </a-step> -->
              </a-steps>
            </a-col>
            <a-col :span="8" style="height: 248px;line-height: 248px;position:relative">
              <div style="display:flex;align-items:center;justify-content: center;width:100%;height:100%;">
                <MyIcon type="icon-shenpitongyi" style="font-size:100px"/>
              </div>
              
            </a-col>
          </a-row>
        </a-card>
      </a-col>
    </a-row>

    <!-- <ProjectReviewDialog 
    
    />-->
    <ProjectReviewDialog
      inititle="提交审核"
      :initvisibled="openReview"
      @reviewSave="reviewSave"
      @reviewCancel="reviewCancel"
    />
  </div>
</template>

<script>
import { Icon } from 'ant-design-vue'
const MyIcon = Icon.createFromIconfontCN({
  scriptUrl: '//at.alicdn.com/t/font_1018467_zbasjalz32.js' // 在 iconfont.cn 上生成
})

import ProjectReviewDialog from '@/components/dialog/project/ProjectReviewDialog'
import gql from 'graphql-tag'

import {
		mapState,
		mapMutations
	} from 'vuex'

export default {
  name: 'ProjectReview',
  props: {
    projectId: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      openReview: false,
      data: [],
      reviewData:{},
      stepArr:['一级审核','二级审核','三级审核']
    }
  },
  components: {
    MyIcon,
    ProjectReviewDialog
  },
  created() {
    this.loadData(this.projectId)
  },
  computed: {
    ...mapState({
      userInfo: state => state.user.info
    })
  },
  methods: {
    loadData(id) {
      var _this = this
      _this.$apollo
        .query({
          query: gql`
            query {
             
                tasks(where: {}) {
                  id
                  name
                  type
                  step
                  reviewComment
                  createdBy {
                    id
                    name
                  }
                  handler{
                    id
                    name
                  }
                  project{
                    id
                    name
                  }
                  parentNode {
                    id
                    name
                    status
                    createdBy {
                      id
                      name
                    }
                    childTasks {
                      id
                      name
                      approvedAmount
                      reviewComment
                      handler{
                        id
                        name
                      }
                      createdBy {
                        id
                        name
                      }
                      updatedAt
                    }
                  }
                }
              }
            
          `,
          fetchPolicy:'network-only'
        })
        .then(res => {
          console.log(res.data.tasks, 'duojifuhe')
          _this.data = res.data.tasks.filter(ele => ele.type == 2 && ele.project.id == id && ele.step == 0 )
          var reviewData = res.data.tasks.filter(ele => ele.type == 3)
          reviewData.forEach(ele => {
            if(_this.reviewData[ele.name]){
              _this.reviewData[ele.name].push(ele)
            }else {
              _this.reviewData[ele.name] = [ele]
            }
          })
          // if(res.elsedata.projects.length){
          //   _this.data = res.data.projects[0].tasks.filter(ele => ele.type == 2 || ele.type == 3)
            console.log(_this.data ,_this.reviewData,'dfdfdfdfd')
          // }else {
          //   _this.data = []
          // }
          // _this.data = res.data.projects.tasks.filter(ele => type == 2 || type == 3)
        })
        .catch(err => {
          console.log(err)
        })
    },
    showReview() {
      this.openReview = true
    },
    reviewSave(data) {
      console.log(data, '___________')
      var _this = this
      this.$apollo.mutate({
        mutation:gql`
        mutation {
          createTask(
            data: {
              project: { connect: { id: "${_this.projectId}" } }
              type: 3
              handler: { connect: { id: "${data.NextHandler.id}" } }
              name: "${data.name}"
              step: ${data.step}
              submitAmount: ${data.submitAmount}
              approvedAmount: ${data.approvedAmount}
              unitPrice: ${data.unitPrice || 0}
              remark: "${data.remark}"
              status: 4
              reviewResult: -1
              createdBy:{connect:{id:"${_this.userInfo.id}"}}
            }
          ) {
            id
          }
        }`
      }).then(res => {

      }).catch(err => {
        console.log(err)
      })
      this.openReview = false
    },
    reviewCancel() {
      this.openReview = false
    }
  }
}
</script>

<style>
.ProjectReview {
  /* margin-top: 10px; */
}
.ProjectReview .ant-card-head-title {
  padding: 0;
}
.ProjectReview .ant-card-wider-padding .ant-card-body {
  padding: 15px;
}
.ProjectReview p {
  margin-bottom: 10px;
}
</style>

