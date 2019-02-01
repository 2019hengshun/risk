<template>
    <div class="container">
        <el-row>
            <el-col :span="6">
                <el-button type="primary" @click="handleEdit()">新增用户</el-button>
            </el-col>
            <el-col :span="18" >
              
                <el-row>
                    <el-col :span="4">
<el-button  type="primary" @click="reset">重置</el-button>
                    </el-col>
                    <el-col :span="10">

                       <el-date-picker
                           v-model="select_time"
                              type="daterange"
                           range-separator="至"
                           start-placeholder="开始日期"
                           end-placeholder="结束日期"
                           value-format="yyyy-MM-dd">
                       </el-date-picker>   
                    </el-col>
                    <el-col :span="10" >
                        <el-row>
                            <el-col :span="8">
                                <el-input v-model="select_phone" placeholder="请输入手机号搜索" ></el-input>
                            </el-col>
                            <el-col :span="8" class="l20">
                                <el-input v-model="realname" placeholder="请输入姓名" ></el-input>
                            </el-col>                            
                            <el-col :span="4" class="l20">
                                 <el-button type="primary" icon="search" @click="handleSearch" >搜索</el-button> 
                            </el-col>    
                        </el-row>
                    </el-col>
                </el-row>

            </el-col>            
        </el-row>
        
        <el-row style="margin-top:20px">
           <el-table
                :data="tableData"
                 v-loading="loading"
                border
                style="width: 100%">
                <el-table-column
                    sortable
                    prop="id"
                    label="序号"
                     width="100">
                </el-table-column>
                 <el-table-column
                  v-if="companyName"
                    prop="companyname"
                    label="公司名称"
                    width="120">
                 </el-table-column>                 
                 <el-table-column
                    prop="realname"
                    label="姓名"
                    width="100">
                 </el-table-column>
                <el-table-column
                    prop="idcard"
                    width="300"
                    label="身份证">
                </el-table-column>
                <el-table-column
                    prop="phonenume"
                    label="手机号">
                </el-table-column>
                <el-table-column
                    sortable
                    label="时间">
                    <template slot-scope="scope">
                      <i class="el-icon-time"></i>
                      <span style="margin-left: 10px">{{ scope.row.applytime |dateServer }}</span>
                    </template> 
                </el-table-column> 
                <el-table-column
                    label="操作">
                    <template slot-scope="scope">
                    <el-button
                        size="mini"
                        type="primary"
                        @click="handleReport(scope.$index, scope.row)">查看报告</el-button>
                    </template>                                
                </el-table-column>
            </el-table>            
        </el-row>
        <el-row style="margin-top:20px" v-if="allpage>0">
            <div style="float:right">
                <el-pagination
                  @current-change="handleCurrentChange"
                  
                   @size-change="handleSizeChange"
                  :current-page="currentPage"
                   :page-sizes="[10, 20, 30, 40]"
                    :page-size="pageSize"
                  background
                  layout="total,sizes,prev, pager, next,jumper"
                  :total="allpage">
                </el-pagination>   
            </div>
        </el-row>  
   
            <el-dialog 
            title="白骑士报告" 
            :visible.sync="dialogWhiteVisible" 
            center
            width="60%"
              ref="multipleTable" 
            >
              <el-card class="box-card">
                <table style="font-size:14px;line-height:30px;width:100%;table-layout: fixed" id="bqs">
                  <template v-if="whiteForm&&whiteForm.loan">
                    <tr>
                      <th style="width:250px">
                        <el-alert
                            title="贷款情况"
                            type="success"
                            center
                            :closable="false">
                        </el-alert>
                      </th>
                      <th >
                        <el-tag :type="whiteForm.loan.finalDecision=='Reject'?'danger':
                                whiteForm.loan.finalDecision=='Review'?'info':
                                whiteForm.loan.finalDecision=='Accept'?'success':''"> 
                          {{whiteForm.loan.finalDecision=='Reject'?'建议拒绝':
                            whiteForm.loan.finalDecision=='Review'?'建议重审':
                            whiteForm.loan.finalDecision=='Accept'?'建议通过':''
                          }}
                        </el-tag>
                      </th>
                    </tr>
                    <template v-for="(temp,index) in whiteForm.loan.strategySet">
                        <tr :key="index">
                          <th style="width:250px">
                            <el-alert
                                :title="temp.strategyName"
                                type="success"
                                center
                                :closable="false">
                            </el-alert>
                          </th>
                          <th >
                            <el-tag :type="temp.strategyDecision=='Reject'?'danger':
                                temp.strategyDecision=='Review'?'info':
                                temp.strategyDecision=='Accept'?'success':''"> 
                              {{temp.strategyDecision=='Reject'?'建议拒绝':
                                temp.strategyDecision=='Review'?'建议重审':
                                temp.strategyDecision=='Accept'?'建议通过':''
                              }}
                            </el-tag>
                          </th>
                        </tr>
                        <template v-for="(t,i) in temp.hitRules">
                          <tr :key="i" >
                            <th  style="width:250px">
                                <el-alert
                                    :title="t.ruleName"
                                    type="info"
                                    center
                                    :closable="false">
                                </el-alert>                      
                                </th>
                                <td  style="text-align:center;width:450px">
                                  <el-tag type="danger"> 
                                    {{t.memo}}
                                  </el-tag>
                                </td>
                          </tr>
                        </template>
                    </template>
                  </template>
                  <template v-if="whiteForm&&whiteForm.register">
                    <tr>
                      <th style="width:250px">
                        <el-alert
                            title="注册情况"
                            type="success"
                            center
                            :closable="false">
                        </el-alert>
                      </th>
                      <th >
                        <el-tag :type="whiteForm.register.finalDecision=='Reject'?'danger':
                                whiteForm.register.finalDecision=='Review'?'info':
                                whiteForm.register.finalDecision=='Accept'?'success':''"> 
                          {{whiteForm.register.finalDecision=='Reject'?'建议拒绝':
                            whiteForm.register.finalDecision=='Review'?'建议重审':
                            whiteForm.register.finalDecision=='Accept'?'建议通过':''
                          }}
                        </el-tag>
                      </th>
                    </tr>
                    <template v-for="(temp,index) in whiteForm.register.strategySet">
                        <tr :key="index">
                          <th style="width:250px">
                            <el-alert
                                :title="temp.strategyName"
                                type="success"
                                center
                                :closable="false">
                            </el-alert>
                          </th>
                          <th >
                            <el-tag :type="temp.strategyDecision=='Reject'?'danger':
                                temp.strategyDecision=='Review'?'info':
                                temp.strategyDecision=='Accept'?'success':''"> 
                              {{temp.strategyDecision=='Reject'?'建议拒绝':
                                temp.strategyDecision=='Review'?'建议重审':
                                temp.strategyDecision=='Accept'?'建议通过':''
                              }}
                            </el-tag>
                          </th>
                        </tr>
                        <template v-for="(t,i) in temp.hitRules">
                          <tr :key="i">
                            <th  style="width:250px"> 
                                <el-alert
                                    :title="t.ruleName"
                                    type="info"
                                    center
                                    :closable="false">
                                </el-alert>                      
                              </th>
                              <td style="text-align:center;width:450px" >
                                  <el-tag type="danger" v-if="t.memo"> 
                                    {{t.memo}}
                                  </el-tag>
                                </td>
                          </tr>
                        </template>
                    </template>
                  </template>                  
                                  
                </table>
              </el-card>            
        </el-dialog>             

    </div>
</template>

<script>
import { mapGetters } from "vuex";
import {
  httpGetCreditReport,
  httpPostCarryReport,
  httpGetZMCReport,
  httpGetBQSReport
} from "@/api/http";
import { timeFormat } from "../../../../static/js/time";
import Print from "print-js";
import "../../../assets/libs/jquery/jQuery.print.js";
export default {
  data() {
    return {
      form: {
        name: "",
        date: "",
        address: ""
      },
      loading: true,
      select_time: [],
      select_word: "",
      editVisible: false,
      showVisible: false,
      fullscreen: false,
      tableData: [],
      pageSize: 20,
      currentPage: 1,
      allpage: 0,
      select_phone: "",
      userBasicInformation: {}, //用户基本信息检测用户基本信息检测
      userInfo: {},
      fullscreenWidth: "80%",
      fullscreenHeight: "750px",
      kinsfolkTableData: [], //亲属联系人
      testTableData: [],
      operatorTableData: [], //运营商消费数据
      linkmanTableData: [], //联系人区域汇总
      operatorDataTableData: [], //运行商数据分析
      contactTableData: [], //联系人信息
      esAddTableData: [], //电商信息
      esDataTableData: [], //电商数据统计
      tripTableData: [],
      canvasimg: "",
      canvasShow: true,
      note: {
        backgroundImage: ""
        // backgroundRepeat: "no-repeat",
        // backgroundSize: "25px auto",
        // marginTop: "5px"
      },
      realname: "",
      dialogWhiteVisible: false,
      whiteForm: {}
    };
  },
  computed: {
    ...mapGetters([
      "role"
      // ...
    ]),
    companyName: () => {
      return localStorage.getItem("chbR") == null
        ? false
        : JSON.parse(localStorage.getItem("chbR")).indexOf("view_admin") != -1;
    }
  },
  methods: {
    reset() {
      this.select_phone = "";
      this.select_time = [];
      this.getTableData(
        this.currentPage,
        this.pageSize,
        "",
        "",
        this.select_phone,
        5
      );
    },
    _httpGetZMCReport(zmcReportId) {
      httpGetZMCReport(zmcReportId)
        .then(res => {
          console.log(res);
        })
        .catch();
    },
    handleEdit() {
      this.editVisible = true;
    },
    // 保存编辑
    saveEdit() {
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
    },
    //查看报告
    handleReport(index, row) {
      let mifengReportId = row.reportId;
      let _this = this;
      // let zmcReportId = row.zmcReportId;
      // this._httpGetZMCReport(zmcReportId);
      httpGetBQSReport(mifengReportId)
        .then(res => {
          let data = res.data;
          this.dialogWhiteVisible = true;
          if (data.code == 200) {
            _this.whiteForm = JSON.parse(JSON.stringify(data.data));
          }
        })

        .catch(err => {});
    },
    changeFullscreen() {
      if (this.fullscreen) {
        this.fullscreen = false;
        this.fullscreenWidth = "80%";
        this.fullscreenHeight = "450px";
      } else {
        this.fullscreen = true;
        this.fullscreenWidth = "100%";
        this.fullscreenHeight = "1050px";
      }
    },
    //得到数据
    getTableData(
      pagesize,
      npage,
      startDate,
      EndDate,
      phonenume,
      interfaceCallType,
      realname
    ) {
      let _this = this;
      _this.loading = true;
      httpGetCreditReport(
        pagesize,
        npage,
        startDate,
        EndDate,
        phonenume,
        5,
        realname
      )
        .then(res => {
          let data = res.data;
          _this.allpage = data.allsize;
          _this.currentPage = data.npage;
          _this.tableData = data.list;
          _this.loading = false;
          setTimeout(() => {
            _this.loading = false;
          }, 5000);
        })
        .catch(() => {
          _this.loading = false;
        });
    },
    handleSearch() {
      if (
        this.select_time == null &&
        this.select_time &&
        this.select_time.length
      ) {
        this.getTableData(
          this.currentPage,
          this.pageSize,
          "",
          "",
          this.select_phone,
          5,
          this.realname
        );
      } else {
        this.getTableData(
          this.currentPage,
          this.pageSize,
          this.select_time[0],
          this.select_time[1],
          this.select_phone,
          5,
          this.realname
        );
      }
    },
    // handleSearch() {
    //   this.currentPage = 1;
    //   console.log(this.select_time);
    //   if (this.select_time && this.select_time.length) {
    //     this.getTableData(
    //       this.currentPage,
    //       this.pageSize,
    //       "",
    //       "",
    //       this.select_phone
    //     );
    //   } else {
    //     this.getTableData(
    //       this.currentPage,
    //       this.pageSize,
    //       ...this.select_time,
    //       this.select_phone
    //     );
    //   }
    // },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.handleSearch();
    },
    handleSizeChange(val) {
      this.pageSize = val;
      this.handleSearch();
    },
    // handleCurrentChange(val) {
    //   if (this.select_time.length === 0) {
    //     this.getTableData(val, this.pageSize, "", "", this.select_phone);
    //   } else {
    //     this.getTableData(
    //       val,
    //       this.pageSize,
    //       ...this.select_time,
    //       this.select_phone
    //     );
    //   }
    // },
    // handleSizeChange(val) {
    //   this.pageSize = val;
    //   this.getTableData(this.currentPage, this.pageSize);
    // },

    Print() {
      console.log(jQuery("#subOutputRank-print"));
      // jQuery("#subOutputRank-print").print({
      //   //Use Global styles
      //   globalStyles: true,
      //   //Add link with attrbute media=print
      //   mediaPrint: false,
      //   //Custom stylesheet
      //   //Print in a hidden iframe
      //   //Don't print this
      //   //Add this at top
      //   //   stylesheet: "./static/css/print.css",
      //   prepend: "恒盾云<br/>",
      //   timeout: 4000
      //   //Add this on bott
      //   // append: "<br/>Buh Bye!"
      // });

      jQuery("#subOutputRank-print").print({
        //Use Global styles
        globalStyles: false,
        //Add link with attrbute media=print
        mediaPrint: false,
        //Custom stylesheet
        //Print in a hidden iframe
        //Don't print this
        //Add this at top
        stylesheet: "./static/css/print.css",
        prepend: "恒盾云<br/>",
        timeout: 4000
        //Add this on bott
        // append: "<br/>Buh Bye!"
      });
    },
    toPrint() {
      console.log(Print);
      let subOutputRankPrint = document.getElementById("subOutputRank-print");
      let newContent = subOutputRankPrint.innerHTML;
      let oldContent = document.body.innerHTML;
      document.body.innerHTML = newContent;
      window.print();
      window.location.reload();
      document.body.innerHTML = oldContent;
      return false;
    }

    // print() {
    //   console.log(1);
    //   var Print = function(dom, options) {
    //     if (!(this instanceof Print)) return new Print(dom, options);

    //     this.options = this.extend(
    //       {
    //         noPrint: ".no-print"
    //       },
    //       options
    //     );

    //     if (typeof dom === "string") {
    //       this.dom = document.querySelector(dom);
    //     } else {
    //       this.dom = dom;
    //     }

    //     this.init();
    //   };
    //   Print.prototype = {
    //     init: function() {
    //       var content = this.getStyle() + this.getHtml();
    //       var isFirst = true;
    //       // function completeLoading() {
    //       //   if (document.readyState == "complete") {
    //       //     console.log(document.readyState);
    //       //
    //       //   }
    //       // }
    //       this.writeIframe(content);
    //     },
    //     extend: function(obj, obj2) {
    //       for (var k in obj2) {
    //         obj[k] = obj2[k];
    //       }
    //       return obj;
    //     },

    //     getStyle: function() {
    //       var str = "",
    //         styles = document.querySelectorAll("style,link");

    //       for (var i = 0; i < styles.length; i++) {
    //         str += styles[i].outerHTML;
    //       }
    //       str +=
    //         "<style>" +
    //         (this.options.noPrint ? this.options.noPrint : ".no-print") +
    //         "{display:none;}</style>";

    //       return str;
    //     },

    //     getHtml: function() {
    //       var inputs = document.querySelectorAll("input");
    //       var textareas = document.querySelectorAll("textarea");
    //       var selects = document.querySelectorAll("select");

    //       for (var k in inputs) {
    //         if (inputs[k].type == "checkbox" || inputs[k].type == "radio") {
    //           if (inputs[k].checked == true) {
    //             inputs[k].setAttribute("checked", "checked");
    //           } else {
    //             inputs[k].removeAttribute("checked");
    //           }
    //         } else if (inputs[k].type == "text") {
    //           inputs[k].setAttribute("value", inputs[k].value);
    //         }
    //       }

    //       for (var k2 in textareas) {
    //         if (textareas[k2].type == "textarea") {
    //           textareas[k2].innerHTML = textareas[k2].value;
    //         }
    //       }

    //       for (var k3 in selects) {
    //         if (selects[k3].type == "select-one") {
    //           var child = selects[k3].children;
    //           for (var i in child) {
    //             if (child[i].tagName == "OPTION") {
    //               if (child[i].selected == true) {
    //                 child[i].setAttribute("selected", "selected");
    //               } else {
    //                 child[i].removeAttribute("selected");
    //               }
    //             }
    //           }
    //         }
    //       }

    //       return this.dom.outerHTML;
    //     },

    //     writeIframe: function(content) {
    //       var w,
    //         doc,
    //         iframe = document.createElement("iframe"),
    //         f = document.body.appendChild(iframe);
    //       iframe.id = "myIframe";
    //       iframe.style =
    //         "position:absolute;width:0;height:0;top:-10px;left:-10px;";

    //       w = f.contentWindow || f.contentDocument;
    //       doc = f.contentDocument || f.contentWindow.document;
    //       doc.open();
    //       doc.write(content);
    //       doc.close();
    //       this.toPrint(w);
    //       setTimeout(function() {
    //         document.body.removeChild(iframe);
    //       }, 100);
    //     },

    //     toPrint: function(frameWindow) {
    //       try {
    //         console.log(frameW);
    //         setTimeout(function() {
    //           frameWindow.focus();
    //           try {
    //             if (!frameWindow.document.execCommand("print", false, null)) {
    //               frameWindow.print();
    //             }
    //           } catch (e) {
    //             frameWindow.print();
    //           }
    //           frameWindow.close();
    //         }, 10);
    //       } catch (err) {
    //         console.log("err", err);
    //       }
    //     }
    //   };

    //   Print("#subOutputRank-print");
    // }
  },
  mounted: function() {
    this.getTableData(
      this.currentPage,
      this.pageSize,
      "",
      "",
      this.select_phone,
      5
    );
  },
  created() {}
};
</script>
<style >
#table .el-dialog__body {
  padding: 0px 20px 20px;
  height: auto;
}
</style>

<style scoped>
#subOutputRank-print {
  height: auto;
}
.title {
  padding: 10px;
  font-weight: bold;
  font-size: 18px;
}
.input-width {
  width: 215px;
}
.btn-fullscreen {
  position: relative;
  width: 30px;
  height: 30px;
  text-align: center;
  border-radius: 15px;
  cursor: pointer;
}
.el-main-info {
  color: #999;
}
.l {
  float: left;
}
.r {
  float: right;
}
.report_t {
  overflow: hidden;
  color: #999;
}
.el-card {
  margin-top: 24px;
}
.panel_title {
  margin-bottom: 10px;
  border-radius: 10px;
  position: relative;
  height: 40px;
  text-align: center;
}
.panel_title h2 {
  height: 30px;
  line-height: 30px;
  display: inline-block;
  background: #e88f08;
  color: #fff;
  border-radius: 100px;
  padding: 0;
  margin: 0;
  font-size: 16px;
  position: absolute;
  z-index: 99;
  width: 200px;
  left: 50%;
  margin-left: -100px;
}
.line {
  background: #e88f08;
  height: 4px;
  top: 13px;
  position: absolute;
  width: 100%;
  border-radius: 10px;
}
.table {
  width: 100%;
  border-radius: 2px;
  border: 1px solid #f1f1f1;
  border-bottom: none;
}
/* .table tr:hover {
  background:#c0b184 ;
} */
.table tr td {
  padding: 10px;
  border-bottom: 1px solid #f1f1f1;
}
.table tr th {
  padding: 10px;
  border-bottom: 1px solid #f1f1f1;
  font-size: 14px;
  text-align: left;
  background: #f7f7f9;
}

.table span {
  margin-right: 20px;
  display: inline-block;
}
span.item {
  width: 60px;
  font-weight: bold;
  text-align: right;
}
.tip_y {
  background: #5cb85c;
  padding: 2px 10px;
  border-radius: 50px;
  color: #fff;
  font-size: 12px;
}
span.remarks {
  display: block;
  padding-left: 85px;
  padding-top: 5px;
  color: #c0b184;
}

.tip {
  background: #ccbfae;
  padding: 2px 10px;
  border-radius: 50px;
  color: #fff;
  font-size: 12px;
}
.wrap {
  margin: 0 auto;
  padding: 20px;
  width: 640px;
  border: 1px solid #ccc;
}
.form .row {
  padding: 10px 0 0;
}
.btn {
  display: block;
  margin: 20px auto;
  padding: 8px 16px;
}
</style>

<style media="print" type="text/css" scoped>
.noprint {
  display: none;
}
#subOutputRank-print {
  height: auto;
}
.title {
  padding: 10px;
  font-weight: bold;
  font-size: 18px;
}
.input-width {
  width: 215px;
}
.btn-fullscreen {
  position: relative;
  width: 30px;
  height: 30px;
  text-align: center;
  border-radius: 15px;
  cursor: pointer;
}
.el-main-info {
  color: #999;
}
.l {
  float: left;
}
.r {
  float: right;
}
.report_t {
  overflow: hidden;
  color: #999;
}
.el-card {
  margin-top: 24px;
}
.panel_title {
  margin-bottom: 10px;
  border-radius: 10px;
  position: relative;
  height: 40px;
  text-align: center;
}
.panel_title h2 {
  height: 30px;
  line-height: 30px;
  display: inline-block;
  background: #e88f08;
  color: #fff;
  border-radius: 100px;
  padding: 0;
  margin: 0;
  font-size: 16px;
  position: absolute;
  z-index: 99;
  width: 200px;
  left: 50%;
  margin-left: -100px;
}
.line {
  background: #e88f08;
  height: 4px;
  top: 13px;
  position: absolute;
  width: 100%;
  border-radius: 10px;
}
.table {
  width: 100%;
  border-radius: 2px;
  border: 1px solid #f1f1f1;
  border-bottom: none;
}
/* .table tr:hover {
  background:#c0b184 ;
} */
.table tr td {
  padding: 10px;
  border-bottom: 1px solid #f1f1f1;
}
.table tr th {
  padding: 10px;
  border-bottom: 1px solid #f1f1f1;
  font-size: 14px;
  text-align: left;
  background: #f7f7f9;
}

.table span {
  margin-right: 20px;
  display: inline-block;
}
span.item {
  width: 60px;
  font-weight: bold;
  text-align: right;
}
.tip_y {
  background: #5cb85c;
  padding: 2px 10px;
  border-radius: 50px;
  color: #fff;
  font-size: 12px;
}
span.remarks {
  display: block;
  padding-left: 85px;
  padding-top: 5px;
  color: #c0b184;
}

.tip {
  background: #ccbfae;
  padding: 2px 10px;
  border-radius: 50px;
  color: #fff;
  font-size: 12px;
}
.wrap {
  margin: 0 auto;
  padding: 20px;
  width: 640px;
  border: 1px solid #ccc;
}
.form .row {
  padding: 10px 0 0;
}
.btn {
  display: block;
  margin: 20px auto;
  padding: 8px 16px;
}
</style> 
<style>
#bqs .el-tag {
  white-space: wrap;
}
</style>
