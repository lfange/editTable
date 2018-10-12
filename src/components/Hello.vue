<template>
  <div class="index" v-loading="indexloading">
    <el-button :plain="true" @click="alertmsg" v-if="false">消息</el-button>
    <el-container style="width:100%;height:100%;box-sizing:border-box;padding:7px;">
      <el-header>
        <el-row>
          <el-col :span="18">
            <el-menu :default-active="defaultTypeId" class="el-menu-demo" mode="horizontal" @select="handleSelect" background-color="#545c64" text-color="#fff" active-text-color="#ffd04b">
              <el-menu-item index="10"><el-button type="primary">瞬时数据</el-button></el-menu-item>
              <el-menu-item index="11"><el-button type="primary">当日数据</el-button></el-menu-item>
              <el-menu-item index="20"><el-button type="primary">其他数据</el-button></el-menu-item>
            </el-menu>
          </el-col>
          <el-col :span="6" style="height:60px;background-color:#545c64">
            <el-button type="primary" icon="el-icon-search" style="float:right;margin:10px 40px 0 0" @click="GetGridData()">查询</el-button>
            <el-button type="primary" style="float:right;margin:10px 20px 0 0" @click="Showdialog">导入</el-button>
          </el-col>
        </el-row>
      </el-header>
      <div class="tablebox">
        <h3>
          <span class="left" :class="{wid: onlyoneunit}"> <span class="circle"></span> 当前：{{CompanyName}}{{TypeName}}
            <div>
              <span style="margin-left:30px;height:35px;line-height:35px;font-size:14px">
                  图例
                <span class="overs"></span>录入数据超过设定上限
                <span class="unders"></span>录入数据低于设定下限
              </span>
            </div>
          </span>
          <span class="right" v-if=" !onlyoneunit">
            <i class="el-icon-arrow-left" @click="allscreen = !allscreen" v-if="allscreen" style="float:right;font-size:22px;padding:8px 20px 0 0"></i>
            <i class="el-icon-arrow-right" @click="allscreen = !allscreen" v-else style="float:right;font-size:22px;padding:8px 20px 0 0"></i>
            <span v-show="!allscreen">单位导航</span>
          </span>
        </h3>
        <div style="width:100%;height:calc(100% - 30px)">
          <div class="main" :class="{main2:allscreen}">
            <div class="items" style="border-bottom:1px solid #e5e5e5">
              <span class="textmargin">数据类型:</span>
              <span class="textmargin">
                  <el-radio-group v-model="DateType"  @change="checkedDateType">
                    <el-radio v-for="i in radioType" :label="i.TYPE_ID" :key="i.TYPE_ORDER">{{i.TYPE_NAME}}</el-radio>
                  </el-radio-group>
                </span>
              <el-select v-loading="reportLoading" v-model="ReportId" placeholder="请选择" @change="choiceunit" v-if="ontime" style="width:176px" :selectloading="true">
                <el-option v-for="item in ReportList" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
            </div>
            
            <!-- 瞬时数据 -->
            <el-table id="table" v-loading="tableloading" :data="tableData3" 
                style="color:black" height="calc(100% - 70px)" border :row-class-name="rowColor"  @cell-click="celledit" 
                :fit="false" :highlight-current-row="false" resizable  v-if="checkedTypeid=='10'">
              <el-table-column v-for="(item,key1) in tablehead" :key="key1"
               :label="item.lable" v-if="key1!=0" header-align="center" :fixed="key1==1" :render-header="labelHead">
                <el-table-column v-for="(value,key) in item.children" :prop="key" :key="key"
                   :label-class-name="key" :label="value"  :align="key1==1?'center':''" min-width="100">
                   </el-table-column>
              </el-table-column>
            </el-table>
            <!-- 当日数据table渲染 -->
            <el-table id="table"  v-loading="tableloading" :data="tableData3" 
                style="color:black" height="calc(100% - 70px)"  border 
                :fit="false" @cell-click="celledit"  :row-class-name="rowColor" resizable v-else>
              <el-table-column v-for="(i,key,index) in tablehead.children" header-align="center" 
               :align="index==1?'center':''" :key="key" :label-class-name="key" 
              :prop="key" :label="i" v-if="index!=0" :fixed="index==1" :width="index==1?tablehead.width:''" :render-header="labelHead" min-width="100"></el-table-column>
            </el-table>
            <div class="items">
              <div class="block inlinediv textmargin">
                <span class="demonstration">时间</span>
                <el-date-picker v-model="date1" @change="changtime" type="date" format="yyyy-MM-dd" value-format="yyyy-MM-dd" placeholder="选择日期">
                </el-date-picker>
              </div>
              <span v-if="ontime" style="margin-left:10px"> 时间间隔：</span>
              <el-select v-model="timeInterval" placeholder="请选择" v-if="ontime" @change="choicetime">
                <el-option v-for="item in timespan" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
              <span class="errmsg textmargin" v-if="errmsg!=''">{{errmsg}}</span>
            </div>
          </div>
          <div class="nav"  v-show="!allscreen" >
            <el-aside width="100%" style="height:100%;">
              <el-tree :data="menu" :props="defaultmenu" @node-click="handleNodeClick"></el-tree>
            </el-aside>
          </div>
        </div>
      </div>
    </el-container>
    <!-- 导入文件弹出框 -->
    <el-dialog title="导入文件" :visible.sync="dialogvisible" >
      <el-row style="color:red">
        <el-col :span="4" >提&nbsp;&nbsp;&nbsp;&nbsp;示：</el-col>
        <el-col :span="20">
          请下载要录入单位的模板，完成数据填写后，再进行导入。 <br>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="4">导出模板</el-col>
        <el-col :span="20">
          <el-button type="text" @click="download" size="medium">{{CompanyName}}{{date1}}{{DataTypeName}}{{TypeName}}模板下载</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="4">导入文件</el-col>
        <el-col :span="20">
          <el-button type="primary" style="position: relative;overflow:hidden" size="small" @click="clickBtn">
            选择文件<input type="file" @change="selectfile" id="fileup" style="position:absolute;display:none;left:0;top:0;width:100%;height:100%;"></el-button>
          <span v-if="dialogform.fileName!=''">{{dialogform.fileName}}</span>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24"><span style="color:red" v-if="dialogform.err!=''">{{dialogform.err}}</span></el-col>
      </el-row>
      <el-row>
        <el-col :span="6" :offset="18" style="margin-top:20px">
          <el-button type="primary" size="medium" @click="uploading" :class="{isactive:dialogform.fileName==''}">导入</el-button>
          <el-button type="primary" size="medium" @click="dialogvisible = false">取消</el-button>
        </el-col>
      </el-row>
          <!-- 上传遮罩层 -->
      <el-dialog 
        width="30%"
        title="正在导入" 
        :visible.sync="innerVisible" :close-on-click-modal="false" :show-close="false"
        append-to-body>
        <i class="el-icon-loading"></i>请稍后...
      </el-dialog>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    label: "HelloWorld",
    data() {
      return {
        dialogform: {
          fileName: "",
          err: "请上传xml文件，"
        },
        progress: "",
        defaultTypeId: "10",
        checkedTypeid: "10",
        count: 0,
        num: 0,
        DateType: "",
        onlyoneunit: false,
        radioType: [],
        company: "中24井集配气站",
        TypeName: "瞬时数据",
        date1: new Date(),
        allscreen: false,
        selectloading: true,
        CompanyName: "",
        DataTypeName: "",
        info: {
          //个人信息
          userid: "",
          token: "",
          key: "",
          CompanyCode: "",
          KeyId: "",
          Dwdm: ""
        }, //http://192.168.7.250/GridInput/GridInput/
        url_s: "GridInput.aspx",
        indexloading: false, //整个页面加载显示
        tableloading: false, //table
        reportLoading: false, //获取下拉列表reportLoading
        dialogvisible: false,
        innerVisible: false,
        waitsubmit:"",
        ReportId: "",
        ontime: true, //控制瞬时数据显示与否
        timeInterval: "", //时间间隔选择
        errmsg: "",
        tableData3: [],
        tablehead: "",
        sourcehead:[],
        ReportList: [], //数据类型的下拉框
        timespan: [], //时间间隔
        menus: "",
        menu: [],
        defaultmenu: {
          children: "children",
          label: "label"
        }
      };
    },
    created() {
      this.GetUserInfo();
    },
    methods: {
      //导航栏选择
      handleSelect(key, keyPath) {
        this.checkedTypeid = key;
        this.GetDataType();
        this.errmsg = "";
        this.tableData3 = [];
        this.tablehead = [];
        if (key === "10") {
          this.TypeName = "瞬时数据";
          this.ontime = true;
          this.GetReportList();
        } else if (key === "11") {
          this.TypeName = "当日数据";
          this.ontime = false;
        } else {
          this.TypeName = "其他数据";
          this.ontime = false;
        }
      },
      checkedDateType(checkedDateType) {
        let self = this;
        self.GetReportList();
        self.GetGridData(); //
        this.radioType.forEach(function(item) {
          if (item.TYPE_ID == checkedDateType) {
            self.DataTypeName = item.TYPE_NAME;
          }
        });
      },
      //选择具体时间
      changtime(time) {
        this.GetGridData();
      },
      //上面选择
      choiceunit(es) {
        this.GetGridData(); //
      },
      //时间间隔选择
      choicetime(es) {
        this.GetGridData();
        this.SaveTimeIntervel(es)
      },
      handleNodeClick(data) {
        this.info.CompanyCode = data.DWDM;
        this.CompanyName = data.DWJC;
        this.GetDataType();
        this.GetReportList();
        this.tableData3 = [];
        this.tablehead = [];
      },
      adInput(cell){
        // 获取选定对象
        if(cell.children.length>1) return
        var cellInput = document.createElement("input");
        cellInput.setAttribute("type", "text");
        cellInput.setAttribute("class", "edit");
        cellInput.value = cell.children[0].innerText;
        cellInput.style.width = "100%";
        cellInput.style.height = "100%"
        cellInput.style.border = "none";
        cellInput.style.backgroundColor = "transparent";
        cellInput.style.paddingLeft = "10px";
        cellInput.style.paddingRight = "4px";
        cellInput.style.fontSize = "15px";
        cellInput.style.outline = "none";
        cellInput.style.color = "black";
        cellInput.style.fontFamily = "Microsoft YaHei" ;
        cellInput.style.boxSizing = "content-box"
        cell.children[0].style.display = " none";
        cell.appendChild(cellInput);
        cellInput.focus();　　　　//主动聚焦
        cellInput.select()
        this.errmsg = ""
      },
      //点击单元格编辑
      celledit(row, column, cell, event) {
        let self = this;
        if(this.checkedTypeid != '10'){
          if (column.label == "项目") return;
        }else{
          if (column.label == "时间") return;
        }
        this.adInput(cell)
        self.onkey(row, column, cell, event, true);
      },
      onkey(row, column, cell, event, boolen) {
        let self = this; 
        var thead = document.getElementsByTagName("thead");
        var curel = document.activeElement; //当前元素
        // oldtext = curel.textContent;
        var curcellIndex = curel.parentNode.cellIndex; // 当前元素行单元格索引
        var curRowIndex = curel.parentNode.parentElement.sectionRowIndex; //当前元素行的索引；
        var curtbody = curel.parentNode.parentElement.parentElement.children; //当前tbody内容的整个表单
        if(self.checkedTypeid != '10'){
          var ColumnCode = thead[0].children[0].children[curcellIndex].classList[2];
        }else{
          var ColumnCode = thead[0].children[1].children[curcellIndex].classList[1];
        }
        curel.onblur = function() {
          let cellInput=  document.getElementsByClassName("edit")
          let parent = this.parentNode;
          parent.children[0].style.display = "block";
          parent.children[0].innerText = this.value
          if(parent.textContent != '' ){
            self.SaveGridData(curRowIndex, ColumnCode, parent.textContent, curel.parentElement);
          }
          parent.removeChild(this);
        };
        document.onkeydown = function(e) {
          var e = e || window.event;
          var thead = document.getElementsByTagName("thead");
          var curel = document.activeElement; //当前元素
          var curcellIndex = curel.parentNode.cellIndex; // 当前元素行单元格索引
          var curRowIndex = curel.parentNode.parentElement.sectionRowIndex; //当前元素行的索引；
          var curtbody = curel.parentNode.parentElement.parentElement.children; //当前tbody内容的整个表单
          if(self.checkedTypeid != '10'){
            var ColumnCode = thead[0].children[0].children[curcellIndex].classList[2];
          }else{
            var ColumnCode = thead[0].children[1].children[curcellIndex].classList[1];
          }
          curel.onblur = function() {
            let cellInput=  document.getElementsByClassName("edit")
            let parent = this.parentNode;
            parent.children[0].style.display = "block";
            parent.children[0].innerText = this.value
            if(parent.textContent != '' ){
              self.SaveGridData(curRowIndex, ColumnCode, parent.textContent, curel.parentElement);
            }
            parent.removeChild(this);
          };
          if (e.keyCode == 38) {
            //按上键
            e.preventDefault();
            if (curRowIndex - 1 < 0) {
              self.adInput(curtbody[curtbody.length - 1].children[curcellIndex])
            } else {
              self.adInput(curtbody[curRowIndex - 1].children[ curcellIndex ])
            }
          } else if (e.keyCode == 37) {
            let preCellindex = curtbody[curtbody.length - 1].children.length - 1;
            //键盘按钮左键
            e.preventDefault();
            if (curcellIndex - 1 == 0) {
              if (curRowIndex - 1 < 0) {
                self.adInput( curtbody[curtbody.length - 1].children[preCellindex]);
              } else {
                self.adInput(curtbody[curRowIndex - 1].children[preCellindex]);
              }
            } else {
              self.adInput(curel.parentElement.parentElement.children[curcellIndex - 1]);
            }
          } else if (e.keyCode == 39 || e.keyCode == 9) {
            //键盘按钮右键
            e.preventDefault();
            if (curcellIndex + 1 == curel.parentElement.parentElement.children.length) {
              if (curRowIndex + 1 == curtbody.length) {
                self.adInput(curtbody[0].children[1]);
              } else {
                self.adInput(curtbody[curRowIndex + 1].children[1]);
              }
            } else {
              self.adInput(curel.parentElement.parentElement.children[curcellIndex + 1]);
            }
          } else if (e.keyCode == 40 || e.keyCode == 13) {
            //向下按钮按键
            e.preventDefault();
            if (curRowIndex + 1 == curtbody.length) {
              self.adInput(curtbody[0].children[curcellIndex]);
            } else {
              self.adInput(curtbody[curRowIndex + 1].children[curcellIndex]);
            }
          }
        };
      },
      selectText(curel) {
        if (document.selection) {
          var range = document.body.createTextRange();
          range.moveToElementText(curel);
          range.select();
        } else if (window.getSelection) {
          var range = document.createRange();
          range.selectNodeContents(curel);
          window.getSelection().removeAllRanges();
          window.getSelection().addRange(range);
        }
      },
      alertmsg(msg) {
        this.$message({
          message: msg,
          type: "warning"
        });
      },
      //获取用户信息
      GetUserInfo() {
        this.indexloading = true;
        // 判断权限
        if (this.$route.query.userid === undefined) {
          this.indexloading = false;
          return this.alertmsg("登录过期，请从新登录");
        } else {
          this.info.userid = this.$route.query.userid;
          this.info.token = this.$route.query.token;
          this.info.key = this.$route.query.key;
        }
        this.axios
          .post(this.url_s + "/GetUserInfo", {
            userid: this.$route.query.userid,
            token: this.$route.query.token,
            key: this.$route.query.key
          })
          .then(res => {
            res = JSON.parse(res.data.d);
            if (res.CompanyCode === undefined) {
              this.indexloading = false;
              //window.location.href = "https://www.baidu.com"
            }
            this.info.CompanyCode = res.CompanyCode;
            this.info.KeyId = res.KeyId;
            this.info.Dwdm = res.CompanyModule.Dwdm;
            this.GetTimeIntervel();
            this.getCompanyMd();
          })
          .catch(err => {
            this.alertmsg("网络错误！！");
            this.indexloading = false;
          });
      },
      //获取单位结构
      getCompanyMd() {
        this.axios
          .post(this.url_s + "/GetCompanyMd", {
            companyCode: this.info.CompanyCode
          })
          .then(res => {
            res = JSON.parse(res.data.d);
            if(res.length == 1){
              this.allscreen = true;
              this.onlyoneunit = true;
            }
            this.indexloading = false;
            this.menu = JSON.parse(JSON.stringify(this.toTree(res)));
            this.CompanyName = this.menu[0].DWJC;
            this.GetDataType();
          });
      },
      ///获取单位的默认时间间隔
      GetTimeIntervel() {
        //获取用户列表
        this.axios.post(this.url_s + "/GetTimeIntervel", {}).then(res => {
          res = JSON.parse(res.data.d);
          res = res.map(_ => {
            _.value = _.SJJG;
            _.label = _.SJJG_NAME;
            return _;
          });
          this.timespan = res;
          //获取用户时间间隔默认选择值
          this.axios
            .post(this.url_s + "/GetTimeIntervel_Dwdm", {
              dwdm: this.info.Dwdm
            })
            .then(res => {
              res = JSON.parse(res.data.d);
              this.timeInterval = res[0].SJJG;
            });
        });
      },
      //选择时间间隔后保存时间间隔
      SaveTimeIntervel(es){
        this.axios.post( this.url_s + "/SaveTimeIntervel_Dwdm",{
          dwdm: this.info.Dwdm,
          sjjg:es
        }).then(res=>{
          // console.log("SaveTimeIntervel",res)
        })
      },
      //获取当前单位所有的数据类型   生成radio button使用
      GetDataType() {
        this.axios
          .post(this.url_s + "/GetDataType", {
            CompanyCode: this.info.CompanyCode,
            TypeId: this.checkedTypeid
          })
          .then(res => {
            res = JSON.parse(res.data.d);
            if (res == null) {
              this.radioType = [];
            } else {
              this.radioType = res;
              this.DateType = this.radioType[0].TYPE_ID;
              this.DataTypeName = this.radioType[0].TYPE_NAME;
              this.GetReportList();
              this.GetGridData();
            }
          });
      },
      /// 获取下拉数据对象
      GetReportList() {
        if(this.checkedTypeid != "10") return
        this.reportLoading = true;
        this.ReportId = "";
        this.axios
          .post(this.url_s + "/GetReportList", {
            CompanyCode: this.info.CompanyCode,
            TypeId: this.checkedTypeid,
            DataType: this.DateType
          })
          .then(res => {
            res = JSON.parse(res.data.d);
            res = res.map(_ => {
              _.value = _.REPORTINFO_ID;
              _.label = _.REPORTNAME;
              return _;
            });
            this.reportLoading = false;
            this.ReportList = res;
          });
      },
      toTable(res,obj){
        this.sourcehead = obj
        var arr = [];
        for (var key in obj) {
          var item = {};
          item.children = new Object();
          item.children[key] = obj[key];
          item.id = res[0][key];
          item.lable = res[1][key];
          arr.push(item);
        }
        let map = {}, dest = [];
        for (var i = 0; i < arr.length; i++) {
          var ai = arr[i];
          if (!map[ai.lable]) {
            dest.push({
              id: ai.id,
              lable: ai.lable,
              children: ai.children
            });
            map[ai.lable] = ai.lable;
          } else {
            for (var j = 0; j < dest.length; j++) {
              var dj = dest[j];
              if (dj.lable == ai.lable) {
                for (var o in ai.children) {
                  dj.children[o] = ai.children[o];
                }
                break;
              }
            }
          }
        }
        return JSON.parse(JSON.stringify(dest))
      },
      //获取表格信息
      GetGridData() {
        this.tableData3 = [];
        this.tablehead = [];
        this.errmsg = "";
        if (typeof this.date1 === "object") {
          this.date1 = this.formatDate(this.date1);
        }
        this.tableloading = true;
        if (this.ReportId === "") {     //没有选择下拉选项时候
          this.axios
            .post(this.url_s + "/GetGridData", {
              CompanyCode: this.info.CompanyCode,
              InputDate: this.date1,
              TypeId: this.checkedTypeid,
              DataType: this.DateType,
              DataInterval: this.timeInterval
            })
            .then(res => {
              res = JSON.parse(res.data.d);
              if(this.checkedTypeid != "10"){   
                console.log("res",res)
                let wid = "6";      //默认至少为6个字节，     通过计算数据第一个字段的长度来计算第一行宽度为多少，避免换行
                for(var i=0;i<res.length;i++){
                  if(res[i].COL_NAME.length > wid){
                    wid = res[i].COL_NAME.length
                  }
                }
                console.log("wid",wid)
                this.tablehead.children = new Object()
                this.tablehead.children = res[1];
                this.tablehead.width = wid*16
                console.log("this.tablehead",this.tablehead)
                let arr = JSON.parse(JSON.stringify(res));
                arr.splice(0,2)
                this.tableData3 = arr;
                this.sourcehead = res[1];
              } else{
                this.tablehead =this.toTable(res,res[2])
                res.splice(0, 3);
                this.tableData3 = res;
              }
              this.tableloading = false;
            }).catch(err => {
              this.alertmsg(err.message);
              this.tableloading = false;
            });
        } else {
          this.axios
            .post(this.url_s + "/GetGridData_New", {
              CompanyCode: this.info.CompanyCode,
              InputDate: this.date1,
              TypeId: this.checkedTypeid,
              DataType: this.DateType,
              DataInterval: this.timeInterval,
              ReportId: this.ReportId
            })
            .then(res => {
              res = JSON.parse(res.data.d);
              if(this.checkedTypeid != "10"){  
                 let wid = "6";      //默认至少为6个字节，     通过计算数据第一个字段的长度来计算第一行宽度为多少，避免换行
                for(var i=0;i<res.length;i++){
                  if(res[i].COL_NAME.length > wid){
                    wid = res[i].COL_NAME.length
                  }
                }
                console.log("wid",wid)
                this.tablehead.children = new Object()
                this.tablehead.children = res[1];
                this.tablehead.width = wid*16
                let arr = JSON.parse(JSON.stringify(res));
                arr.splice(0,2)
                this.tableData3 = arr;
                this.sourcehead = res[1];
              } else{
                this.tablehead =this.toTable(res,res[2])
                res.splice(0, 3);
                this.tableData3 = res;
              }
              this.tableloading = false;
            }).catch(err => {
              this.alertmsg(err.message);
              this.tableloading = false;
            });
        }
      },
      SaveGridData(rowCode, Col, Value, cell) {
        console.log("Col",Col)
        rowCode = this.tableData3[rowCode].COL_CODE;
        if (typeof this.date1 === "object") {
          this.date1 = this.formatDate(this.date1);
        }
        rowCode = rowCode.substring(0, rowCode.indexOf("@"));
        Col = Col.substring(0, Col.indexOf("@"));
        this.axios
          .post(this.url_s + "/SaveGridData", {
            TypeId: this.checkedTypeid,
            InputDate: this.date1,
            RowCode: rowCode,
            ColumnCode: Col,
            Value: Value,
            UserId: this.info.CompanyCode
          })
          .then(res => {
            res = res.data.d;
            this.num +=1;
            this.errmsg = res.substring(res.indexOf("&") + 1, res.length);
            if (this.errmsg == "数据小于最小值限定!") {
              cell.classList.add("underline") 
            } else if (this.errmsg == "数据不符合规则,此处需要输入数字!") {
              cell.classList.add("overline") 
            } else if (this.errmsg == "数据超过最大值限定!") {
              cell.classList.add("overline") 
            } else {
              cell.classList.remove("underline") 
              cell.classList.remove("overline") 
            }
            if(this.innerVisible == true && this.num == this.count){     //通过请求次数来关闭导入loading遮罩
              this.innerVisible = false;
              this.dialogvisible = false;
            }
          });
      },
      formatDate(date) {
        var y = date.getFullYear();
        var m = date.getMonth() + 1;
        m = m < 10 ? "0" + m : m;
        var d = date.getDate();
        d = d < 10 ? "0" + d : d;
        var h = date.getHours();
        var minute = date.getMinutes();
        minute = minute < 10 ? "0" + minute : minute;
        var second = date.getSeconds();
        second = minute < 10 ? "0" + second : second;
        return y + "-" + m + "-" + d;
      },
      toTree(data) {
        // 删除 所有 children,以防止多次调用
        data.forEach(function(item) {
          delete item.children;
        });
        // 将数据存储为 以 id 为 KEY 的 map 索引数据列
        var map = {};
        data.forEach(function(item) {
          item.label = item.DWJC;
          map[item.DWDM] = item;
        });
        var val = [];
        data.forEach(function(item) {
          // 以当前遍历项，的pid,去map对象中找到索引的id
          var parent = map[item.SJDWBM];
          // 好绕啊，如果找到索引，那么说明此项不在顶级当中,那么需要把此项添加到，他对应的父级中
          if (parent) {
            (parent.children || (parent.children = [])).push(item);
          } else {
            //如果没有在map中找到对应的索引ID,那么直接把 当前的item添加到 val结果集中，作为顶级
            val.push(item);
          }
        });
        if (val.length > 1) {
          for (var i = 1; i < val.length; i++) {
            val[0].children.push(val[i]);
          }
          val.splice(1);
        }
        return val;
      },
      GetParentArry(Code, arry) {
        var newArr = [];
        for (var i in arry) {
          if (arry[i].ParentCode == Code) newArr.push(arry[i]);
        }
        return newArr;
      },
      download() {
        this.axios
          .post(this.url_s + "/GetExcelTemplate", {
            CompanyCode: this.info.CompanyCode,
            CompanyName: this.CompanyName,
            InputDate: this.date1,
            TypeId: this.checkedTypeid,
            TypeName: this.TypeName,
            DataTypeName: this.DataTypeName,
            DataType: this.DateType,
            DataInterval: this.timeInterval
          })
          .then(res => {
            res = JSON.parse(res.data.d);
            let fileName = res[0].Name;
            let reportValue = res[0].Value;
            this.Down(fileName, reportValue);
          })
          .catch(err => {
            this.alertmsg(this.CompanyName+"暂无模板")
          });
      },
      Down(fileName, reportValue) {
        this.postcall("DownExcel.aspx", {name: fileName,data: reportValue}, "_blank");
      },
      postcall(url, params, target) {
        var tempform = document.createElement("form");
        tempform.action = url;
        tempform.method = "post";
        tempform.style.display = "none";
        if (target) {
          tempform.target = target;
        }
        for (var x in params) {
          var opt = document.createElement("input");
          opt.name = x;
          opt.value = params[x];
          tempform.appendChild(opt);
        }
        var opt = document.createElement("input");
        opt.type = "submit";
        tempform.appendChild(opt);
        document.body.appendChild(tempform);
        tempform.submit();
        document.body.removeChild(tempform);
      },
      clickBtn(){     //兼容ie
        var file = document.getElementById("fileup");
        file.click();
      },
      selectfile() {
        var file = document.getElementById("fileup");
        this.dialogform.fileName = file.files[0].name;
      },
      uploading() {
        if(this.dialogform.fileName=='') 
        return this.dialogform.err='请选择Excel文件导入.';
          var file = document.getElementById("fileup");
          let name = file.files[0].name;
          name = name.substring(0,name.indexOf("."))
        if(this.tableData3 == ''){
          return this.dialogform.err = '当前导入表格暂无数据,请打开要导入表格以便保存上传excel文件能进行录入。'
        }
        this.innerVisible = true;
        let files = file.files[0];
        let params = new FormData();
        params.append("name", files); //
        this.axios.post("ExcelImport.aspx",params)
          .then(res => {
            res = JSON.parse(JSON.stringify(res.data));
            if(typeof(res) == "string"){
              return this.errmsg(res)
            }
        // res = JSON.parse(res)
           let arr = JSON.parse(JSON.stringify(res))
            if(typeof(res) !== "object"){
              this.innerVisible = false;
              return this.alertmsg(res.data)
            } 
            if(this.checkedTypeid != '10'){     //瞬时数据和当日数据表头格式不一样
              res.splice(0,2)
              var datahead = arr[1];
              var thead = document.getElementsByTagName("thead")[0].children[0].children;
            }else{
              res.splice(0,3)
              var datahead = arr[2];
              var thead = document.getElementsByTagName("thead")[0].children[1].children;
            }
            if(res.length === this.tableData3.length){
              for(var i=0;i<res.length;i++){
                if(res[i].COL_CODE !== this.tableData3[i].COL_CODE){
                  this.innerVisible = false;
                  return this.dialogform.err = "上传excel文件模板与当前表格内容不符，请上传正确模板."
                }
              }
              for(var j in datahead){
                if(datahead[j] !== this.sourcehead[j]){
                  this.innerVisible = false;
                  return this.dialogform.err = "上传excel文件模板与当前表格内容不符，请上传正确模板."
                }
              }
            }else{
                this.innerVisible = false;
              return  this.dialogform.err = "上传excel文件与当前表格内容长度不匹配，或者是项目时间间隔不一致，请上传正确模板."
            }
            for(var i=0;i<res.length;i++){
              for(var j in res[i]){     //j键值
                if(j !=="COL_CODE" && j !== "COL_NAME" && res[i][j] !== null && res[i][j] !== ""){
                    this.save(i,j,res[i][j],res.length -1);
                    this.count +=1;
                }
              }
            }
            if(this.count == 0){
              this.innerVisible = false;
              this.dialogform.err = "上传excel文件为空."
            }
          }).catch(err=>{
            this.alertmsg(err.message);
            this.innerVisible = false;
          });
      },
      save(row,col,value,length){
        if(this.checkedTypeid != '10'){  
          var thead = document.getElementsByTagName("thead")[0].children[0].children;
        }else{
          var thead = document.getElementsByTagName("thead")[0].children[1].children;
        }
        let tbody = document.getElementsByTagName("tbody")[0].children[row];
        for(var i=0;i<thead.length;i++){
          if(this.checkedTypeid != '10'){  
            if(col == thead[i].classList[2]){
              tbody.children[thead[i].cellIndex].textContent = value;
              this.SaveGridData(row,col,value,tbody.children[thead[i].cellIndex])
            }
          }else{
            if(col == thead[i].classList[1]){
              tbody.children[thead[i].cellIndex].textContent = value
              this.SaveGridData(row,col,value,tbody.children[thead[i].cellIndex])
            }
          }
        }
      },
      Showdialog() {
        this.dialogvisible = !this.dialogvisible;
        let file = document.getElementById("fileup")
        if (typeof this.date1 === "object") {
          this.date1 = this.formatDate(this.date1);
        }
        if(file != null){
          file.value = ""
        }
        for (var i in this.dialogform) {
          this.dialogform[i] = "";
        }
        
      },
      labelHead(h,{column,index}){
         var reg = new RegExp("[`~!@#$^&*()=|{}':;',\\[\\].<>/?~！@#￥……&*（）——|{}【】‘；：”“'。，、？]");
         let text = column.label;
          let l = "";
           l = column.label.length 
          let f = 16 //每个字大小，其实是每个字的比例值，大概会比字体大小差不多大一点，
          if(reg.test(text)){
            if(f*l<100){
              column.minWidth = 100 + 10
            }else{
              column.minWidth = f*l + 10//字大小乘个数即长度 ,注意不要加px像素，这里minWidth只是一个比例值，不是真正的长度
            }
          }else{
            if(f*l<100){
              column.minWidth = 100
            }else{
              column.minWidth = f*l //字大小乘个数即长度 ,注意不要加px像素，这里minWidth只是一个比例值，不是真正的长度
            }
          }
    　　　　//然后将列标题放在一个div块中，注意块的宽度一定要100%，否则表格显示不完全
            return h('div',{class:'table-head',style:{width:'100%'}},[column.label])
      },
      rowColor({row, rowIndex}){
        if (rowIndex%2 == 1) {
          return 'stripe';
        } 
        return '';
      }
    }
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .table-head{
    font-size:14px!important;
    padding: 0;
  }
  .el-table .cell{
    position:relative;
  }
  .el-table .caret-wrapper{
    position:absolute;
    top:2px;
    right:0;
  }
  .index {
    width: 100%;
    height: 100%;
    display: flex;
    box-sizing: border-box;
  }
  
  .main {
    width: 80%;
    box-sizing: border-box;
    float: left;
    height: 100%;
  }
  
  .main2 {
    width: 100%;
  }
  .nav {
    width: 20%;
    height: 100%;
    padding: 4px 0 0 0px;
    float: left;
    box-sizing: border-box;
    -webkit-box-sizing: border-box;
    position: relative;
    background-color: #f1f2f4;
    border: 1px solid #ccc;
  }
  .isactive{
    background-color: #909399 !important;
    border-color: #909399 !important;
  }
  .el-tree {
    background-color: inherit;
  }
  
  .el-header {
    padding: 0;
  }
  h3 {
    width: 100%;
    height: 40px;
    line-height: 40px;
    font-size: 0;
    box-sizing: border-box;
    background-color: #484c58;
    color: white;
    font-weight: normal;
  }
  h3 .left{
    width: 80%;
    height: 100%;
    font-size: 14px;
    display: inline-block;
    box-sizing: border-box;
    vertical-align: top;
    padding-left: 20px;
  }
  .wid{
    width: 100% !important;
  }
  h3 .circle{
    width: 10px;
    height: 10px;
    border-radius: 50%;
    display: inline-block;
    background-color: #e75f49;
    vertical-align: baseline;
    margin-right: 6px;
  }
  h3 .left div{
    float: right;
    margin-right: 20px;
  }
  h3 .right {
    width: 20%;
    height: 100%;
    font-size: 14px;
    display: inline-block;
    box-sizing: border-box;
  }
  
  h3 .right>span {
    height: 100%;
    font-size: 14px;
    padding-left: 25px;
  }
  
  .items {
    width: 100%;
    height: 35px;
    line-height: 35px;
    padding-left: 20px;
    box-sizing: border-box;
  }
  
  .tablebox {
    width: 100%;
    height: calc(100% - 65px);
    box-sizing: border-box;
  }
  .overs {
    display: inline-block;
    width: 16px;
    margin: 0 6px 0 18px;
    height: 16px;
    vertical-align: middle;
    background-color: #ff0000;
  }

  .unders {
    display: inline-block;
    width: 16px;
    margin: 0 6px 0 16px;
    height: 16px;
    vertical-align: middle;
    background-color: #00ff00;
  }
  .errmsg {
    color: red;
    font-weight: bold;
  }
  
  .block {
    display: inline-block;
  }
  .textmargin {
    margin-right: 10px;
  }
  .edit {
    width: 100%!important;
    height: 100%!important;
    border: none;
    background-color: transparent;
    outline: none!important;
  }
</style>
