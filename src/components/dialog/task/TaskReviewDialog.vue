<template>
  <div>
    <!-- <a-button type="primary" @click="showModal">Open Modal</a-button> -->
    <a-modal
      :title="inititle"
      v-model="visibled"
      @ok="handleOk"
      @cancel="handleCancel"
      :maskClosable="maskClosable"
      :width="600"
    >
      <a-row class="task-review-dialog">
        <a-col :span="24">
          <table class="form-table" width="100%" cellspacing="0" cellpadding="0">
            <tbody>
              <tr>
                <th width="80">审核主题：</th>
                <td colspan="3">
                  <a-select
                    showSearch
                    placeholder="审核主题"
                    optionFilterProp="children"
                    v-model="name"
                    style="width: 470px"
                    disabled
                  >
                    <a-select-option value="初稿">初稿</a-select-option>
                    <a-select-option value="二稿">二稿</a-select-option>
                    <a-select-option value="成果审核">成果审核</a-select-option>
                  </a-select>
                </td>
              </tr>
              <tr>
                <th width="80"></th>
                <td class="ui-radio_group" style="width: 160px">
                  <a-radio-group name="radioGroup" :defaultValue="1">
                    <a-radio :value="1">同意</a-radio>
                    <a-radio :value="0">不同意</a-radio>
                  </a-radio-group>
                </td>
                <th width="80">审核阶段：</th>
                <td>
                  <a-select v-model="step" style="width: 120px" disabled>
                    <a-select-option value="0">一级审核</a-select-option>
                    <a-select-option value="1">二级审核</a-select-option>
                    <a-select-option value="2">三级审核</a-select-option>
                    <a-select-option value="3">一般审核</a-select-option>
                  </a-select>
                </td>
              </tr>
              <tr>
                <th class="th-top">审核意见：</th>
                <td colspan="3">
                  <a-textarea v-model="reviewComment" style="width:470px" rows="3" placeholder="审核意见"></a-textarea>
                </td>
              </tr>
              <tr>
                <th>关键指标：</th>
                <td colspan="3">
                  <table width="100%" cellspacing="0" cellpadding="0">
                    <tbody>
                      <tr>
                        <th width="90">送审金额：</th>
                        <td>
                          <a-input-number v-model="submitAmount" :min="0" :step="0.1" style="width: 110px"/>&nbsp;元
                        </td>
                        <th width="90">审定金额：</th>
                        <td>
                          <a-input-number v-model="approvedAmount" :min="0" :step="0.1" style="width: 110px"/>&nbsp;元
                        </td>
                      </tr>
                      <tr>
                        <th>审增审减额：</th>
                        <td>
                          <a-input-number v-model="differenceValue" :min="0" :step="0.1" style="width: 110px" readonly/>&nbsp;元
                        </td>
                        <th>单方造价：</th>
                        <td>
                          <a-input-number v-model="unitPrice" :min="0" :step="0.1" style="width: 110px"/>&nbsp;元
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </td>
              </tr>
              <tr>
                <th class="th-top">附件：</th>
                <td colspan="3">
                  <a-upload name="file" :multiple="true" action="//jsonplaceholder.typicode.com/posts/"
                            @change="uploadChange">
                    <a-button>
                      <a-icon type="upload"/>
                      添加文件
                    </a-button>
                  </a-upload>
                </td>
              </tr>
              <tr class="nextTrClass">
                <th width="80"></th>
                <td><a-checkbox @change="toNextReviewer" v-model="isShowNextReview">是否给下一审核人</a-checkbox></td>
              </tr>
              <tr id="nextTr" :class="isShowNextReview ? showNextReview : hideNextReview">
                <th width="80">下一阶段：</th>
                <td>
                  <a-select v-model="NextStep" style="width: 120px" disabled>
                    <a-select-option value="0">一级审核</a-select-option>
                    <a-select-option value="1">二级审核</a-select-option>
                    <a-select-option value="2">三级审核</a-select-option>
                  </a-select>
                </td>
                <th width="80">
                  <label class="ant-form-item-required">审核人：</label>
                </th>
                <td width="230">
                  <a-input
                    type="text"
                    v-model="NextHandler.name"
                    placeholder
                    readonly
                    maxlength="200"
                    @click="showEModal(1, 1, true)"
                  />
                  <!--ms-if-->
                </td>
              </tr>
            </tbody>
          </table>
        </a-col>
      </a-row>
    </a-modal>

    <!-- 人员弹框 -->
    <employee-tree
      :inititle="emTitle"
      :inivisible="visible"
      :type="employeeType"
      :isOne="isOne"
      @changeStatus="changeEMStatus"
    />
  </div>
</template>

<script>

  import employeeTree from '@/components/same/employeeTree'

  export default {
    props: {
      inititle: {
        type: String,
        default: ''
      },
      initvisibled: {
        type: Boolean,
        default: false
      },
      maskClosable: {
        type: Boolean,
        default: false
      },
      currentTask: {
        type: Object,
        default: null
      }
    },
    components: {
      employeeTree
    },
    data() {
      return {
        showNextReview: 'showNextReview',
        hideNextReview: 'hideNextReview',
        visible: false,
        visibled: this.initvisibled,
        isShowNextReview: false,
        employeeType: 1, //1显示员工 2只显示部门
        subType: 1,     //1：负责人 2：通知到
        isOne: true,     //true: 只选一个 false: 可以多选
        emTitle: '添加人员',
        name: '成果审核', //审核主题
        step: '0',  //审核阶段
        type: 1, //任务审核任务
        sendTo: [], //通知到的人
        sendDesc: '',
        handler: {}, //审核人id
        submitAmount: 0, //送审金额
        approvedAmount: 0, //审定金额
        unitPrice: 0, //单方造价
        remark: 0, //审核纪要
        reviewResult: 0, //审核结果 -1：未审核 0:不同意 1:同意
        reviewComment: '',                    //审核意见
        reviewDate: new Date(),               //审核日期
        NextStep: '1',                        //下级审核阶段
        NextHandler: {},                      //下级审核人
        files: [] //上传文件
      }
    },
    computed: {
      differenceValue: function() {
        return this.approvedAmount - this.submitAmount
      }
    },
    methods: {
      handleOk() {
        this.$emit('reviewSave', this)
      },
      handleCancel() {
        this.$emit('reviewCancel')
      },
      toNextReviewer (e) {
        console.log(`checked = ${e.target.checked}`)
      },
      //显示人员或部门弹框
      showEModal(type, subType, isOne) {
        this.employeeType = type
        this.subType = subType
        this.isOne = isOne
        this.visible = true
        if (subType === 1) {
          this.emTitle = '选择审核人'
        } else {
          this.emTitle = '通知到'
        }
      },
      //人员/部门弹框状态改变
      changeEMStatus(obj) {
        console.log(obj)
        this.visible = obj.visibled
        if (!obj.data.length) {
          return
        }
        if (this.subType === 1) {
          this.NextHandler.id = obj.data[0].key
          this.NextHandler.name = obj.data[0].title
          //员工
        } else {
          this.sendTo = obj.data
          this.sendDesc = ''
          const _this = this
          obj.data.forEach(function(item){
            _this.sendDesc = _this.sendDesc + item.title + ', '
          })
        }
      },
      uploadChange(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          this.$message.success(`${info.file.name} file uploaded successfully`)
        } else if (info.file.status === 'error') {
          this.$message.error(`${info.file.name} file upload failed.`)
        }
      }
    },
    watch: {
      initvisibled(newT, oldT) {
        console.log(newT, oldT, '-------')
        this.visibled = newT
      },
      currentTask: function(task) {
        // alert('currentTask change')
        // 设置审核任务的信息
        if(this.currentTask){
          this.name = task.name
          this.step = task.step.toString()
          this.type = task.type
          this.submitAmount = task.submitAmount
          this.approvedAmount = task.approvedAmount
          this.unitPrice = task.unitPrice
          this.nextStep = (task.step + 1).toString()
          this.reviewResult = task.reviewResult
        }
      }
    }
  }
</script>
<style>
  .ui-w115 {
    width: 115px !important;
  }

  .form-table th,
  .form-table td {
    padding: 5px 0;
    font-size: 14px;
    position: relative;
  }

  .th-right {
    text-align: right;
  }

  .th-top {
    padding-top: 9px !important;
    vertical-align: top;
  }

  .add-task_dialog th {
    text-align: right;
  }

  .showNextReview {
    display: table-row;
  }

  .hideNextReview {
    display: none;
  }
</style>

