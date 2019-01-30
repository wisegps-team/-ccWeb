<template>
  <div class="projectInfo">
    <a-row style="min-width: 1190px;">
      <!-- 左边 -->
      <a-col :span="24">
        <!-- 第一段 -->
        <div class="divItem borderBottom">
          <table class="form-table table-layout" width="870" cellspacing="0" border="0">
            <tbody>
              <tr>
                <th width="120" class="th-right">
                  <label title="业务类型" class="ant-form-item-required">业务类型：</label>
                </th>
                <td width="750" colspan="3" style="position:relative">
                  <a-input
                    type="text"
                    placeholder
                    readonly
                    maxlength="200"
                    @click="showBusItem"
                    :value="businessType.join(',')"
                  />
                  <a-row class="bussT" ref="busDiv" v-if="isShow">
                    <a-col :span="24">
                      <a-checkbox-group
                        @change="busItemChange"
                        style="width:100%"
                        :value="businessType"
                      >
                        <a-row>
                          <a-col :span="8" v-for="(Bitem,index) in BusinessType" :key="index">
                            <div v-for="(sBitem,sindex) in Bitem" :key="sindex">
                              <a-checkbox :value="sBitem">{{ sBitem }}</a-checkbox>
                            </div>
                          </a-col>
                        </a-row>
                      </a-checkbox-group>
                    </a-col>
                    <!-- <a-col :span="8">2</a-col>
                    <a-col :span="8">3</a-col>-->
                  </a-row>
                </td>
              </tr>
              <tr>
                <th width="100" class="th-right">
                  <label title="项目编号" class="ant-form-item-required">项目编号：</label>
                </th>
                <td width="260">
                  <a-input
                    type="text"
                    class="sameWidth"
                    v-model="projectNo"
                    placeholder="请选择关联合同或业务类型，编号自动生成"
                  />
                </td>
                <th width="130" class="th-right">关联合同：</th>
                <td width="360">
                  <a-input
                    type="text"
                    class="sameWidth"
                    style="width: 360px;"
                    placeholder
                    @click="showContractDialog"
                    readonly="readonly"
                  />
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">
                  <label title="项目名称" class="ant-form-item-required">项目名称：</label>
                </th>
                <td width="750" colspan="3">
                  <a-input type="text" v-model="name" placeholder="项目名称为必填项目" maxlength="200"/>
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">
                  <label title="项目地址">项目地址：</label>
                </th>
                <td width="750" colspan="3">
                  <a-select
                    v-model="selectProvice"
                    style="width: 120px;margin-right:5px"
                    @change="changeAreaData(1)"
                  >
                    <a-select-option
                      v-for="province in provices"
                      :key="province"
                      :value="province"
                    >{{ province }}</a-select-option>
                  </a-select>
                  <a-select
                    v-model="selectCity"
                    style="width: 120px;margin-right:5px"
                    @change="changeAreaData(2)"
                  >
                    <a-select-option v-for="city in cities" :key="city" :value="city">{{ city }}</a-select-option>
                  </a-select>
                  <a-select v-model="selectArea" style="width: 120px;margin-right:5px">
                    <a-select-option v-for="area in areas" :key="area" :value="area">{{ area }}</a-select-option>
                  </a-select>
                  <a-input type="text" placeholder="输入详细地址" v-model="address" style="width: 375px"/>
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">工程类型：</th>
                <td width="260">
                  <!-- <a-date-picker class="sameWidth"/> -->
                  <a-select class="sameWidth" v-model="engineeringType">
                    <a-select-option
                      v-for="(item,index) in engineeringTypeArr"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>
                </td>
                <th width="100" class="th-right">结构类型：</th>
                <td width="360">
                  <a-select class="sameWidth" v-model="structureType" style="width: 360px;">
                    <a-select-option
                      v-for="(item,index) in struType"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>
                  <!-- <a-date-picker class="sameWidth"/> -->
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">项目类型：</th>
                <td width="260">
                  <a-input type="text" v-model="projectType"/>
                  <!-- <a-date-picker class="sameWidth"/> -->
                  <!-- <a-select class="sameWidth" :value="projectType[0]">
                    <a-select-option
                      v-for="(item,index) in projectType"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>-->
                </td>
                <th width="120" class="th-right">工程规模：</th>
                <td width="260">
                  <a-input
                    type="text"
                    placeholder
                    v-model="projectScale"
                    maxlength="200"
                    style="width: 255px;margin-right:5px"
                  />
                  <a-select style="width: 100px;" v-model="projectScaleUnit">
                    <a-select-option
                      v-for="(item,index) in measurArr"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">项目经理：</th>
                <td width="260">
                  <!-- <a-date-picker class="sameWidth"/> -->
                  <a-input
                    type="text"
                    v-model="principal.name"
                    placeholder
                    readonly
                    maxlength="200"
                    @click="showEModal(1)"
                  />
                </td>
                <th width="120" class="th-right">项目归属部门：</th>
                <td width="260">
                  <a-input
                    type="text"
                    v-model="department.name"
                    placeholder
                    readonly
                    maxlength="200"
                    @click="showEModal(2)"
                  />
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right">项目资金来源：</th>
                <td width="260">
                  <!-- <a-date-picker class="sameWidth"/> -->
                  <a-select style="width:130px;margin-right: 10px;" v-model="moneySource">
                    <a-select-option
                      v-for="(item,index) in moneySourceArr"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>
                </td>
                <th width="120" class="th-right">预估咨询费：</th>
                <td width="260">
                  <a-input
                    type="text"
                    placeholder
                    maxlength="200"
                    style="width: 255px;margin-right:5px"
                  />元
                </td>
              </tr>
              <tr>
                <th width="120" class="th-top">项目概况：</th>
                <td width="750" colspan="3">
                  <a-textarea
                    rows="54"
                    placeholder="最多1000字"
                    maxlength="1000"
                    style="height: 100px;"
                  ></a-textarea>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <!-- 第二段 -->
        <div class="divItem">
          <table class="form-table table-layout" width="870" cellspacing="0" border="0">
            <tbody>
              <tr>
                <th width="120" class="th-right">
                  <label title="委托人" class="ant-form-item-required">委托人：</label>
                </th>
                <td width="750" colspan="3">
                  <a-input
                    type="text"
                    placeholder="委托人为必填项目"
                    maxlength="200"
                    v-model="client.name"
                    style="width: 680px;margin-right: 5px;"
                  />
                  <a-button type="primary" @click="showClientDialog">选择</a-button>
                </td>
              </tr>

              <tr>
                <th width="120" class="th-right">
                  <label title="项目地址">委托方地址：</label>
                </th>
                <td width="750" colspan="3">
                  <a-select
                    v-model="selectProvice2"
                    style="width: 120px;margin-right:5px"
                    @change="changeAreaData2(1)"
                  >
                    <a-select-option
                      v-for="province in provices2"
                      :key="province"
                      :value="province"
                    >{{ province }}</a-select-option>
                  </a-select>
                  <a-select
                    v-model="selectCity2"
                    style="width: 120px;margin-right:5px"
                    @change="changeAreaData2(2)"
                  >
                    <a-select-option v-for="city in cities2" :key="city" :value="city">{{ city }}</a-select-option>
                  </a-select>
                  <a-select v-model="selectArea2" style="width: 120px;margin-right:5px">
                    <a-select-option v-for="area in areas2" :key="area" :value="area">{{ area }}</a-select-option>
                  </a-select>
                  <a-input
                    type="text"
                    v-model="address2"
                    placeholder="输入详细地址"
                    style="width: 375px"
                  />
                </td>
              </tr>
              <tr v-for="(it,i) in organizations" :key="i">
                <th width="120" class="th-right">
                  <label title="项目地址">参与单位{{ i+1 }}：</label>
                </th>
                <td>
                  <a-select style="width:130px;margin-right: 10px;" v-model="it.type">
                    <a-select-option
                      v-for="(item,index) in deparArr"
                      :key="index"
                      :value="item"
                    >{{ item }}</a-select-option>
                  </a-select>
                  <a-input
                    type="text"
                    style="width: 580px;margin-right: 10px;"
                    placeholder
                    class="sameWidth"
                    v-model="it.name"
                  />
                  <a-icon type="delete" style="cursor:pointer" @click="DelOrganizations(i)"></a-icon>
                </td>
              </tr>
              <tr>
                <th width="120" class="th-right"></th>
                <td>
                  <a @click="addOrganizations">
                    <a-icon type="plus"></a-icon>添加参与单位
                  </a>
                </td>
              </tr>
            </tbody>
          </table>

          <div style="text-align:center">
            <a-button type="primary" style="margin-right:10px" @click="submit">保存</a-button>
            <a-button type="default" @click="CreateCancal">取消</a-button>
          </div>
        </div>
        <!-- 第三段 -->
        <!-- 第四段 -->
      </a-col>
      <!-- 右边 -->
    </a-row>

    <!-- 人员弹框 -->
    <employee-tree
      :inititle="emTitle"
      :inivisible="visible"
      :type="employeeType"
      @changeStatus="changeEMStatus"
    />
    <!-- 委托人 -->
    <cilentDialog
      :inititle="clientTitle"
      :inivisible="clientVisible"
      @changeStatus="changeClStatus"
    />
    <!-- 关联合同 -->
    <contractDialog
      :inititle="contractTitle"
      :inivisible="contractVisible"
      @changeStatus="changeConStatus"
    />
  </div>
</template>

<script>
// import { getServiceList } from '@/api/manage'

import employeeTree from '@/components/same/employeeTree'
import cilentDialog from '@/components/dialog/project/clientDialog'
import contractDialog from '@/components/dialog/project/contractDialog'
import { AreaData } from '@/components/same/area'

import gql from 'graphql-tag'

var deparArr = ['施工单位', '监理单位', '编制单位', '项目业主', '设计单位', '咨询单位', '代建单位', '其他']
var moneySourceArr = ['内资项目', '外资项目', '港资项目', '其它']
var measurArr = ['平方米', '立方米', '公里', '米', '公斤', '吨', '工日', '其他']
var engineeringTypeArr = [
  '房屋建筑工程',
  '市政公用工程',
  '公路工程',
  '铁路工程',
  '水利水电工程',
  '通信工程',
  '化工石油工程',
  '电力工程',
  '农林工程',
  '冶炼工程',
  '机电安装工程',
  '港口与航道工程',
  '航天航空工程',
  '其他'
]
var struType = [
  '网架结构',
  '拱式结构',
  '膜结构',
  '砖混结构',
  '框架结构',
  '剪力墙结构',
  '框架-剪力墙结构',
  '筒体结构',
  '钢结构',
  '排架结构',
  '砖木结构',
  '轻钢结构',
  '重钢结构',
  '全现浇结构'
]

var BusinessType = [
  [
    '编制投资估算',
    '编制设计概算',
    '编制施工图预算',
    '编制工程量清单',
    '编制工程量清单控制价',
    '编制投标报价',
    '编制竣工结算',
    '编制竣工决算'
  ],
  ['审核投资估算', '审核设计概算', '审核施工图预算', '审核清单控制价', '审核工程变更', '审核竣工结算'],
  ['可行性经济评价', '全过程工程造价管理', '工程造价鉴定', '工程造价信息咨询服务', '其他工程造价咨询工作']
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
  name: 'ProjectInfo',
  inject: ['reload'],
  props: {
    type: {
      type: String,
      default: '1'
    },
    projectId: {
      type: String,
      defalut: ''
    }
    // dataSource: {
    //   type: Object,
    //   default: function(value) {
    //     return { value }
    //   }
    // }
  },
  components: {
    employeeTree,
    cilentDialog,
    contractDialog
  },
  data() {
    return {
      loading: false,
      AreaData,

      provices: [],
      cities: [],
      areas: [],
      selectProvice: '广东省', //项目地址 省
      selectCity: '汕头市', //项目地址 市
      selectArea: '龙湖区', //项目地址 区
      address: '', //项目地址 详细地址

      provices2: [],
      cities2: [],
      areas2: [],
      selectProvice2: '广东省', //委托人地址 省
      selectCity2: '汕头市', //委托人地址 市
      selectArea2: '龙湖区', //委托人地址 区
      address2: '', //委托人地址 详细地址
      // provinceData,
      engineeringTypeArr,
      struType,
      deparArr,
      moneySourceArr,
      measurArr,
      BusinessType,

      businessType: [], //业务类型
      projectNo: '2019估编010', //项目编号
      name: '', //项目名称
      shortName: '', //项目简称
      engineeringType: '房屋建筑工程', //工程类型
      structureType: '', //结构类型
      projectType: '', //项目类型
      projectScale: '', //工程规模
      projectScaleUnit: '平方米', //工程规模计量单位
      principal: {}, //项目经理
      department: {}, //项目归属部门
      client: {}, //委托人
      contract: {}, //关联合同
      moneySource: '其它', //项目资金来源
      status: 0, //项目状态
      commission: 0, //项目提成比例
      organizations: [{ type: '施工单位', name: '' }], //参与单位
      // organizationsType: '施工单位',
      // organizationsName: '',
      remark: '', //备注

      isShow: false,
      booleanFalse: true,
      visible: false,
      clientVisible: false,
      clientTitle: '选择委托方',
      contractTitle: '关联合同',
      contractVisible: false,

      employeeType: 1, //1显示员工2只显示部门
      emTitle: '添加人员',
      steps: [
        {
          name: '编制类',
          order: 1,
          status: 0
        },
        {
          name: '资料收集',
          order: 2,
          status: 0
        },
        {
          name: '项目启动',
          order: 3,
          status: 0
        },
        {
          name: '算量组价',
          order: 4,
          status: 0
        },
        {
          name: '初稿结果',
          order: 5,
          status: 0
        },
        {
          name: '初稿复核',
          order: 6,
          status: 0
        },
        {
          name: '送出报告书',
          order: 7,
          status: 0
        },
        {
          name: '开票',
          order: 8,
          status: 0
        },
        {
          name: '收款',
          order: 9,
          status: 0
        },
        {
          name: '项目归档',
          order: 10,
          status: 0
        },
        {
          name: '项目奖金分配',
          order: 11,
          status: 0
        }
      ]
    }
  },
  created() {
    //数据请求参数配置
    console.log(this.type, 'projectInfo')

    if (this.type > 4) {
      //获取项目信息信息
      this.loadData(this.projectId)
    } else {
      // 初始化地址
      this.changeAreaData(0)
      this.changeAreaData2(0)
    }
  },

  mounted() {
    document.removeEventListener('click', this.handleDocumentClick)
    document.addEventListener('click', this.handleDocumentClick)
  },
  methods: {
    //接口获取数据
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
                businessType
                engineeringType
                projectType
                structureType
                submitAmount
                moneySource
                approvedAmount
                projectScale
                projectScaleUnit
                status
                desc
                projectAddress {
                  province
                  city
                  district
                  address
                }
                contract {id,name}
                department {id, name}
                organizations {type,name}
                client {
                  id,name,address {
                    province,city,district,address
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
      var _data = data[0]
      console.log(_data)
      this.selectProvice = _data.projectAddress.province || '广东省' //项目地址 省
      this.selectCity = _data.projectAddress.city || '汕头市' //项目地址 市
      this.selectArea = _data.projectAddress.district || '龙湖区' //项目地址 区
      this.address = _data.projectAddress.address || '' //项目地址 详细地址

      this.selectProvice2 = _data.client.address.province || '广东省' //委托人地址 省
      this.selectCity2 = _data.client.address.city || '汕头市' //委托人地址 市
      this.selectArea2 = _data.client.address.district || '龙湖区' //委托人地址 区
      this.address2 = _data.client.address.address || '' //委托人地址 详细地址

      this.businessType = _data.businessType.split(',') //业务类型
      this.projectNo = _data.projectNo //项目编号
      // this.name = _data.name //项目名称
      this.shortName = _data.shortName //项目简称
      this.engineeringType = _data.engineeringType //工程类型
      this.structureType = _data.structureType //结构类型
      this.projectType = _data.projectType //项目类型
      this.projectScale = _data.projectScale //工程规模
      this.projectScaleUnit = _data.projectScaleUnit || '平方米' //工程规模计量单位
      this.principal = { name: _data.principal, id: _data.principalId } //项目经理
      this.department = _data.department || {} //项目归属部门
      this.client = _data.client || {} //委托人
      this.contract = _data.contract || {} //关联合同
      this.moneySource = _data.moneySource //项目资金来源
      this.status = _data.status //项目状态
      // this.commission= 0 //项目提成比例
      this.organizations = _data.organizations //参与单位
    },

    //显示人员或部门弹框
    showEModal(type) {
      this.employeeType = type
      this.visible = true
      if (type == 1) {
        this.emTitle = '添加人员'
      } else {
        this.emTitle = '选择项目归属部门'
      }
    },
    //人员/部门弹框状态改变
    changeEMStatus(obj) {
      console.log(obj)
      this.visible = obj.visibled
      if (!obj.data.length) {
        return
      }
      if (this.employeeType == 1) {
        this.principal.id = obj.data[0].key
        this.principal.name = obj.data[0].title
        //员工
      } else {
        this.department.id = obj.data[0].key
        this.department.name = obj.data[0].title
      }
    },
    //显示关联合同弹框
    showContractDialog() {
      this.contractVisible = true
    },
    //关联合同弹框状态改变
    changeConStatus(obj) {
      console.log(obj, 'dfdfdfdf')
      this.contractVisible = obj.visibled

      if (obj.data) {
        this.contract.id = obj.data.id
        this.contract.name = obj.data.name
        // this.selectProvice2 = obj.data.address.province
        // this.selectCity2 = obj.data.address.city
        // this.selectArea2 = obj.data.address.district
        // this.address2 = obj.data.address.address
      }
    },
    //显示委托人弹框
    showClientDialog() {
      this.clientVisible = true
    },
    //委托人弹框状态改变
    changeClStatus(obj) {
      console.log(obj, 'dfdfdfdf')
      this.clientVisible = obj.visibled

      if (obj.data) {
        this.client.id = obj.data.id
        this.client.name = obj.data.name
        this.selectProvice2 = obj.data.address.province
        this.selectCity2 = obj.data.address.city
        this.selectArea2 = obj.data.address.district
        this.address2 = obj.data.address.address
      }
    },

    //业务类型的显示和隐藏
    handleDocumentClick(e) {
      if (!this.$refs.busDiv) {
        return
      }
      if (!this.$refs.busDiv.$el) {
        return
      }
      if (!this.$refs.busDiv.$el.contains(e.target)) {
        this.isShow = false
      }
    },

    //显示业务类型
    showBusItem() {
      this.isShow = true
    },

    //业务类型多选函数
    busItemChange(checkedValues) {
      this.businessType = checkedValues
    },

    //添加参与单位
    addOrganizations() {
      this.organizations.push({ type: '施工单位', name: '' })
    },
    //删除参与单位
    DelOrganizations(index) {
      this.organizations.splice(index, 1)
    },
    //项目地址
    changeAreaData(type) {
      console.log(type)
      this.provices = []
      this.cities = []
      this.areas = []
      this.AreaData.forEach(ele => {
        this.provices.push(ele.name)
        if (ele.name == this.selectProvice) {
          if (type == 1) {
            this.selectCity = ele.city[0].name
          }
          ele.city.forEach(cele => {
            this.cities.push(cele.name)
            if (cele.name == this.selectCity) {
              if (type == 1 || type == 2) {
                this.selectArea = cele.area[0]
              }
              cele.area.forEach(aele => {
                this.areas.push(aele)
              })
            }
          })
        }
      })
    },
    //委托方地址
    changeAreaData2(type) {
      console.log(type)
      this.provices2 = []
      this.cities2 = []
      this.areas2 = []
      this.AreaData.forEach(ele => {
        this.provices2.push(ele.name)
        if (ele.name == this.selectProvice2) {
          if (type == 1) {
            this.selectCity2 = ele.city[0].name
          }
          ele.city.forEach(cele => {
            this.cities2.push(cele.name)
            if (cele.name == this.selectCity2) {
              if (type == 1 || type == 2) {
                this.selectArea2 = cele.area[0]
              }
              cele.area.forEach(aele => {
                this.areas2.push(aele)
              })
            }
          })
        }
      })
    },

    //取消
    CreateCancal() {
      this.reload()
    },
    //保存
    submit() {
      var _this = this
      // var ql = `

      //     `
      //     console.log(ql)
      this.$apollo
        .mutate({
          mutation: gql`
            mutation {
              createProject(
                data: {
                  businessType: "${_this.businessType.join(',')}"
                  projectNo: "${_this.projectNo}"
                  name: "${_this.name}"
                  shortName: "${_this.name}"
                  contract: { connect: { id: "cjr5wxqyw00h3073507j0kk8k" } }
                  projectAddress: {
                    create: { 
                      province: "${_this.selectProvice}", 
                      city: "${_this.selectCity}", 
                      district: "${_this.selectArea}", 
                      address: "${_this.address}"
                    }
                  }
                  summarys: { create: { content: "创建了【${_this.name}】项目", createdBy: "吴立新" } }
                  engineeringType: "${_this.engineeringType}"
                  structureType: "${_this.structureType}"
                  projectType: "${_this.projectType}"
                  principal: "${_this.principal.name}"
                  moneySource: "${_this.moneySource}"
                  client: { connect: { id: "${_this.client.id}" } }
                  department: {connect: {id: "${_this.department.id}"}}
                  status: 0
                  commission: 0
                  organizations: {
                    create: ${ArrayToString(_this.organizations)}
                  }
                  projectScale: ${parseInt(_this.projectScale || 0)}
                  projectScaleUnit: "${_this.projectScaleUnit}"
                  steps: {
                    create: ${ArrayToString(_this.steps)}
                  }
                }
              ) {
                id
              }
            }
          `
        })
        .then(res => {
          console.log(res, 'res')
          _this.reload()
        })
        .catch(err => {
          console.log(err)
        })
    }
  },
  watch: {}
}
</script>

<style>
.ant-form-item-required:before {
  display: inline-block;
  margin-right: 4px;
  content: '*';
  font-family: SimSun;
  line-height: 1;
  font-size: 14px;
  color: #f5222d;
}
.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.ant-tabs-extra-content {
  line-height: 55px;
}

.divItem {
  padding: 10px 0;
}
table td {
  padding: 4px 0px;
}
table th {
  padding: 4px 0px;
  text-align: right;
}
.borderBottom {
  border-bottom: 1px solid #e8e8e8;
}
.sameWidth {
  width: 260px;
}
.th-right {
  text-align: right;
}
.th-top {
  /* text-align: top !important; */
  padding-top: 9px !important;
  vertical-align: top;
}
.label-title {
  margin: 30px 0 10px 0;
  color: #555;
}
.projectInfo .bussT {
  position: absolute;
  width: 100%;
  z-index: 2;
  background: #fff;
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 2px;
}
</style>
