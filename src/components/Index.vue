<template>
  <div>
    <el-container class="main-container">
      <el-header style="background: #08163d;color:#fff;padding: 0 20px" height="60px">
        <div style="display:-webkit-box;display:-ms-flexbox;display:flex;flex-direction: row;height: 60px;width: 100%"
             @click="runSearch=false">
          <div align="left"
               style="display:-webkit-box;display:-ms-flexbox;display:flex;height: 60px;align-items: center;flex: 0 0 auto;margin-right:10px">
            <img src="../assets/img/icon_logo.svg" style="display:inline-block;height: 30px;width: 30px">
            <span style="display:inline-block;font-size:16px;margin-left:10px;color:#22CEFC">图码联侦实战布控平台</span>
          </div>
          <div class="head-menu"
               style="flex:1 1 auto;height:60px;align-items:center;justify-content:flex-start;white-space:nowrap;overflow:hidden">
            <el-menu :default-active="$route.path" background-color="#08163d" text-color="#B3B3B3" router
                     active-text-color="#3FA8F3" mode="horizontal" @select="handleSelectItem">
              <template v-for="item in menu">
                <el-submenu :index="item.permissionUrl" v-if="item.permissionUrl!='/searchAll'&&item.childs">
                  <template slot="title">
                    <i :class="item.icon" v-bind:style="item.orders>5?'font-size: 1.6em':'font-size: 1.3em'"></i>
                    <span>{{item.name}}</span>
                    <el-badge class="mark" :value="imsiCount+faceCount" :max="99"
                              v-if="item.permissionUrl=='/imsiWarnings'&&(imsiCount+faceCount)>0"/>
                    <el-badge class="mark" :value="turnCount" :max="99"
                              v-if="item.permissionUrl=='/myApply'&&turnCount>0"/>
                  </template>
                  <el-menu-item :index="child.permissionUrl" v-for="child in item.childs" :key="child.permissionUrl"
                                v-if="child.permissionUrl!='/searchAll'">
                    <template slot="title">
                      <span>{{child.name}}</span>
                      <el-badge class="mark" :value="imsiCount" :max="99"
                                v-if="child.permissionUrl=='/imsiWarnings'&&imsiCount>0"/>
                      <el-badge class="mark" :value="faceCount" :max="99"
                                v-if="child.permissionUrl=='/catchWarnings'&&faceCount>0"/>
                      <el-badge class="mark" :value="turnCount" :max="99"
                                v-if="child.permissionUrl=='/myApproval'&&turnCount>0"/>
                    </template>
                  </el-menu-item>
                </el-submenu>
                <el-menu-item :index="item.permissionUrl" v-if="item.permissionUrl!='/searchAll'&&!item.childs">
                  <template slot="title">
                    <i :class="item.icon"></i>
                    <span>{{item.name}}</span>
                  </template>
                </el-menu-item>
              </template>
            </el-menu>
          </div>
          <div align="right"
               style="display:-webkit-box;display:-ms-flexbox;display:flex;height: 60px;flex: 0 0 auto;align-items: center">
            <div class="all-search" style="margin-right: 15px">
              <el-popover ref="imsi" placement="bottom-end" width="200" trigger="click">
                <el-row style="background-color: #efefef">
                  <el-col :span="12" style="text-align: left;padding-left: 10px">
                    <span style="height: 40px;line-height: 40px;color:#888">搜索记录</span>
                  </el-col>
                  <el-col :span="12" style="text-align: right;padding-right: 10px">
                    <el-button type="text" @click="clearImsi()">清空</el-button>
                  </el-col>
                </el-row>
                <el-row v-for="(item,idx) in searchImsis" :key="idx+''" v-show="searchImsis.length>0">
                  <div @click.stop="imsi=item;searchImsi()" style="cursor: pointer">
                    <el-col :span="18" style="text-align: left;padding-left: 10px" align="left">
                      <span style="height: 40px;line-height: 40px;color:#333">{{item}}</span>
                    </el-col>
                    <el-col :span="6" style="text-align: right;padding-right: 10px" align="right">
                      <el-button type="text">IMSI</el-button>
                    </el-col>
                  </div>
                </el-row>
                <div v-show="searchImsis.length==0"
                     style="height: 40px;line-height: 40px;font-size:13px;color:#999;text-align: center;width: 194px">
                  无记录
                </div>
              </el-popover>
              <el-popover ref="image" placement="bottom-start" width="235" trigger="click">
                <div style="padding:10px" class="image">
                  <el-upload :action="uploadUrl" :show-file-list="false" :on-success="handleAvatarSuccess"
                             :before-upload="beforeAvatarUpload" drag>
                    <img v-if="imageUrl" :src="imageUrl?imageUrl:imgPath" class="avatar" :onerror="img404">
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                    <span style="display:inline-block;font-size:12px;color:#5F6165;margin:5px 0">请选择人脸照片。支持：JPG、JPEG、PNG格式，且文件大小不超过2M。</span>
                  </el-upload>
                  <el-button type="primary" size="medium" @click="searchImage()" style="width: 100%">搜索</el-button>
                </div>
              </el-popover>
              <el-button icon="el-icon-picture" v-popover:image v-show="getButtonVial('home:allSearch')"></el-button>
              <el-input placeholder="请输入IMSI" :maxlength="15" style="width: 190px" v-popover:imsi clearable
                        v-model="imsi" @keyup.13.native="searchImsi()" v-show="getButtonVial('home:allSearch')">
                <el-button slot="append" icon="el-icon-search" @click.stop="searchImsi()"></el-button>
              </el-input>
            </div>
            <!--<div class="item" style="text-align: center" @click="runMsg = true">-->
            <!--<i class="fa fa-bell-o fa-2x" style="padding-top: 20px;font-size: 1.8em"></i>-->
            <!--</div>-->
            <el-popover ref="modifyPsw" placement="bottom-start" width="180" trigger="click">
              <el-col :span="24">
                <el-button type="text" @click="runModifyPsw = true;psw={}"
                           style="width: 100%;border-radius: 0 0 0 4px;border: none;height: 40px">
                  <i class="fa fa-edit" style="margin-right: 5px;font-size: 1.1em"></i>
                  修改密码
                </el-button>
              </el-col>
              <el-col :span="24">
                <el-button type="text" @click="goLoginDialog()"
                           style="width: 100%;border-radius: 0 0 0 4px;border: none;height: 40px">
                  <i class="fa fa-sign-out" style="margin-right: 5px;font-size: 1.2em"></i>
                  退出登录
                </el-button>
              </el-col>
            </el-popover>
            <el-button class="item" style="text-align: center;width: auto" v-popover:modifyPsw type="text">
              <i class="fa fa-user" style="display: inline-block;padding-top: 0;font-size: 2.3em"></i>
              <span style="display: inline-block;padding-left: 5px">{{userName}}</span>
            </el-button>
            <!--<div class="item" style="text-align: center" @click="$router.push({path: '/platforms'})">-->
            <!--<i class="fa fa-retweet fa-2x" style="padding-top: 13px"></i>-->
            <!--</div>-->
            <!--<div class="item" style="text-align: center" @click="loginOut()">-->
            <!--<i class="fa fa-sign-out fa-2x" style="padding-top: 15px"></i>-->
            <!--</div>-->
          </div>
        </div>
      </el-header>
      <el-main
        v-bind:style="{width:'100%',margin:'0',background:$route.path!=='dataOverview'?'#E9ECF0':'#040d2e',padding:$route.path!=='dataOverview'?'10px 20px 20px 20px':'0',overflowY:$route.path!=='dataOverview'?'auto':'hidden'}">
        <div style="margin: 0;padding: 0">
          <div align="left" style="text-align: left;margin: 0;padding: 0" v-show="$route.path!=='dataOverview'">
            <span style="color: #999;font-size: 14px;vertical-align: middle;">当前位置：</span>
            <el-breadcrumb separator-class="el-icon-arrow-right" class="bread-bar">
              <el-breadcrumb-item v-for="item in $route.matched" v-if="item.name && item.path"
                                  :key="item.name" :to="{ path: item.path}">{{ item.name}}
              </el-breadcrumb-item>
            </el-breadcrumb>
          </div>
          <div
            v-bind:style="$route.path!=='dataOverview'?'background:#ffffff;margin:10px 0 0 0;padding:10px':'margin:10px 0 0 0'">
            <transition name="fade" mode="out-in">
              <router-view @refreshData="refreshData" v-bind:faceCount="faceCount"
                           v-bind:imsiCount="imsiCount" ref="mychild"></router-view>
            </transition>
          </div>
        </div>
      </el-main>
    </el-container>
    <!--告警通知-->
    <div class="view-msg">
      <transition name="fade" mode="out-in" appear>
        <el-dialog fullscreen width="430px" :visible.sync="runMsg" :show-close="false">
          <div class="block">
            <div class="msg-title">
              <span>告警通知</span>
              <el-button class="btn" round plain size="small" @click="clearAll()">清空</el-button>
            </div>
            <div class="msg-item" v-for="item in [1,1,1,1,2]" @click="deleteMsg()">
              <span>嫌疑告警  08:23</span>
            </div>
          </div>
        </el-dialog>
      </transition>
    </div>
    <audio id='audio' :src='warning'></audio>
    <!--IMSI告警 :close-on-click-modal="false"-->
    <div class="warning">
      <transition name="fade" mode="out-in" appear>
        <el-dialog width="500px" :visible.sync="runImsiWarning" style="border-radius: 6px" top="60px" title="嫌疑告警">
          <el-form :model="imsiWarning" align="left" style="padding: 10px 50px;border-top: 1px #efefef solid"
                   label-width="100px" label-position="left">
            <el-form-item label="IMSI" style="margin:0" align="left">
              <div>{{imsiWarning.imsi?imsiWarning.imsi:'--'}}</div>
            </el-form-item>
            <el-form-item label="运营商" style="margin:0" align="left">
              <div>
                {{imsiWarning.isp == 0 ? '移动' : imsiWarning.isp == 1 ? '联通' : imsiWarning.isp == 2 ? '电信' : '--'}}
              </div>
            </el-form-item>
            <el-form-item label="告警时间" style="margin:0" align="left">
              <div>{{imsiWarning.timeStr ? imsiWarning.timeStr : '--'}}</div>
            </el-form-item>
            <el-form-item label="告警场所" style="margin:0" align="left">
              <div>{{imsiWarning.placeName?imsiWarning.placeName:'--'}}</div>
            </el-form-item>
            <el-form-item label="设备标识" style="margin:0" align="left">
              <div>{{imsiWarning.deviceName?imsiWarning.deviceName:'--'}}</div>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer" align="center" v-show="getButtonVial('warning:getImsiWarning')">
            <el-button type="primary" size="medium" @click="gotoImsiDetail()">查看人员信息</el-button>
          </div>
        </el-dialog>
      </transition>
    </div>
    <!--头像告警 :close-on-click-modal="false"-->
    <div class="warning">
      <transition name="fade" mode="out-in" appear>
        <el-dialog width="500px" :visible.sync="runFaceWarning" style="border-radius: 6px" top="60px" title="嫌疑告警">
          <el-row style="padding: 20px 20px 10px;text-align: left;border-top: 1px #efefef solid">
            <el-col :span="11">
              <img :src="faceWarning.faceUrl?faceWarning.faceUrl:imgPath" :onerror="img404">
            </el-col>
            <el-col :span="13">
              <el-form :model="faceWarning" align="left" label-width="80px" label-position="left">
                <el-form-item label="年龄段" style="margin:0" align="left">
                  <div>
                    {{(faceWarning.age==undefined||faceWarning.age<0)?'--':(faceWarning.age-3)+"~"+(faceWarning.age+3)}}
                  </div>
                </el-form-item>
                <el-form-item label="性别" style="margin:0" align="left">
                  <div>{{faceWarning.sex == 0 ? '男' : faceWarning.sex == 1 ? '女' : '--'}}</div>
                </el-form-item>
                <el-form-item label="告警时间" style="margin:0" align="left">
                  <div>{{faceWarning.timeStr ? faceWarning.timeStr : '--'}}</div>
                </el-form-item>
                <el-form-item label="告警场所" style="margin:0" align="left">
                  <div>{{faceWarning.placeName?faceWarning.placeName:'--'}}</div>
                </el-form-item>
                <el-form-item label="设备标识" style="margin:0" align="left">
                  <div>{{faceWarning.deviceName?faceWarning.deviceName:'--'}}</div>
                </el-form-item>
              </el-form>
            </el-col>
          </el-row>
          <div slot="footer" class="dialog-footer" align="center" v-show="getButtonVial('warning:getFaceWarning')">
            <el-button type="primary" size="medium" @click="gotoFaceDetail()">查看人员信息</el-button>
          </div>
        </el-dialog>
      </transition>
    </div>
    <!--修改密码-->
    <el-dialog title="修改密码" width="500px" :visible.sync="runModifyPsw">
      <div class="block">
        <el-form label-width="100px" :rules="rules" ref="psw" :model="psw">
          <el-form-item label="当前密码" prop="password">
            <el-input type="password" :maxlength="16" :minlength="6" placeholder="输入密码"
                      v-model="psw.password"></el-input>
          </el-form-item>
          <el-form-item label="新密码" prop="password1">
            <el-input type="password" v-model="psw.password1"
                      placeholder="请输入6-16位密码" :maxlength="16" :minlength="6"></el-input>
          </el-form-item>
          <el-form-item label="密码确认" prop="password2">
            <el-input type="password" v-model="psw.password2"
                      placeholder="请再次输入新密码" :maxlength="16" :minlength="6"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer" align="center">
          <el-button @click="runModifyPsw = false">取消</el-button>
          <el-button type="primary" @click="modifyPsw('psw')">确定</el-button>
        </div>
      </div>
    </el-dialog>
    <!--搜索弹框全局-->
    <div class="search">
      <el-dialog title="" :visible.sync="runSearch" fullscreen style="margin-top: 60px" :modal="false">
        <searchAll v-bind:searchParam="searchParam" ref="search"></searchAll>
      </el-dialog>
    </div>
  </div>
</template>

<script>
  import md5 from 'js-md5';
  import searchAll from './SearchAll.vue'
  import {numValid, pswValidator, globalValidImg} from '../assets/js/api';
  import {formatDate, buttonValidator, encryData, decryData} from "../assets/js/util";

  export default {
    data() {
      let pswValidate = (rule, value, callback) => {
        if (value.length < 6) {
          callback(new Error('密码不能小于6位'));
        } else if (value.length > 16) {
          callback(new Error('密码不能大于16位'));
        } else if (!pswValidator(value)) {
          callback(new Error("密码由英文字母、数字以及~!@#$%^&*=+/-组成"));
        } else {
          callback();
        }
      };
      return {
        imsiCount: 0, faceCount: 0, turnCount: 0,
        runSearch: false,
        imageUrl: '',
        imsi: '',
        searchImsis: [],
        searchParam: {},
        uploadUrl: this.axios.defaults.baseURL + 'file/upload',
        systemParam: {sysLogo: '../assets/img/icon_logo.svg'},
        runMsg: false,
        runImsiWarning: false,
        runFaceWarning: false,
        runModifyPsw: false,
        warning: require('../assets/warning.mp3'),
        imgPath: require('../assets/img/icon_people.png'),
        img404: "this.onerror='';this.src='" + require('../assets/img/icon_people.png') + "'",
        userName: JSON.parse(decryData(sessionStorage.getItem("user"))).realName || '',
        userId: JSON.parse(decryData(sessionStorage.getItem("user"))).userId,
        psw: {password: '', password1: '', password2: ''},
        imsiWarning: {id: ''},
        faceWarning: {id: ''},
        menu: [],
        intervalid: null,
        loginOutIntervalid: null,
        audio: null,
        rules: {
          password1: [
            {required: true, message: '请输入新密码', trigger: 'blur'},
            {validator: pswValidate, trigger: "change,blur"}
          ], password2: [
            {required: true, message: '请再次输入新密码', trigger: 'blur'},
            {validator: pswValidate, trigger: "change,blur"}
          ], password: [
            {required: true, message: '请输入密码', trigger: 'blur'},
          ]
        }
      }
    },
    components: {searchAll},
    //页面关闭时停止更新设备在线状态
    beforeDestroy() {
      this.audio = null;
      clearInterval(this.intervalid);
      clearInterval(this.loginOutIntervalid);
    },
    methods: {
      getButtonVial(msg) {
        return buttonValidator(msg);
      },
      //以图搜图
      searchImage() {
        if (this.imageUrl.length == 0) {
          this.$message.error("请上传头像");
          return;
        }
        this.searchParam = JSON.stringify({type: 'img', value: this.imageUrl, time: new Date().getTime()});
        this.runSearch = true;
      },
      //以码搜图
      searchImsi() {
        if (this.imsi.length == 0) {
          this.$message.error("请输入IMSI");
          return;
        }
        if (!numValid(this.imsi) || this.imsi.length != 15) {
          this.$message.error('请输入15位正确的IMSI');
          return;
        }
        if (this.isMultiple(this.imsi)) {
          this.searchImsis.unshift(this.imsi);
          localStorage.setItem("imsis", JSON.stringify({imsi: this.searchImsis}));
        }
        this.searchParam = JSON.stringify({type: 'imsi', value: this.imsi, time: new Date().getTime()});
        this.runSearch = true;
      },
      //是否重复
      isMultiple(val) {
        let bol = true;
        if (this.searchImsis.length > 0) {
          this.searchImsis.forEach((item) => {
            if (val == item) {
              bol = false;
            }
          });
        }
        return bol;
      },
      //清空搜索IMSI记录
      clearImsi() {
        localStorage.removeItem('imsis');
        this.searchImsis = [];
      },
      handleAvatarSuccess(res, file) {
        if (res.code === '000000') {
          if (res.data) {
            this.imageUrl = res.data.fileUrl;
          }
        } else {
          this.$message.error(res.msg);
        }
      },
      beforeAvatarUpload(file) {
        if (globalValidImg(file, this.$message)) {
        }
        return globalValidImg(file, this.$message);
      },
      //IMSI/人脸告警-->10s请求一次
      statusTask() {
        if (!this.intervalid) {
          this.intervalid = setInterval(() => {
            this.getImsiWarning();
            this.getFaceWarning();
            this.getWarningCount();
            this.getTurnCount();
          }, 10 * 1000);
        }
      },
      //页面变化-->导航栏选中状态变化
      refreshData(sys, val) {
        if (sys == 'sys') {//系统参数变动
          this.systemParam = JSON.parse(decryData(sessionStorage.getItem("system")));
        } else if (sys == 'warning') {//告警数量变动
          this.getWarningCount();
        } else if (sys == 'menu') {//权限变动
          this.getButton();
        } else if (sys == 'turn') {//翻码审批中的数量
          this.getTurnCount();
        }
      },
      //上方导航栏点击切换页面
      handleSelectItem(item) {
        sessionStorage.removeItem("query");
        sessionStorage.removeItem("qTime");
        sessionStorage.removeItem("page");
        sessionStorage.removeItem("activeItem");
        sessionStorage.removeItem("secItem");
        localStorage.removeItem("pathTime");//轨迹
        localStorage.removeItem("pathImsi");
        localStorage.removeItem("pathFace");
        localStorage.removeItem("pathUrl");
        sessionStorage.removeItem('apply');
      },
      getImsiWarning() {
        if (this.getButtonVial('warning:get:listImsiToday')) {
          this.imsiWarning = JSON.parse(sessionStorage.getItem("imsi"));
          this.$post("warning/get/listImsiToday", {size: 1}).then((data) => {
            if (data.data && data.data.length > 0) {
              let imsi = data.data[0];
              // console.log(new Date().getTime() - imsi.createTime * 1000);
              if ((new Date().getTime() - imsi.createTime * 1000) >= -60 * 3 * 1000 && (new Date().getTime() - imsi.createTime * 1000) <= 60 * 3 * 1000) {//30s内的数据
                // console.log(this.imsiWarning.id);
                if (this.imsiWarning.id !== imsi.id) {
                  // console.log(imsi.id);
                  sessionStorage.setItem("imsi", JSON.stringify(imsi));
                  this.audio.play();
                  this.imsiWarning = imsi;
                  this.imsiWarning.timeStr = formatDate(new Date(this.imsiWarning.createTime * 1000), 'yyyy-MM-dd hh:mm:ss');
                  this.runFaceWarning = false;
                  this.runImsiWarning = true;
                  setTimeout(() => {
                    this.runImsiWarning = false;
                  }, 5000);
                }
              }
            }
          });
        }
      },
      getFaceWarning() {
        if (this.getButtonVial('warning:get:listFaceToday')) {
          this.faceWarning = JSON.parse(sessionStorage.getItem("face"));
          this.$post("warning/get/listFaceToday", {size: 1}).then((data) => {
            if (data.data && data.data.length > 0) {
              let face = data.data[0];
              // console.log(new Date().getTime() - face.createTime * 1000);
              if ((new Date().getTime() - face.createTime * 1000) >= -60 * 3 * 1000 && (new Date().getTime() - face.createTime * 1000) <= 60 * 3 * 1000) {//30s内的数据
                // console.log(this.faceWarning.id);
                if (this.faceWarning.id !== face.id) {
                  // console.log(face.id);
                  sessionStorage.setItem("face", JSON.stringify(face));
                  this.audio.play();
                  this.faceWarning = face;
                  this.faceWarning.timeStr = formatDate(new Date(this.faceWarning.createTime * 1000), 'yyyy-MM-dd hh:mm:ss');
                  this.runImsiWarning = false;
                  this.runFaceWarning = true;
                  setTimeout(() => {
                    this.runFaceWarning = false;
                  }, 5000);
                }
              }
            }
          });
        }
      },
      gotoImsiDetail() {
        if (this.getButtonVial('warning:getImsiWarning')) {
          this.runImsiWarning = false;
          this.$router.push({
            path: '/imsiWarningDetail',
            query: {id: this.imsiWarning.id, imsi: this.imsiWarning.imsi, taskId: this.imsiWarning.dispositionTaskId}
          });
        }
      },
      gotoFaceDetail() {
        if (this.getButtonVial('warning:getFaceWarning')) {
          this.runFaceWarning = false;
          this.$router.push({
            path: '/faceWarningDetail',
            query: {
              id: this.faceWarning.id, faceId: this.faceWarning.faceId,
              taskId: this.faceWarning.dispositionTaskId
            }
          });
        }
      },
      //告警数量
      getWarningCount() {
        if (this.getButtonVial('warning:countNoDealWithImsiWarning')) {
          this.$post('/warning/countNoDealWithImsiWarning', {}).then((data) => {
            if ("000000" === data.code) {
              this.imsiCount = data.data;
              this.$nextTick(() => {
                this.$refs.mychild.imsi = this.imsiCount;
                this.$refs.mychild.face = this.faceCount;
              });
            }
          }).catch((err) => {
          });
        }
        if (this.getButtonVial('warning:countNoDealWithFaceWarning')) {
          this.$post('/warning/countNoDealWithFaceWarning', {}).then((data) => {
            if ("000000" === data.code) {
              this.faceCount = data.data;
              this.$nextTick(() => {
                this.$refs.mychild.imsi = this.imsiCount;
                this.$refs.mychild.face = this.faceCount;
              });
            }
          }).catch((err) => {
          });
        }
      },
      /*获取翻码待审批数量*/
      getTurnCount() {
        if (this.getButtonVial('workflow:translation:myapprove')) {
          this.$post('/workflow/translation/myapprove/' + this.userId,
            {page: 1, size: 10, approveStatus: 1}).then((data) => {
            if ("000000" === data.code) {
              this.turnCount = data.data.count;
            }
          }).catch((err) => {
            this.turnCount = 0;
          });
        }
      },
      //退出
      goLoginDialog() {
        this.$confirm('确认退出系统吗?', '提示', {type: 'info'}).then(() => {
          this.loginOut();
        }).catch(() => {
        });
      },
      loginOut() {
        this.$post('/manager/user/logout', {}, '退出成功').then((data) => {
          if ('000000' == data.code) {
            sessionStorage.removeItem("user");
            sessionStorage.removeItem("button");
            sessionStorage.removeItem("menu");
            sessionStorage.removeItem("query");
            sessionStorage.removeItem("qTime");
            sessionStorage.removeItem("activeItem");
            sessionStorage.removeItem("index");
            sessionStorage.removeItem("face");
            sessionStorage.removeItem("imsi");
            sessionStorage.removeItem("search");
            sessionStorage.removeItem('apply');
            localStorage.removeItem("login");
            localStorage.removeItem("clickTime");
            this.$router.push("/login");
          }
        });
      },
      //页面30分钟不操作退出
      pageEvent() {
        if (this.$route.path != '/login') {//登录页面不执行
          localStorage.setItem("clickTime", new Date().getTime());
          this.timeOut();
        }
      },
      timeOut() {
        if (!this.loginOutIntervalid) {
          this.loginOutIntervalid = setInterval(() => {
            let clickTime = localStorage.getItem("clickTime");
            let subTime = new Date().getTime() - clickTime;
            if (subTime > 30 * 60 * 1000) {//超过30分钟退出
              clearInterval(this.loginOutIntervalid);
              this.loginOut();
            }
          }, 10 * 1000);
        }
      },
      //修改密码
      modifyPsw(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            if (this.psw.password1 !== this.psw.password2) {
              this.$message.error('2次密码输入不一致，请重新输入');
              return;
            }
            this.$post('/manager/user/updatePwdByOldPwd', {
              userId: this.userId, oldPwd: md5(this.psw.password), newPwd: md5(this.psw.password2)
            }, '修改成功').then((data) => {
              if (data.code === '000000') {
                var bol = JSON.parse(localStorage.getItem("user"));
                bol.psw = this.psw.password2;
                localStorage.setItem("user", JSON.stringify(bol));
              }
            });
            this.runModifyPsw = false;
          }
        });
      },
      /**删除告警通知*/
      deleteMsg() {
        this.$confirm('确定要删除该条告警通知?', '提示', {type: 'info'}).then(() => {
        }).catch(() => {
        });
      },
      /**
       * 清空告警通知
       */
      clearAll() {
        this.$confirm('确定要清除所有的告警通知?', '提示', {type: 'info'}).then(() => {
        }).catch(() => {
        });
      },
      //获取用户按钮权限
      getButton() {
        //菜单类型(1:目录,2:菜单,3:按钮)
        this.$post('/manager/permission/listByType/' + this.userId + '/3', {}).then((data) => {
          sessionStorage.setItem("button", JSON.stringify(data.data));
        });
      },
      //获取系统参数配置
      getSystemDetail() {
        this.$post('sysparam/query', {}).then((data) => {
          if ("000000" === data.code) {
            if (data.data.length > 0) {
              data.data.forEach((item) => {
                if (item.code == 'sys_name') {
                  this.systemParam.sysName = item.value;
                }
                if (item.code == 'sys_logo') {
                  this.systemParam.sysLogo = item.value;
                }
                if (item.code == 'sys_map_location') {
                  this.systemParam.localPoint = item.value;
                }
                if (item.code == 'heatChart_refresh_freq') {
                  this.systemParam.refreshTime = item.value;
                }
                if (item.code == 'heatChart_time_limit') {
                  this.systemParam.limitTime = item.value;
                }
                if (item.code == 'heatChart_color_range') {
                  this.systemParam.heatRanges = item.value;
                }
                if (item.code == 'image_search_threshold') {
                  this.systemParam.similarThreshold = item.value;
                }
              });
              this.$nextTick(() => {
                this.$refs.mychild.systemParam = this.systemParam;
              });
              sessionStorage.setItem("system", encryData(JSON.stringify(this.systemParam)));
            }
          }
        }).catch((err) => {
          this.$message.error(err);
        });
      }
    },
    mounted() {
      this.getButton();
      this.getSystemDetail();

      this.searchImsis = localStorage.getItem("imsis") ? JSON.parse(localStorage.getItem("imsis")).imsi : [];
      this.menu = JSON.parse(decryData(sessionStorage.getItem("menu"))) || [];
      this.audio = document.getElementById('audio');

      this.getImsiWarning();
      this.getFaceWarning();
      this.getWarningCount();
      this.getTurnCount();
      this.statusTask();
      /*账号在15分钟或者30分钟无人使用的情况下应自动退出；*/
      document.body.addEventListener("click", this.pageEvent);
      document.body.addEventListener("keydown", this.pageEvent);
      document.body.addEventListener("mousemove", this.pageEvent);
      document.body.addEventListener("mousewheel", this.pageEvent);
    }
  }
</script>
<style scoped>
  .warning img {
    max-height: 190px;
    max-width: 190px;
    display: inline-block;
    border-radius: 6px;
  }

  .warning div {
    font-size: 15px;
    color: #000;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis
  }

  .msg-title {
    height: 60px;
    border-bottom: 1px #ddd solid;
    margin-bottom: 15px;
    background: #fff;
  }

  .msg-title span {
    height: 60px;
    line-height: 60px;
    font-size: 18px;
    color: #000;
  }

  .msg-title .btn {
    position: absolute;
    height: 30px;
    top: 15px;
    right: 20px;
    padding: 0 20px;
  }

  .msg-item {
    background: #fff;
    border-radius: 4px;
    padding: 20px 30px;
    margin: 0 15px 15px 15px;
  }

  .main-container {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    min-width: 1366px;
    margin-left: -8px;
  }

  .item {
    display: inline-block;
    cursor: pointer;
    width: 80px;
    height: 60px;
    padding: 0;
    margin: 0;
    color: #fff;
    background: transparent;
    font-size: 14px;
    border: none;
    border-radius: 0;
  }

  .item * {
    vertical-align: middle;
  }

  .item.active {
    background: #46467C;
  }

  .item:hover, .item:active {
    background: #181663;
  }

  .item .title {
    display: block;
  }

  .item i {
    display: block;
    margin-top: 3px;
  }

  .avatar {
    max-width: 130px;
    max-height: 130px;
    border: 1px dashed #ccc;
    border-radius: 6px;
  }

  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 130px;
    height: 130px;
    border: 1px dashed #ccc;
    border-radius: 6px;
    line-height: 130px;
    text-align: center;
  }
</style>
