<template>
  <div id="wrapper">
    IP段：
    <el-input size="mini" style="width:120px;" placeholder="IP段" v-model="IP_head">
    </el-input>
    开始IP：
    <el-input size="mini" style="width:80px;" maxlength='3' placeholder="开始IP" v-model="IP_start">
    </el-input>
    ~ 结束IP：
    <el-input size="mini" style="width:80px;" maxlength='3' placeholder="结束IP" v-model="IP_end">
    </el-input>
    <el-button size="mini" type="info" @click="searchFun" :loading="loading">扫描</el-button>
    <el-button size="mini" type="info" @click="resetFun('all','cx')">全部程序重启</el-button>
    <el-button size="mini" type="info" @click="resetFun('','cx')">选中程序重启</el-button>
    <!-- 重启设备 -->
    <el-button size="mini" type="info" @click="resetFun('all','sb')">全部设备重启</el-button>
    <el-button size="mini" type="info" @click="resetFun('','sb')">选中设备重启</el-button>

    <!-- <el-tabs style="margin-top:20px;height: -webkit-calc(100vh - 60px);" type="border-card">
      <el-tab-pane label="矿池相关">矿池相关</el-tab-pane>
    </el-tabs> -->
    <div class="setPool">
      <div>
        <p class="mb10">矿池1 <input type="text" v-model="poolConfig.poolurl0"></p>
        <p class="mb10">矿池2 <input type="text" v-model="poolConfig.poolurl1"></p>
        <p>矿池2 <input type="text" v-model="poolConfig.poolurl2"></p>
      </div>
      <div>
        <p class="mb10">账户名1 <input type="text" v-model="poolConfig.poolusr0"></p>
        <p class="mb10">账户名2 <input type="text" v-model="poolConfig.poolusr1"></p>
        <p>账户名3 <input type="text" v-model="poolConfig.poolusr2"></p>
      </div>
      <div>
        <p class="mb10">密码1 <input type="text" v-model="poolConfig.poolpasswd0"></p>
        <p class="mb10">密码2 <input type="text" v-model="poolConfig.poolpasswd1"></p>
        <p>密码3 <input type="text" v-model="poolConfig.poolpasswd2"></p>
      </div>

        <el-popover
    placement="top-start"
    trigger="hover"
    content="配置之后生效">
    <span slot="reference">PLL<span class="tips"></span> ：</span>
  </el-popover>
  <input type="text" style="width:60px;margin-right:10px;" v-model="poolConfig.pll">
      <el-button size="mini" type="info" @click="configPool('all')">配置全部</el-button>
      <el-button size="mini" type="info" @click="configPool()">配置选中</el-button>
    </div>

  <el-popover
    placement="top-start"
    trigger="hover"
    content="设置之后下一次扫描生效">
    <span slot="reference">异常配置<span class="tips"></span> ：</span>
  </el-popover>
  温度： <input type="text" style="width:60px;margin-right:10px;outline: none;" v-model="temperature_err"> 5s平均值：<input type="text" style="width:60px;margin-right:40px;outline: none;" v-model="force_err"> 异常筛选：

    <div class='container'>
      <div class='container_item'>
        <p v-for='(item,index) in err_select' :key="index">
          <input v-model='item.status' :id='"demo"+index' type="checkbox">
          <label :for='"demo"+index' v-text='item.val'></label>
        </p>
      </div>
    </div>

    <el-button size="mini" type="info" @click="filtrate">筛选</el-button>

    <el-button size="mini" type="info" @click="refresh" :loading="refresh_load">刷新</el-button>

    <!-- 表格数据 -->
    <el-table empty-text="" ref="multipleTable" class="tableClass" border :data="tableData3" tooltip-effect="dark" height='calc(100vh - 180px)' style="width: 100%;margin-top:6px;" @selection-change="handleSelectionChange">
      <el-table-column type="selection" fixed="left" width="55" align='center'>
      </el-table-column>
      <el-table-column label="ip" width="120" align='center'>
        <template slot-scope="scope">{{ scope.row.ip }}</template>
      </el-table-column>
      <el-table-column prop="status" label="状态" width="120" align='center'>
        <template slot-scope="scope">
          <span :class="[scope.row.status != 'open'?'redColor':'']">{{scope.row.status}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="result[1].temperature" label="温度" width="180" align='center'>
        <template slot-scope="scope">
          <span :class="[scope.row.fever?'redColor':'']" v-for="(item, index) of scope.row.result[1].temperature" :key="index">
            {{item}}
            <i v-if="index != scope.row.result[1].temperature.length -1">,</i>
          </span>
        </template>
      </el-table-column>
      <el-table-column prop="result[1].pll" label="PLL" width="60" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].fan" label="转速" width="100" align='center'>
      </el-table-column>
      <!-- 5s 平均值 -->
      <el-table-column prop="result[1].5s" label="5s平均值" width="100" align='center'>
        <template slot-scope="scope">
          <span :class="[scope.row.force?'redColor':'']">{{scope.row.result[1]['5s']}}</span>
        </template>
      </el-table-column>
      <!-- 60s 平均值 -->
      <el-table-column prop="result[1].60s" label="60s平均值" width="100" align='center'>
      </el-table-column>
      <!-- 平均值 -->
      <el-table-column prop="result[1].avg" label="平均值" width="100" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolurl0" label="矿池1" width="250" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolusr0" label="账户名1" width="160" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolpasswd0" label="密码1" width="120" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolurl1" label="矿池2" width="250" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolusr1" label="账户名2" width="160" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolpasswd1" label="密码2" width="120" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolurl2" label="矿池3" width="250" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolusr2" label="账户名3" width="160" align='center'>
      </el-table-column>
      <el-table-column prop="result[1].poolpasswd2" label="密码3" width="120" align='center'>
      </el-table-column>

    </el-table>

  </div>
</template>

<script>
export default {
  name: "landing-page",
  components: {},
  data() {
    return {
      IP_head: "",
      IP_start: null,
      IP_end: null,
      default_port: "3080",
      openArr: [],
      ind: 0,
      loading: false,
      refresh_load:false,
      closeArr: [],
      joinArr: [],
      tableData3: [], //展示数据
      dataArr: [], //重启的数据
      configArr: [], //配置的数据
      multipleSelection: [], //选中的数据
      tableData1: [], //查询的临时数组
      poolConfig: {
        poolpasswd0: "",
        poolpasswd1: "",
        poolpasswd2: "",
        poolurl0: "",
        poolurl1: "",
        poolurl2: "",
        poolusr0: "",
        poolusr1: "",
        poolusr2: "",
        pll: ''
      }, //矿机123配置
      temperature_err: "",
      force_err: "",
      err_select: [
        {
          status: false,
          val: "状态"
        },
        {
          status: false,
          val: "温度"
        },
        {
          status: false,
          val: "5s算力"
        }
      ]
    };
  },
  mounted() {
    let that = this;
    // 设置温度 算力 pll
    that.temperature_err = window.localStorage.getItem('temperature_storage')?window.localStorage.getItem('temperature_storage'):'80';
    that.force_err = window.localStorage.getItem('force_storage')?window.localStorage.getItem('force_storage'):'100';
    that.poolConfig.pll = window.localStorage.getItem('pll')?window.localStorage.getItem('pll'):'550';
    // 设置ip
    that.IP_head = window.localStorage.getItem('ip_before')?window.localStorage.getItem('ip_before'):'';
    that.IP_start = window.localStorage.getItem('ip_start')?window.localStorage.getItem('ip_start'):null;
    that.IP_end = window.localStorage.getItem('ip_end')?window.localStorage.getItem('ip_end'):null;
    // 设置矿池
    that.poolConfig.poolurl0 = window.localStorage.getItem('pool1')?window.localStorage.getItem('pool1'):'';
    that.poolConfig.poolurl1 = window.localStorage.getItem('pool2')?window.localStorage.getItem('pool2'):'';
    that.poolConfig.poolurl2 = window.localStorage.getItem('pool3')?window.localStorage.getItem('pool3'):'';
    that.poolConfig.poolusr0 = window.localStorage.getItem('account1')?window.localStorage.getItem('account1'):'';
    that.poolConfig.poolusr1 = window.localStorage.getItem('account2')?window.localStorage.getItem('account2'):'';
    that.poolConfig.poolusr2 = window.localStorage.getItem('account3')?window.localStorage.getItem('account3'):'';
  },
  methods: {
    // 浏览器打开外链
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    verifyFun() {
      var re = /^(\d+)\.(\d+)\.(\d+)$/; //正则表达式
      if (re.test(this.IP_head)) {
        if (RegExp.$1 < 256 && RegExp.$2 < 256 && RegExp.$3 < 256) {
          var re = /^[0-9]+.?[0-9]*$/; //判断字符串是否为数字 //判断正整数 /^[1-9]+[0-9]*]*$/
          if (re.test(parseInt(this.IP_start))) {
            if (parseInt(this.IP_start) >= 0 && parseInt(this.IP_start) <= 255) {
            } else {
              return "lasterr";
            }
          } else {
            return "lasterr";
          }
          if (re.test(parseInt(this.IP_end))) {
            if (parseInt(this.IP_end) >= 0 && parseInt(this.IP_end) <= 255) {
            } else {
              return "lasterr";
            }
          } else {
            return "lasterr";
          }
        } else {
          return "iperr";
        }
      } else {
        return "iperr";
      }
    },
    // 配置检测是否配置项为空
    config_verify() {
      let that = this;
      if (that.poolConfig.poolurl0 == "") {
        return "矿池1不能为空！";
      }
      if (that.poolConfig.poolurl1 == "") {
        return "矿池2不能为空！";
      }
      if (that.poolConfig.poolurl2 == "") {
        return "矿池3不能为空！";
      }
      if (that.poolConfig.poolusr0 == "") {
        return "账户名1不能为空！";
      }
      if (that.poolConfig.poolusr1 == "") {
        return "账户名2不能为空！";
      }
      if (that.poolConfig.poolusr2 == "") {
        return "账户名3不能为空！";
      }
      if (that.poolConfig.pll != "") {
        var re = /^[0-9]+.?[0-9]*$/; //判断字符串是否为数字 //判断正整数 /^[1-9]+[0-9]*]*$/
        if (re.test(that.poolConfig.pll)) {
          if (that.poolConfig.pll < 500 || that.poolConfig.pll > 800) {
            return "PLL范围为500 - 800";
          } else {
            return "";
          }
        } else {
          return "PLL为数字（500 - 800）";
        }
      } else {
        return "PLL不能为空！";
      }
      return "";
    },
    // 获取数据的函数
    getdata(val) {
      let that = this;
      that.ind++;
      if (that.ind == that.openArr.length) {
        that.loading = false;
      }
      return new Promise((resolve, reject) => {
        that.$http
          .post(`http://${val}:${that.default_port}/ubus`, {
            jsonrpc: "2.0",
            id: 1,
            method: "call",
            params: [
              "00000000000000000000000000000000",
              "luci2.basic",
              "btcmanage",
              {
                action: "query" //value:query,Olny query
              }
            ]
          })
          .then(res => {
            resolve(res.data);
          })
          .catch(err => {
            reject(err);
          });
      });
    },
    // 端口扫描函数（返回open状态的ip）
    scanPort(target, port) {
      let that = this;
      var timeout = 500;
      var img = new Image();
      return new Promise((resolve, reject) => {
        img.onerror = function() {
          if (!img) {
            return;
          }
          img = undefined;
          resolve(target);
        };
        img.onload = img.onerror;
        img.src = "http://" + target + ":" + port;

        setTimeout(function() {
          if (!img) {
            return;
          }
          img = undefined;
          reject(target);
        }, timeout);
      });
    },
    // 搜索函数
    searchFun() {
      let that = this;
    // 设置本地存储（温度和5s算力）
      window.localStorage.setItem('temperature_storage',that.temperature_err);
      window.localStorage.setItem('force_storage',that.force_err);
      that.ind = 0;
      that.openArr = [];
      that.closeArr = [];
      that.tableData3 = [];
      that.$refs.multipleTable.clearSelection();
      if (this.verifyFun() === "iperr") {
        this.$alert("请输入正确的IP段！eg：192.168.1", "", {
          confirmButtonText: "确定",
          callback: action => {}
        });
        return;
      }
      if (this.verifyFun() === "lasterr") {
        this.$alert("开始IP和结束IP在0-255之间", "", {
          confirmButtonText: "确定",
          callback: action => {}
        });
        return;
      }
      if (parseInt(this.IP_start) > parseInt(this.IP_end)) {
        this.$alert("开始IP不能大于结束IP", "", {
          confirmButtonText: "确定",
          callback: action => {}
        });
        return;
      }
      // 存储ip段
      window.localStorage.setItem('ip_before',that.IP_head);
      window.localStorage.setItem('ip_start',that.IP_start);
      window.localStorage.setItem('ip_end',that.IP_end);
      // 发起所有请求
      that.loading = true;
      that.getrerch();
    },
    // 刷新
    refresh(){
      let that = this;
      that.refresh_load = true;
      that.err_select[0].status = false;
      that.err_select[1].status = false;
      that.err_select[2].status = false;
      that.tableData1 =[];
      that.tableData3 =[];
      that.getcync();
    },
    async getcync() {
      let that = this;
      if (this.openArr.length == 0) {
        that.loading = false;
        that.refresh_load = false;
      }
      for (const iterator of this.openArr) {
        await this.getdata(iterator)
          .then(res => {
            // console.log("当前异步完成了，可以进行下次循环", res);
            res.ip = iterator;
            // 判断状态
            for (const item1 of res.result[1].core) {
              if (item1 == "open") {
                res.status = "open";
              } else {
                res.status = "close";
              }
            }
            // 判断温度
            res.fever = false;
            for (const item2 of res.result[1].temperature) {
                if (parseInt(item2) > parseInt(that.temperature_err)) {
                  // 温度异常
                  res.fever = true;
                }
            }
            // 判断5s算力
            if (parseFloat(res.result[1]["5s"]) < parseFloat(that.force_err)) {
              // 算力异常
              res.force = true;
            } else {
              res.force = false;
            }
            that.tableData3.push(res);
            that.tableData1 = that.tableData3;
            // console.log("得到最后的数组", that.tableData3);
          })
          .catch(err => {
            console.log(err);
          });
          that.refresh_load = false;
      }
    },
    async getrerch() {
      let that = this;
      var x = parseInt(this.IP_start) - 1;
      for (let i = parseInt(this.IP_start); i <= parseInt(this.IP_end); i++) {
        x++;
        if (x == parseInt(this.IP_end)) {
          // 判断是否扫描完成
          that.getcync();
        }
        await this.scanPort(this.IP_head + "." + i, this.default_port)
          .then(res => {
            that.openArr.push(res);
            // 一次扫描完成
          })
          .catch(err => {
            // console.log(err);
          });
      }
    },
    // 获取选中的数据
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 配置矿机
    async configPool(val) {
      let that = this;
      if (that.config_verify() != "") {
        that.$message({
          showClose: true,
          type: "error",
          center: true,
          message: that.config_verify()
        });
        return;
      }
      // 矿机
      window.localStorage.setItem('pool1',that.poolConfig.poolurl0);
      window.localStorage.setItem('pool2',that.poolConfig.poolurl1);
      window.localStorage.setItem('pool3',that.poolConfig.poolurl2);
      window.localStorage.setItem('account1',that.poolConfig.poolusr0);
      window.localStorage.setItem('account2',that.poolConfig.poolusr1);
      window.localStorage.setItem('account3',that.poolConfig.poolusr2);
      // pll
      window.localStorage.setItem('pll',that.poolConfig.pll);
      // 判断配置那些矿机
      if (val == "all") {
        that.configArr = that.tableData3;
      } else {
        that.configArr = that.multipleSelection;
      }
      if (that.configArr.length == 0) {
        that.$message({
          showClose: true,
          type: "info",
          center: true,
          message: "没有可以配置的数据！"
        });
        return;
      }
      var x = 0;
      for (let i = 0; i < that.configArr.length; i++) {
        await this.configBack(that.configArr[i])
          .then(res => {
            x++;
            if (x == that.configArr.length) {
              // 判断是否所有重启完成
              // console.log("所有配置完成");
              that.$message({
                showClose: true,
                type: "info",
                center: true,
                message: "操作完成！"
              });
            }
          })
          .catch(err => {
            x++;
            if (x == that.configArr.length) {
              // 判断是否所有重启完成
              that.$message({
                showClose: true,
                type: "info",
                center: true,
                message: "操作完成！"
              });
            }
            console.log(err);
          });
      }
    },
    // 配置的回调
    configBack(val) {
      let that = this;
      return new Promise((resolve, reject) => {
        that.$http
          .post(`http://${val.ip}:${that.default_port}/ubus`, {
            jsonrpc: "2.0",
            id: 1,
            method: "call",
            params: [
              "00000000000000000000000000000000",
              "luci2.basic",
              "btcsetinfo",
              {
                routeruuid: val.result[1].routeruuid, //UUID
                poolurl0: that.poolConfig.poolurl0, //矿池信息
                poolurl1: that.poolConfig.poolurl1,
                poolurl2: that.poolConfig.poolurl2,
                poolusr0: that.poolConfig.poolusr0, //矿池用户
                poolusr1: that.poolConfig.poolusr1,
                poolusr2: that.poolConfig.poolusr2,
                poolpasswd0: that.poolConfig.poolpasswd0, //矿池用户密码
                poolpasswd1: that.poolConfig.poolpasswd1,
                poolpasswd2: that.poolConfig.poolpasswd2,
                pll: that.poolConfig.pll + "", //PLL值
                restart: "true", //true,重启程序，false，不重启
                reboot: "false" //true，重启设备，false，不重启
              }
            ]
          })
          .then(res => {
            resolve(res.data);
          })
          .catch(err => {
            reject(err);
          });
      });
    },
    // 重启的回调
    resetBack(val, type) {
      let that = this;
      return new Promise((resolve, reject) => {
        that.$http
          .post(`http://${val.ip}:${that.default_port}/ubus`, {
            jsonrpc: "2.0",
            id: 1,
            method: "call",
            params: [
              "00000000000000000000000000000000",
              "luci2.basic",
              "btcsetinfo",
              {
                routeruuid: val.result[1].routeruuid, //UUID
                poolurl0: val.result[1].poolurl0, //矿池信息
                poolurl1: val.result[1].poolurl1,
                poolurl2: val.result[1].poolurl2,
                poolusr0: val.result[1].poolusr0, //矿池用户
                poolusr1: val.result[1].poolusr1,
                poolusr2: val.result[1].poolusr2,
                poolpasswd0: val.result[1].poolpasswd0, //矿池用户密码
                poolpasswd1: val.result[1].poolpasswd1,
                poolpasswd2: val.result[1].poolpasswd2,
                pll: val.result[1].pll, //PLL值
                restart: type == "cx" ? "true" : "false", //true,重启程序，false，不重启
                reboot: type == "cx" ? "false" : "true" //true，重启设备，false，不重启
              }
            ]
          })
          .then(res => {
            resolve(res.data);
          })
          .catch(err => {
            reject(err);
          });
      });
    },
    // 重启
    async resetFun(val, type) {
      // 判断重启那些矿机
      let that = this;
      if (val == "all") {
        that.dataArr = that.tableData3;
      } else {
        that.dataArr = that.multipleSelection;
      }
      if (that.dataArr.length == 0) {
        that.$message({
          showClose: true,
          type: "info",
          center: true,
          message: "没有可以重启的数据！"
        });
        return;
      }
      var x = 0;
      for (let i = 0; i < that.dataArr.length; i++) {
        await this.resetBack(that.dataArr[i], type)
          .then(res => {
            x++;
            if (x == that.dataArr.length) {
              // 判断是否所有重启完成
              // console.log("所有重启完成");
              that.$message({
                showClose: true,
                type: "info",
                center: true,
                message: "操作完成！"
              });
            }
          })
          .catch(err => {
            x++;
            if (x == that.dataArr.length) {
              // 判断是否所有重启完成
              that.$message({
                showClose: true,
                type: "info",
                center: true,
                message: "操作完成！"
              });
            }
            console.log(err);
          });
      }
    },
    filtrate() {
      if (this.tableData1.length == 0) {
        return;
      }
      var arr1, arr2, arr3;
      if (this.err_select[0].status) {
        arr1 = this.tableData1.filter(function(item, index, array) {
          //元素值，元素的索引，原数组。
          return item.status === "close";
        });
      } else {
        arr1 = this.tableData1.concat([]);
      }

      // 温度异常
      if (this.err_select[1].status) {
        arr2 = arr1.filter(function(item, index, array) {
          //元素值，元素的索引，原数组。
          return item.fever === true;
        });
      } else {
        arr2 = arr1;
      }

      if (this.err_select[2].status) {
        arr3 = arr2.filter(function(item, index, array) {
          //元素值，元素的索引，原数组。
          return item.force === true;
        });
      } else {
        arr3 = arr2;
      }
      this.tableData3 = arr3;
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
  font-size: 14px;
}

#wrapper {
  background-color: rgb(240, 240, 240);
  height: 100vh;
  padding: 10px;
  width: 100vw;
  min-width: 1180px;
}
.setPool {
  width: 100%;
  height: 80px;
  margin: 10px 0;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.setPool input {
  outline: none;
  width: 200px;
}
.setPool > div {
  margin-right: 10px;
  height: 80px;
}
.mb10 {
  margin-bottom: 10px;
}
.tableClass .el-table th {
  padding: 6px 0;
}
.container {
  display: inline-block;
  width: 226px;
}
.container_item {
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.container_item p {
  margin-right: 16px;
}
.redColor {
  color: red;
}
.tips{
  display: inline-block;
  width: 14px;
  height: 14px;
  position: relative;
  background-color: gray;
  border-radius: 50%;
}
.tips::after{
    content: "?";
    color: #fff;
    font-size: 12px;
    position: absolute;
    top: 1px;
    left: 3px;
}
</style>
