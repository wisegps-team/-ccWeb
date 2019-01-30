<template>
  <div>
    <a-card :loading="loading" :bordered="false" :body-style="{padding: '0',minWidth:'1200px'}">
      <a-tabs
        default-active-key="1"
        size="large"
        :tab-bar-style="{marginBottom: '12px', paddingLeft: '0'}"
      >
        <a-tab-pane loading="true" :tab="title" key="1">
          <!-- 操作栏 -->
          <a-row :style="{marginBottom: '12px'}">
            <a-col :xl="12" :lg="12" :md="12" :sm="24" :xs="24">
              <a-button type="primary" icon="plus" @click="createProject(true)">创建项目</a-button>
            </a-col>

            <a-col :xl="12" :lg="12" :md="12" :sm="24" :xs="24">
              <div :style="{float: 'right',width: '80%',maxWidth:'450px'}">
                <a-button type="primary" icon="search" :style="{width:'25%'}">选择部门</a-button>
                <a-input-search
                  :style="{width:'75%'}"
                  placeholder="项目编号、项目名称、项目负责人、委托人"
                  @search="onSearch"
                  enterButton="搜索"
                />
              </div>
            </a-col>
          </a-row>
          <!-- 列表 -->
          <a-row>
            <a-table
              :size="size"
              :bordered="bordered"
              :loading="loading"
              :columns="columns"
              :dataSource="current"
              :pagination="pagination"
              :scroll="{ x: 1700 }"
              :customRow="changeRow"
            >
              <a slot="description" slot-scope="text" href="javascript:;">{{ text }}</a>

              <!-- 合同编号slot -->
              <div slot="name" slot-scope="text,record,index" style="position:relative">
                <a
                  href="javascript:;"
                  class="ellipsis"
                  style="width:300px;display: block;"
                  :title="record.description"
                  @click="goContractDetail(record)"
                >{{ record.name }}</a>
                <span style="position:absolute;right:-5px;top:0px">
                  <a v-if="record.star">
                    <a-icon
                      type="star"
                      theme="filled"
                      style="color:#ffaf38;font-size:16px"
                      @click="changeStar(1,record,index)"
                    />
                  </a>
                  <a v-else-if="moveStar == record.projectNo">
                    <a-icon type="star" style="font-size:16px" @click="changeStar(2,record,index)"/>
                  </a>
                  <a>
                    <a-icon type="link"/>
                  </a>
                </span>
              </div>

              <!-- 签约日期筛选 -->
              <div
                slot="filterDropdown"
                slot-scope="{ setSelectedKeys, selectedKeys, confirm, clearFilters }"
                class="custom-filter-dropdown"
              >
                <a-modal
                  title="请选择时间段"
                  v-model="visibled"
                  :maskClosable="maskClosabled"
                  okText="确认"
                  cancelText="清空"
                >
                  <a-row>
                    <a-col :span="12">开始日期：
                      <a-date-picker style="width:50%"/>至
                    </a-col>
                    <a-col :span="12">结束日期：
                      <a-date-picker style="width:50%"/>
                    </a-col>
                  </a-row>
                  <!-- <a-input
                    ref="searchInput"
                    placeholder="Search name"
                    :value="selectedKeys[0]"
                    @change="e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
                    @pressEnter="() => handleSearch(selectedKeys, confirm)"
                  />-->
                  <!-- <a-button type="primary" @click="() => handleSearch(selectedKeys, confirm)">Search</a-button>
                  <a-button @click="() => handleReset(clearFilters)">Reset</a-button>-->
                </a-modal>
              </div>

              <span slot="action" slot-scope="text,record,index">
                <a class="font-blue" style="margin-right:5px">讨论</a>
                <a class="font-blue" style="margin-right:5px">归档</a>
                <a class="font-blue" style="margin-right:5px">核算</a>
                <a-dropdown :trigger="['click']">
                  <a class="ant-dropdown-link" href="#">
                    <a-icon type="down-circle" style="font-size:16px;color:#999"/>
                  </a>
                  <a-menu slot="overlay">
                    <a-menu-item key="3">
                      <a>
                        <a-icon type="share-alt"/>创建子项目
                      </a>
                    </a-menu-item>
                    <a-menu-item key="5">
                      <a>
                        <a-icon type="delete"/>删除项目
                      </a>
                    </a-menu-item>
                  </a-menu>
                </a-dropdown>
              </span>
            </a-table>
          </a-row>
        </a-tab-pane>
      </a-tabs>
    </a-card>
  </div>
</template>

<script>
import { getServiceList } from '@/api/manage'

import gql from 'graphql-tag'

export default {
  name: 'TableList',
  props: {
    title: {
      type: String,
      default: ''
    },
    bordered: {
      type: Boolean,
      default: true
    },

    moduleType: {
      type:Number,
      default:1
    },
    size: {
      type: String,
      default: 'small'
    },
  },
  data() {
    return {
      loading: true, //加载动态
      pageSizeOptions: ['10', '20', '30'],

      total: 0,
      currentPageSize: 10,
      defaultCurrent: 1,
      sortParams: {}, //排序参数
      queryParam: {},
      current: [],
      pagination: {},
      paramsName: {},
      maskClosabled: false,
      visibled: false,
      moveStar: '',
      selectedIndex: -1,
      modal1Visible: false,
      value: 2,
      radioStyle: {
        display: 'block',
        height: '30px',
        lineHeight: '30px'
      },
      // 表头项目编号，项目名称，业务类型，负责人，项目类型，送审金额，审定金额，已回款/咨询，截止日期，项目状态，项目进展，项目备注，操作（讨论 归档 核算 创建子项目 删除）
      columns: [
        {
          title: '项目编号',
          dataIndex: 'projectNo',
          sorter: true
        },
        {
          title: '项目名称',
          dataIndex: 'name',
          sorter: true,
          width: '350px',
          scopedSlots: { customRender: 'name' }
        },
        {
          title: '业务类型',
          dataIndex: 'businessType',
          sorter: true
          // needTotal: true,
          // customRender: text => text + ' 次'
        },
        {
          title: '负责人',
          dataIndex: 'principal',
          sorter: true
          // needTotal: true,
          // customRender: text => text + ' 次'
        },
        {
          title: '项目类型',
          dataIndex: 'ProjectType',
          sorter: true
          // needTotal: true,
          // customRender: text => text + ' 次'
        },
        {
          title: '送审金额',
          dataIndex: 'submitAmount',
          sorter: true
          // needTotal: true,
          // customRender: text => text + ' 次'
        },
        {
          title: '审定金额',
          dataIndex: 'approvedAmount',
          sorter: true
          // needTotal: true,
          // customRender: text => text + ' 次'
        },
        {
          title: '已回款/咨询',
          dataIndex: 'backAsck'
        },
        {
          title: '截止日期',
          dataIndex: 'endData',
          // sorter: true
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'customRender'
          },
          onFilter: (value, record) => {
            console.log(value, record)
          },
          onFilterDropdownVisibleChange: visible => {
            // debugger;
            // this._visible = visible
            if (visible) {
              this.visibled = visible
              setTimeout(() => {
                this.$refs.searchInput.focus()
              })
            }
          }
        },

        {
          title: '项目状态',
          dataIndex: 'status',
          // sorter: true
          filters: [
            {
              text: 'London',
              value: 'London'
            },
            {
              text: 'New York',
              value: 'New York'
            }
          ],
          onFilter: (value, record) => {
            console.log(value, record, 'record,value')
          }
        },
        {
          title: '项目进展',
          dataIndex: 'statuss'
        },
        {
          title: '项目备注',
          dataIndex: 'remark'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'right',
          width: '150px',
          fixed: 'right',
          scopedSlots: { customRender: 'action' }
        }
      ],
    }
  },
  created() {
    // //数据请求参数配置
    console.log('createList')
    this.loading = true
    this.loadList()
  },
  methods: {
    
    //接口获取数据
    loadList() {
      var _this = this
      var skip = (this.defaultCurrent -1) * this.currentPageSize
      this.$apollo
        .query({
          query: gql`
            query proje($skip: Int!, $pageSize: Int!) {
              projectsConnection(where: {}) {
                aggregate {
                  count
                }
              }
              projects(where: {}, skip: $skip, first: $pageSize) {
                id
                name
                principal
                projectNo
                businessType
                projectType
                submitAmount
                approvedAmount
                status
                remark
                steps {
                  id
                  name
                  order
                  planTime
                  finishTime
                  status
                }
                plans {
                  id
                  name
                  status
                  startDate
                  endDate
                  principal
                  remark
                  members
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
          _this.total = res.data.projectsConnection.aggregate.count
          _this.current = res.data.projects
          _this.pagination = _this.pager()
          _this.loading = res.loading
        })
        .catch(err => {
          console.log(err)
          _this.total = 0
          _this.defaultCurrent = 1
          _this.current = [];
          _this.pagination = _this.pager()
          _this.loading = false
        })
    },

    onPagerChange(page, pageSize) {
      this.defaultCurrent = page
      this.currentPageSize = pageSize
      this.loading = true
      this.loadList()
      // console.log(page, pageSize,'changePager')
      // this.loadData({ pageNum: page })
    },
    onPagerSizeChange(current, size) {
      // console.log(current, size,'changeSize');
      // debugger
      this.defaultCurrent = current
      this.currentPageSize = size
      this.loading = true
      this.loadList()
      // this.loadData({ pageSize: size })
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

    onSearch() {},
    //表格单元格自定义函数
    changeRow(record, index) {
      return {
        props: {},
        style: {
          // backgroundColor:this.selectedIndex == index ? 'yellow':''
        },
        attrs: {
          id: record.projectNo
        },
        ref: record.projectNo,
        // refInFor: true,
        on: {
          // 事件
          click: () => {
            // 鼠标移入行,改变状态
            var allChildren = document.getElementById(record.projectNo).parentNode.children
            for (var i = 0; i < allChildren.length; i++) {
              if (i == index) {
                // console.log(allChildren[i].className = allChildren[i].className + ' dd' )
                allChildren[i].style.backgroundColor = '#ffe48d'
              } else {
                allChildren[i].style.backgroundColor = 'white'
              }
            }
          },
          mouseover: () => {
            // 鼠标移入行,改变状态
            // console.log(record,index,'index2')

            this.moveStar = record.projectNo
            // console.log(this.dataset)
          },
          mouseout: () => {
            // 鼠标移入行,改变状态
            // console.log(record,index,'index3')
            this.moveStar = false
          }
        }
      }
    },
    //置顶事件
    changeStar(type, record, index) {
      if (type == 2) {
        record.star = true
        this.current.splice(index, 1, record)
      } else {
        record.star = false
        this.current.splice(index, 1, record)
      }
    },
    handleSearch(selectedKeys, confirm) {
      confirm()
      this.searchText = selectedKeys[0]
    },

    handleReset(clearFilters) {
      clearFilters()
      this.searchText = ''
    },

    // showModal() {
    //   this.modal1Visible = true
    // },
    //跳进创建项目界面
    createProject(bol) {
      if (bol) {
        this.$emit('changeCom', { type: this.value })
      }
      this.modal1Visible = false
    },
    goContractDetail(params) {
      console.log(params)
      this.$emit('changeCom', { type: 5, params:params})
    }
  },
  watch: {
    moduleType(newT,oldT){
      console.log(newT,oldT,'moduleType')
    }
    // 'selectedRows': function (selectedRows) {
    //   this.needTotalList = this.needTotalList.map(item => {
    //     return {
    //       ...item,
    //       total: selectedRows.reduce( (sum, val) => {
    //         return sum + val[item.dataIndex]
    //       }, 0)
    //     }
    //   })
    // }
  }
}
</script>

<style>
.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>
