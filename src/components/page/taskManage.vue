<template>
  <div>
    <div>
      <div class="mb20 mt20">
        <span>
          <span>首页</span>
          <span class="fg">/</span>
          <a>任务管理</a>
        </span>
      </div>
    </div>
    <div class="searchBox">
      <el-form :model="searchForm" ref="searchForm" class="form-item" label-width="80px" :inline="true">
        <el-row>
          <el-col :xs="24" :span='24'>
            <el-form-item label="国家">
              <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
              <el-checkbox-group v-model="searchForm.checkedCities" @change="handleCheckedCitiesChange">
                <el-checkbox v-for="(item,index) in cities" :label="item.Id" :key="index">{{item.CountryName}}</el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-col>
        </el-row>
        <div>
          <el-form-item label="搜索内容" class="labelNum">
            <el-input v-model="searchForm.Keyword" style="width: 250px" placeholder="任务编码 / 产品名称 / 购买单号"></el-input>
          </el-form-item>
          <el-form-item label="任务类型">
            <el-select v-model="searchForm.serveType" placeholder="请选择">
              <el-option :value="0" label="全部"></el-option>
              <el-option :value="1" label="评后返（代返）"></el-option>
              <el-option :value="2" label="评后返（自返）"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click='searchOrder' size="medium">搜索</el-button>
            <el-button @click="resetTask" size="medium">重置</el-button>
          </el-form-item>
          <el-button type="warning" size="medium" @click="exportExcel" style="float: right;">导出任务</el-button>
        </div>
      </el-form>
    </div>
    <div>
      <div class="tabList">
        <ul class="tabBlock" v-for="(item,index) in statusList" :key=index>
          <li :class="active === 0 ? 'active':''" @click="allTaskNum" :data-index="0">全部<span>({{Number(item.TotalCount)}})</span></li>
          <li :class="active === 1 ? 'active':''" :data-index="1" @click="daifp">待分配<span>({{Number(item.OrderStateInOne)}})</span></li>
          <li :class="active === 2 ? 'active':''" :data-index="2" @click="daiBuy">待购买<span>({{Number(item.OrderStateInTwo)}})</span></li>
          <li :class="active === 3 ? 'active':''" :data-index="3" @click="daiComfir">订单待确认<span>({{Number(item.OrderStateInThree)}})</span></li>
          <li :class="active === 4 ? 'active':''" :data-index="4" @click="daipj">待评价<span>({{Number(item.OrderStateInFour)}})</span></li>
          <li :class="active === 5 ? 'active':''" :data-index="5" @click="evaluationComfir">评价待确认<span>({{Number(item.OrderStateInFive)}})</span></li>
          <li :class="active === 6 ? 'active':''" :data-index="6" @click="ywc">已完成<span>({{Number(item.OrderStateInSix)}})</span></li>
          <li :class="active === 7 ? 'active':''" :data-index="7" @click="taskCancel">已取消<span>({{Number(item.OrderStateInSeven)}})</span></li>
          <li :class="active === 8 ? 'active':''" :data-index="8" @click="errTask">异常<span>({{Number(item.OrderStateInEight)}})</span></li>
        </ul>
      </div>
    </div>
    <div class="mt10 tableBg" style="overflow-x: auto">
      <el-table :data="allOrderData" id="exportTable" border style="width: 100%;font-size: 15px;" :header-cell-style="{background:'#eef1f6'}">
        <el-table-column prop="OrderNumbers" label="任务编码" align="center" width="170">
          <template slot-scope="scope">
            <el-link type="primary" :underline="false" @click="viewDetails(scope.$index,scope.row)">{{scope.row.OrderNumbers}}</el-link>
            <p>
              <span v-if="scope.row.NoComment==1"><span style="color: #F56C6C;font-size: 10px;">免评单</span></span>
            </p>
          </template>
        </el-table-column>
        <el-table-column prop="countryName" label="国家" align="center"></el-table-column>
        <el-table-column prop="ServiceType" label="任务类型" align="center"></el-table-column>
        <el-table-column prop="Asin" label="产品ASIN" align="center"></el-table-column>
        <el-table-column prop="ProductName" label="产品名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="AmazonNumber" label="购买单号" align="center"></el-table-column>
        <el-table-column prop="AmazonProductPrice" label="购买价格" align="center"></el-table-column>
        <el-table-column prop="BuyTime" label="购买时间" align="center" width="160"></el-table-column>
        <el-table-column prop="PayAccount" label="返款账号" align="center"></el-table-column>
        <el-table-column prop="state" label="状态" align="center" :formatter="txtOrderStatus"></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button size="small" v-if="scope.row.state==1" type="danger" @click="cancelHandel(scope.$index,scope.row)">取消</el-button>
            <el-button size="small" type="primary" v-if="scope.row.state==3" @click="confirmBtn(scope.$index,scope.row)">确认订单</el-button>
            <el-button size="small" v-if="scope.row.state==5" type="success" @click="evalEdit(scope.$index,scope.row,1)">确认评价</el-button>
            <el-button size="small" v-if="scope.row.state==6 && scope.row.ServiceType=='评后返（自返）'" type="success" @click="evalEdit(scope.$index,scope.row,2)">补充返款信息</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="mt30 txtRight" v-if="total>0">
        <el-pagination background @size-change='handleSizeChange' @current-change="handleCurrentChange" :current-page="currentPage"
          :page-sizes="[10, 20, 30, 50,100]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>
    </div>

    <!--评价-->
    <el-dialog :title="commentTitle" :visible.sync="assessModel" center :close-on-click-modal="false" :before-close="assessModelClose"
      width="30%">
      <el-form :model="assessForm" :rules="assessFormRules" label-width="100px" ref="assessForm">
        <el-form-item label="返款账号：" v-if="serviceType=='评后返（自返）'">
          <span>{{assessForm.PPaccount}}</span>
        </el-form-item>
        <el-form-item label="评价链接：">
          <span>{{assessForm.productLink}}</span>
        </el-form-item>
        <el-form-item label="评价截图：">
          <img v-if="assessForm.ProductImage" @click="showBigImg(GLOBAL.IMG_URL+assessForm.ProductImage)" style="width: 150px;height: 150px;cursor: pointer;"
            :src="this.GLOBAL.IMG_URL+assessForm.ProductImage" class="proImg" />
        </el-form-item>
        <div v-if="serviceType=='评后返（自返）'">
          <el-form-item label='交易截图：' class="mt20 p-img">
            <el-upload class="avatar-uploader" name="image" :action="uploadUrl" :show-file-list="false" :on-success="handleAvatarSuccess"
              :on-error="handleError" :before-upload="beforeAvatarUpload" accept="image/jpeg,image/png,image/gif,image/bmp">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
            <el-input v-show="false" v-model='assessForm.ProductPictures'></el-input>
          </el-form-item>
          <!-- 编号小于100018是公司内人 -->
          <el-form-item label="返款金额：" prop="backMoney" v-if="loginUserId<=100018">
            <el-input v-model="assessForm.backMoney"></el-input>
          </el-form-item>
        </div>
      </el-form>
      <span slot='footer' class='dialog-footer'>
        <el-button type="primary" @click='evalEditSubmit'>确认</el-button>
        <el-button @click="assessModelClose">关闭</el-button>
      </span>
    </el-dialog>

    <!-- 确认订单 -->
    <el-dialog title="确认订单" :visible.sync="submitModal" :close-on-click-modal="false" center :before-close="closeSubmit">
      <el-form :model="orderForm" :rules="orderOkRules" ref="orderForm" style="padding: 0 30px;">
        <el-form-item label="订单状态：">
          <el-radio-group v-model="orderForm.orderStatus" @change="orderStateChange">
            <el-radio label="1">正常</el-radio>
            <el-radio label="0">异常</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="购买截图：">
          <img v-if="orderForm.buyImage" @click="showBigImg(GLOBAL.IMG_URL+orderForm.buyImage)" style="width: 150px;height: 150px;cursor: pointer;"
            :src="this.GLOBAL.IMG_URL+orderForm.buyImage" class="proImg" />
        </el-form-item>
        <el-form-item label="购买单号：">
          <span>{{orderForm.buyNumber}}</span>
        </el-form-item>
        <el-form-item label="购买时间：">
          <span>{{orderForm.buyTime}}</span>
        </el-form-item>
        <el-form-item label="购买价格：" prop="buyMoney">
          <el-input v-model="orderForm.buyMoney" :disabled="orderForm.orderStatus=='0'"></el-input>
        </el-form-item>
        <el-form-item label="备注：" prop="remark" v-if="orderForm.orderStatus=='1'">
          <el-input type="textarea" rows="3" v-model="orderForm.remark"></el-input>
        </el-form-item>
        <el-form-item label="异常备注：" prop="orderRemark" v-if="orderForm.orderStatus=='0'">
          <el-input type="textarea" rows="3" v-model="orderForm.orderRemark"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" size="medium" @click='confirmSubmit'>确定</el-button>
        <el-button @click="closeSubmit" size="medium">关闭</el-button>
      </span>
    </el-dialog>

    <!--取消-->
    <el-dialog title="取消订单" :visible.sync="cancelModal" :close-on-click-modal="false" center width="30%">
      <div class="del-dialog-cnt">是否确定取消该任务?</div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click='cancelSubmit'>确定</el-button>
        <el-button @click="cancelModal=false">关闭</el-button>
      </span>
    </el-dialog>
    <!--FBA查看详情-->
    <el-dialog title='任务详情' :visible.sync="viewDetailModal" width='70%'>
      <view-task :viewTaskData="viewTaskData"></view-task>
      <span slot='footer' class="dialog-footer txtCenter">
        <el-button type='primary' @click='viewDetailModal=false'>关闭</el-button>
      </span>
    </el-dialog>
    <!--查看大图-->
    <el-dialog :title='titlePic' :visible.sync='imageModal' :close-on-click-modal='false' width="98%">
      <div class="txtCenter">
        <img :src='this.bigImgUrl' style="width: 100%;" />
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="imageModal=false">关 闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import viewTask from '../common/viewTaskDetails'
  import FileSaver from 'file-saver'
  import XLSX from 'xlsx'
  import {
    getCountry,
    taskList,
    taskStatus,
    taskConfirm,
    taskCancel,
    Rate
  } from '@/request/api'
  export default {
    name: 'taskManage',
    data() {
      return {
        loading: true, //列表加载
        serviceType: 0,
        hideUpload: false, //评论上传图片继续添加按钮
        limitCount: 1, //图片数量
        orderForm: {
          orderStatus: '1', //确认订单状态
          buyImage: '',
          buyNumber: '',
          buyTime: '',
          buyMoney: '',
          remark: '', //正常备注
          orderRemark: '', //异常备注
        },
        checkAll: false,
        cities: [],
        isIndeterminate: true,
        OrderId: '', //任务ID
        uploadUrl: this.axios.defaults.baseURL + '/api/Order/GetProductPictures',
        viewTaskData: [],
        allOrderData: [],
        hasBalance: 0,
        pageSize: 10, //每页条数
        total: 0, //总条数
        currentPage: 1, //页数
        cancelModal: false,
        submitModal: false, //确认
        viewDetailModal: false, //查看详情
        active: 0,
        assessModel: false,
        startTime: '',
        pickerEndDate: this.pickerOptionsEnd(),
        pickerStartDate: this.searchStartDate(),
        paymentForm: {
          balance: 0,
          payMoney: 0,
        },
        searchForm: {
          Keyword: '',
          orderStartTime: '',
          orderEndTime: '',
          checkedCities: [],
          serveType: 0,
        },
        // 评价
        assessForm: {
          PPaccount: '',
          productLink: '', //产品链接
          ProductPictures: '', //交易截图
          backMoney: '' //返款金额
        },
        NoComment: '', //是否免评单
        imageUrl: '',
        obj: [],
        StatusSum: [], //任务状态数量
        statusList: [], //全部状态
        rateData: [], // 费率
        imageModal: false, //大图弹框
        titlePic: '', //大图标题
        orderOkRules: {
          buyMoney: [{
              required: true,
              message: '请输入购买金额',
              trigger: 'blur'
            },
            {
              pattern: /^[0-9]+([.]{1}[0-9]+){0,1}$/,
              message: '金额式不正确',
              trigger: 'blur'
            }
          ],
          orderRemark: [{
            required: true,
            message: '请输入异常备注',
            trigger: 'blur'
          }]
        },
        bigImgUrl: '',
        loginUserId: '',
        assessFormRules: {
          backMoney: [{
              required: true,
              message: '请输入返款金额',
              trigger: 'blur'
            },
            {
              pattern: /^[0-9]+([.]{1}[0-9]+){0,1}$/,
              message: '金额式不正确',
              trigger: 'blur'
            }
          ]
        },
        commentTitle: ''
      }
    },
    components: {
      viewTask
    },
    created() {
      this.getAllCountry()
      this.getAllData()
      this.getAllStatus()
      this.getRateData() //获取汇率数据
    },
    methods: {
      // 图片上传
      handleAvatarSuccess(res, file) {
        if (res.Data != '') {
          this.assessForm.ProductPictures = res.Data
        }
        this.imageUrl = URL.createObjectURL(file.raw);
        this.$message.success('交易截图上传成功！')
      },
      handleError(res) {
        this.$message.error('交易截图上传失败！')
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg';
        const isPNG = file.type === 'image/png';
        const isGIF = file.type === 'image/gif';
        const isBMP = file.type === 'image/bmp';
        const isLt5M = file.size / 1024 / 1024 < 5;
        if (!isJPG && !isPNG && !isGIF && !isBMP) {
          this.$message.error('上传图片必须是JPG/PNG/GIF/BMP 格式!');
        } else if (!isLt5M) {
          this.$message.error('上传图片大小不能超过 5MB!');
        }
        return (isJPG || isPNG || isGIF || isBMP) && isLt5M;
      },

      // 格式化状态
      txtOrderStatus(val) {
        if (val.state == 1) {
          return '待分配'
        } else if (val.state == 2) {
          return '待购买'
        } else if (val.state == 3) {
          return '订单待确认'
        } else if (val.state == 4) {
          return '待评价'
        } else if (val.state == 5) {
          return '评价待确认'
        } else if (val.state == 6) {
          return '已完成'
        } else if (val.state == 7) {
          return '已取消'
        } else if (val.state == 8) {
          return '异常'
        }
      },
      // 初始化状态
      getAllStatus() {
        let _this = this
        let userId = sessionStorage.getItem('userId')
        let param = {
          userId: userId,
          countryIdx: _this.searchForm.checkedCities,
          kWord: _this.searchForm.Keyword,
          type: _this.searchForm.serveType
        }
        taskStatus(param).then(res => {
          _this.statusList = res.data.list
        }).catch((e) => {})
      },
      // 初始化任务列表
      getAllData() {
        let _this = this
        let userId = sessionStorage.getItem('userId')
        _this.loginUserId = userId
        let param = {
          pageNum: _this.currentPage,
          pagesize: _this.pageSize,
          userId: userId, //用户id
          countryIdx: _this.searchForm.checkedCities, //国家数组
          kWord: _this.searchForm.Keyword, //查询（任务编码，产品名称）
          state: parseInt(_this.active), //任务状态
          type: _this.searchForm.serveType
        }
        taskList(param).then(res => {
          _this.loading = false
          _this.allOrderData = res.data.list
          _this.total = parseInt(res.data.total)
          let lists = res.data.list
          for (let i = 0; i < lists.length; i++) {
            let types = lists[i].ServiceType
            if (types == 1) {
              lists[i].ServiceType = '评后返（代返）'
            } else if (types == 2) {
              lists[i].ServiceType = '评后返（自返）'
            }
            _this.allOrderData[i].state = _this.allOrderData[i].TaskState
          }
        }).catch((e) => {})
      },
      // 初始化国家查询
      getAllCountry() {
        let _this = this
        let param = {
          Id: sessionStorage.getItem('userId')
        }
        getCountry(param).then((res) => {
          _this.cities = res.data.list
        }).catch((e) => {})
      },
      // 国家全选
      handleCheckAllChange(val) {
        let _this = this
        val ? (_this.searchForm.checkedCities = _this.cities.map((res) => {
          return res.Id
        })) : (_this.searchForm.checkedCities = [], _this.isIndeterminate = false);
        _this.isIndeterminate = false;
      },
      // 国家选择
      handleCheckedCitiesChange(value) {
        let checkedCount = value.length;
        this.checkAll = checkedCount === this.cities.length;
        this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length;
      },

      //更新余额
      getBalance() {
        let _this = this
        let param = {
          SessionId: sessionStorage.getItem('sessionid')
        }
        _this.axios.post(this.GLOBAL.BASE_URL + '/api/frontConsoleHome', param).then((res) => {
          if (res.data.status == 200) {
            sessionStorage.setItem('userId', res.data.data.userid)
            sessionStorage.setItem('balance', res.data.data.balance)
          }
          if (res.data.status == 400) {
            _this.$message({
              type: 'error',
              message: '登录过期，请重新登录'
            })
            _this.$router.push({
              name: 'index',
              params: {
                indexShow: false
              }
            })
            sessionStorage.clear()
          }
        })
      },
      //订单确认完成弹窗
      confirmBtn(index, row) {
        let _this = this
        _this.submitModal = true
        _this.OrderId = row.Id
        _this.orderForm.buyImage = row.BuyImage
        _this.orderForm.buyNumber = row.AmazonNumber
        _this.orderForm.buyTime = row.BuyTime
        _this.orderForm.buyMoney = row.AmazonProductPrice
      },
      //订单确认完成确定
      confirmSubmit() {
        let _this = this
        _this.$refs.orderForm.validate((valid) => {
          if (valid) {
            let userId = sessionStorage.getItem('userId')
            let state = _this.orderForm.orderStatus
            let remark = ''
            if (state == '1') {
              remark = _this.orderForm.remark
            }
            if (state == '0') {
              remark = _this.orderForm.orderRemark
            }
            let param = {
              Id: _this.OrderId,
              UserId: userId,
              State: Number(state),
              Remark: remark,
              AmazonProductPrice: _this.orderForm.buyMoney
            }
            taskConfirm(param).then(res => {
              if (res.data.Code == 'ok') {
                _this.$message({
                  type: 'success',
                  message: res.data.Msg
                })
              } else {
                _this.$message({
                  type: 'error',
                  message: res.data.Msg
                })
              }
              _this.closeSubmit()
              _this.getAllData()
              _this.getAllStatus()
            }).catch(error => {})
          }
        })
      },

      //关闭订单确认
      closeSubmit() {
        let _this = this
        _this.submitModal = false
        _this.$refs['orderForm'].resetFields()
        _this.orderForm = {
          orderStatus: '1',
          remark: '',
          orderRemark: '',
          buyImage: '',
          buyNumber: '',
          buyTime: '',
          buyMoney: '',
        }
      },

      //切换订单确认状态
      orderStateChange() {
        let _this = this
        _this.$refs['orderForm'].resetFields()
      },

      // 重置
      resetTask() {
        let _this = this
        _this.searchForm = {
          Keyword: '',
          orderStartTime: '',
          orderEndTime: '',
          checkedCities: [],
          serveType: 0
        }
        _this.currentPage = 1
        _this.getAllData()
        _this.getAllStatus()
      },

      // 取消
      cancelHandel(index) {
        let _this = this
        _this.cancelModal = true
        _this.OrderId = _this.allOrderData[index].Id

      },
      //取消确定
      cancelSubmit() {
        let _this = this
        let orderId = _this.OrderId
        let param = {
          UserId: sessionStorage.getItem('userId'),
          Id: orderId,
          UserImage: ''
        }
        taskCancel(param).then(res => {
          if (res.data.Code == 'ok') {
            _this.$message({
              type: 'success',
              message: '操作成功'
            })
            _this.cancelModal = false
            _this.getAllData()
            _this.getAllStatus()
          }
        }).catch(error => {
          console.log(error)
        })
      },
      searchStartDate() {
        return {
          disabledDate: time => {
            let endDateVal = this.searchForm.orderEndTime
            if (endDateVal) {
              return time.getTime() > new Date(endDateVal).getTime()
            }
          }
        }
      },
      // 搜索下单结束时间
      pickerOptionsEnd() {
        return {
          disabledDate: time => {
            let beginDateVal = this.searchForm.orderStartTime
            if (beginDateVal) {
              return (
                time.getTime() <
                new Date(beginDateVal).getTime() - 1 * 24 * 60 * 60 * 1000
              )
            }
          }
        }
      },
      endTimeStatus: function(value) {
        this.orderEndTime = value
      },

      // 填写评价
      evalEdit(index, row, val) {
        let _this = this
        if (val == '1') {
          _this.commentTitle = '确认评价'
        }
        if (val == '2') {
          _this.commentTitle = '补充返款信息'
        }
        _this.assessModel = true
        _this.OrderId = row.Id
        _this.serviceType = row.ServiceType
        _this.assessForm.PPaccount = row.PayAccount
        _this.assessForm.productLink = row.ProductLink
        _this.assessForm.ProductImage = row.ProductImage
        _this.assessForm.backMoney = row.DealMoeny
        _this.assessForm.ProductPictures = row.DealIamge
        _this.imageUrl = row.DealIamge
        _this.NoComment = Number(row.NoComment)
      },
      //评论确定
      evalEditSubmit() {
        let _this = this
        _this.$refs.assessForm.validate((valid) => {
          if (valid) {
            let money = _this.assessForm.backMoney
            if (!money) {
              money = 0
            }
            let type = _this.serviceType
            let uId = sessionStorage.getItem('userId')
            let JYimg = _this.assessForm.ProductPictures
            if (type == '评后返（自返）' && uId <= 100018 && !JYimg) {
              this.$message.error('自返内单必须上传交易截图！')
            } else {
              let param = {
                UserId: uId,
                Id: _this.OrderId,
                UserImage: JYimg,
                BackMoney: money,
                NoComment: _this.NoComment
              }
              taskCancel(param).then(res => {
                if (res.data.Code == 'ok') {
                  _this.$message({
                    type: 'success',
                    message: '操作成功'
                  })
                  _this.assessModelClose()
                  _this.getAllData()
                  _this.getAllStatus()
                } else {
                  _this.$message({
                    type: 'error',
                    message: res.data.Msg
                  })
                }
              }).catch(error => {})
            }
          }
        })
      },

      //确认评价窗口关闭
      assessModelClose() {
        let _this = this
        _this.assessModel = false
        _this.OrderId = ''
        _this.serviceType = ''
        _this.assessForm.PPaccount = ''
        _this.assessForm.productLink = ''
        _this.assessForm.ProductImage = ''
        _this.assessForm.ProductPictures = ''
        _this.assessForm.backMoney = 0
        _this.imageUrl = ''
        _this.NoComment = ''
      },

      // 查看任务详情
      viewDetails(index, row) {
        let _this = this
        let obj = _this.obj
        _this.viewDetailModal = true
        _this.viewTaskData = _this.allOrderData[index]
        let val = _this.allOrderData[index]
        if (val.TaskState == 1) {
          _this.viewTaskData.TaskState = '任务待分配'
        } else if (val.TaskState == 2) {
          _this.viewTaskData.TaskState = '待购买'
        } else if (val.TaskState == 3) {
          _this.viewTaskData.TaskState = '订单待确认'
        } else if (val.TaskState == 4) {
          _this.viewTaskData.TaskState = '待评价'
        } else if (val.TaskState == 5) {
          _this.viewTaskData.TaskState = '评价待确认'
        } else if (val.TaskState == 6) {
          _this.viewTaskData.TaskState = '已完成'
        } else if (val.TaskState == 7) {
          _this.viewTaskData.TaskState = '已取消'
        } else if (val.TaskState == 8) {
          _this.viewTaskData.TaskState = '异常'
        } else {
          return
        }
        _this.viewTaskData.symbol = _this.getSymbol(_this.rateData, row.CountryId) //货币符号
      },

      // 汇率数据
      getRateData() {
        let _this = this
        Rate().then((res) => {
          _this.rateData = res.data.list
        }).catch(err => {
          console.log(err)
        })
      },

      // 取货币符号
      getSymbol(obj, param) {
        let _this = this
        for (let i = 0; i < obj.length; i++) {
          if (obj[i].CountryId == param) {
            return obj[i].CurrencySymbol
          }
        }
      },

      startDate() {
        return {
          disabledDate(time) {
            return time.getTime() < Date.now() - 8.64e7
          }
        }
      },
      handleClick(tab, event) {
        console.log(tab)
      },

      //查询
      searchOrder() {
        let _this = this
        _this.getAllData()
        _this.getAllStatus()
      },
      //上传图片选择
      openFile() {
        this.$refs.file.click();
      },
      //上传视频选择
      openVideo() {
        this.$refs.files.click();
      },
      // 分页导航
      handleCurrentChange(val) {
        let _this = this
        _this.currentPage = val
        _this.getAllData()
      },
      //每页条数
      handleSizeChange(val) {
        let _this = this
        _this.pageSize = val
        _this.handleCurrentChange(_this.currentPage)
        _this.getAllData()
      },
      //全部
      allTaskNum() {
        let _this = this
        _this.active = 0
        _this.getAllData()
      },
      //任务待分配
      daifp() {
        let _this = this
        _this.active = 1
        _this.getAllData()
      },
      //待购买
      daiBuy() {
        let _this = this
        _this.active = 2
        _this.getAllData()
      },
      //订单待确认
      daiComfir() {
        let _this = this
        _this.active = 3
        _this.getAllData()
      },
      // 待评价
      daipj() {
        let _this = this
        _this.active = 4
        _this.getAllData()
      },
      // 评价待确认
      evaluationComfir() {
        let _this = this
        _this.active = 5
        _this.getAllData()
      },
      // 已完成
      ywc() {
        let _this = this
        _this.active = 6
        _this.getAllData()
      },
      // 已取消
      taskCancel() {
        let _this = this
        _this.active = 7
        _this.getAllData()
      },
      //其他任务异常
      errTask() {
        let _this = this
        _this.active = 8
        _this.getAllData()
      },

      //展示大图
      showBigImg(url) {
        let _this = this
        _this.titlePic = '查看大图'
        _this.imageModal = true
        _this.bigImgUrl = url
      },

      // 导出
      exportExcel() {
        var xlsxParam = {
          raw: true
        }
        var wb = XLSX.utils.table_to_book(document.querySelector('#exportTable'), xlsxParam)
        var wbout = XLSX.write(wb, {
          bookType: 'xlsx',
          bookSST: true,
          type: 'array'
        })
        try {
          FileSaver.saveAs(new Blob([wbout], {
            type: 'application/octet-stream'
          }), '任务数据.xlsx')
        } catch (e) {
          if (typeof console !== 'undefined') {
            console.log(e, wbout)
          }
        }
        return wbout
      }

    }
  }
</script>

<style>
  /* 上传组件相关样式 */
  .p-img .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .p-img .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }

  .p-img .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 150px;
    height: 150px;
    line-height: 150px !important;
    text-align: center;
  }

  .p-img .avatar {
    width: 150px;
    height: 150px;
    display: block;
  }

  .p-img .el-upload--text {
    width: 150px;
    height: 150px;
  }

  .fileBtn {
    display: inline-block;
    width: 120px;
    height: 40px;
    background: #0098EF;
    color: #fff;
    text-align: center;
    line-height: 40px;
    cursor: pointer;
  }

  .comImg {
    width: 200px;
    background-size: cover;
    border: solid #ccc 1px;
  }

  .imgItem {
    position: relative;
    z-index: 9;
    display: inline-block;
    cursor: pointer;
    width: 200px;
    height: 115px;
    margin: 0 10px 10px 10px;
  }

  .imgItem:hover .imgDelModal {
    display: inline-block;
    position: absolute;
    width: 200px;
    height: 115px;
    top: 50%;
    left: 50%;
    background: rgba(0, 0, 0, .4);
    z-index: 99;
    transform: translate(-50%, -50%);
  }

  .delIcon,
  .imgDelModal {
    display: none;
  }

  .imgItem:hover .delIcon {
    display: inline-block;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 9999;
    font-size: 26px;
    color: #fff;
    transition: all ease-in-out .5s;
  }

  .detailbox1 li {
    width: 48%;
    float: left;
    list-style: none;
    line-height: 36px;
    margin-bottom: 8px;
    overflow: hidden;
  }

  .detailbox1 li p {
    margin: 0;
    display: inline-block;
    color: #666;
  }

  .detailbox1 li p img {
    display: inline-block;
  }

  .detailbox {
    overflow: hidden;
    margin-top: 20px;
    font-size: 14px;
  }

  .eval_img {
    width: 60px;
    height: 60px;
    border-radius: 4px;
    cursor: pointer;
    border: 1px solid #ccc;
    margin-right: 10px;
  }

  .detailbox1 {
    padding: 0;
    margin: 0 0 0 10px;
    text-align: left;
  }

  @media screen and (max-width: 970px) {
    .detailbox1 li {
      width: 100%;
    }
  }

  @media screen and (min-width: 1440px) {
    .dialogModal {
      width: 60% !important;
      margin: 0 auto;
    }
  }
</style>
