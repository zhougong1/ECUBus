<template>
  <div class="connect">
    <el-row style="margin:0px">
      <el-col :span="11">
        <div class="title">Device:</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="11">
        <el-select v-model="device" placeholder="请选择" :disabled="connected">
          <el-option
            v-for="item in deviceList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-col>
      <el-col :span="11" :offset="1">
        <el-checkbox v-model="canfd" border :disabled="connected">CAN-FD</el-checkbox>
      </el-col>
    </el-row>
    <div v-if="!canfd">
      <el-row style="margin:0px">
        <el-col :span="11">
          <div class="title">Nominal Bit rate:</div>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-select v-model="speed" placeholder="请选择" :disabled="connected">
            <el-option
              v-for="item in canSpeed"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-col>
      </el-row>
    </div>
    <div v-else>
      <el-row style="margin:0px">
        <el-col :span="11">
          <div class="title">Nominal Bit rate:</div>
        </el-col>
        <el-col :span="11" :offset="1">
          <div class="title">Data Bit rate:</div>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-select v-model="nomSpeed" placeholder="请选择" :disabled="connected">
            <el-option
              v-for="item in canNomSpeed"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-col>
        <el-col :span="11" :offset="1">
          <el-select v-model="dataSpeed" placeholder="请选择" :disabled="connected">
            <el-option
              v-for="item in canDataSpeed"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-col>
      </el-row>
    </div>
    <el-row style="text-align:right">
      <el-col :span="4" :offset="20" v-if="!connected">
        <el-button type="primary" plain @click="connect">Connect</el-button>
      </el-col>
      <el-col :span="4" :offset="20" v-else>
        <el-button type="danger" plain @click="disconnect">Disconnect</el-button>
      </el-col>
    </el-row>
    <el-divider></el-divider>
    <div>
      <el-form :model="tpConfig" :disabled="!connected">
        <el-form-item label="Name:">
          <el-input v-model="tpConfig.name" placeholder="Name" maxlength="20" style="width:180px"></el-input>
        </el-form-item>
        <el-form-item label="Format:">
          <el-select v-model="tpConfig.FORMAT" placeholder="地址格式">
            <el-option
              v-for="item in formatList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Address:">
          <el-radio-group v-model="tpConfig.TA_TYPE">
            <el-radio-button :label="1">物理地址</el-radio-button>
            <el-radio-button :label="2">功能地址</el-radio-button>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="Message:">
          <el-radio-group v-model="tpConfig.MSGTYPE">
            <el-radio-button :label="1">本地</el-radio-button>
            <el-radio-button :label="2">远程</el-radio-button>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="CAN ID:">
          <el-radio-group v-model="tpConfig.IDTYPE" :disabled="bit29Must">
            <el-radio-button :label="1">11 Bits</el-radio-button>
            <el-radio-button :label="2">29 Bits</el-radio-button>
          </el-radio-group>
        </el-form-item>
        <el-form-item required>
          <el-col :span="7">
            <div>SA:</div>
            <el-input
              style="width:120px"
              v-model="tpConfig.SA"
              placeholder="SA"
              maxlength="2"
              show-word-limit
            >
              <template slot="prepend">0x</template>
            </el-input>
          </el-col>
          <el-col :span="7" :offset="1">
            <div>TA:</div>
            <el-input
              style="width:120px"
              v-model="tpConfig.TA"
              placeholder="TA"
              maxlength="2"
              show-word-limit
            >
              <template slot="prepend">0x</template>
            </el-input>
          </el-col>
          <el-col :span="7" :offset="1">
            <div>RA:</div>
            <el-input
              style="width:120px"
              v-model="tpConfig.RA"
              placeholder="RA"
              maxlength="2"
              show-word-limit
            >
              <template slot="prepend">0x</template>
            </el-input>
          </el-col>
        </el-form-item>
        <el-alert v-if="mapMust" title="ID和地址的映射是必须的" type="info" effect="dark" :closable="false"></el-alert>
        <!-- <div v-else>
                    <div>TX-CANID: <span class="header">0x{{txId}}</span></div>
                    <div>RX-CANID: <span class="header">0x{{rxId}}</span></div>
        </div>-->
        <el-form-item v-if="mapMust" required>
          <el-col :span="8">
            MAP TX_ID:
            <el-input v-model="tpConfig.txId" placeholder="CAN TX ID" maxlength="8" show-word-limit>
              <template slot="prepend">0x</template>
            </el-input>
          </el-col>
          <el-col :span="8" :offset="1">
            MAP RX_ID:
            <el-input v-model="tpConfig.rxId" placeholder="CAN RX ID" maxlength="8" show-word-limit>
              <template slot="prepend">0x</template>
            </el-input>
          </el-col>
        </el-form-item>
        <el-form-item style="text-align:right">
          <el-button @click="setupAddr" type="primary" plain>ADD</el-button>
        </el-form-item>
      </el-form>
    </div>
    <el-table size="mini" :data="addrTable" style="width: 100%">
      <el-table-column type="index" width="50" align="center"></el-table-column>
      <el-table-column prop="name" label="名字" width="150" align="center"></el-table-column>
      <el-table-column prop="txId" label="发送ID" width="150" align="center">
        <template slot-scope="scope">
          <el-tag size="medium" type="info">0X{{scope.row.txId}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="rxId" label="接受ID" width="150" align="center">
        <template slot-scope="scope">
          <el-tag size="medium" type="info">0X{{scope.row.rxId}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="maped" label="地址映射" width="50" align="center">
        <template slot-scope="scope">
          <i class="el-icon-circle-check" v-if="scope.row.maped" style="color:green"></i>
          <i class="el-icon-circle-close" v-else style="color:red"></i>
        </template>
      </el-table-column>
      <el-table-column prop="SA" label="SA" width="100" align="center">
        <template slot-scope="scope">
          <el-tag size="medium" type="info">0X{{scope.row.SA.toString(16)}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="TA" label="TA" width="100" align="center">
        <template slot-scope="scope">
          <el-tag size="medium" type="info">0X{{scope.row.TA.toString(16)}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="TA_TYPE" label="目标地址类型" align="center" width="150">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.TA_TYPE==1" size="medium">物理地址</el-tag>
          <el-tag v-else size="medium" type="success">功能地址</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="RA" label="RA" width="100" align="center">
        <template slot-scope="scope">
          <el-tag size="medium" type="info">0X{{scope.row.RA.toString(16)}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="FORMAT" label="地址格式" width="200" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.FORMAT==1" type="warning" size="medium">Normal addressing</el-tag>
          <el-tag
            v-else-if="scope.row.FORMAT==2"
            size="medium"
            type="warning"
          >Normal fixed addressing</el-tag>
          <el-tag v-else-if="scope.row.FORMAT==3" size="medium" type="warning">Extended addressing</el-tag>
          <el-tag v-else-if="scope.row.FORMAT==4" size="medium" type="warning">Mixed addressing</el-tag>
          <el-tag v-else size="medium" type="warning">Enhanced addressing</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="IDTYPE" label="ID长度" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.IDTYPE==1" size="medium">11 Bits</el-tag>
          <el-tag v-else size="medium" type="success">29 Bits</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="MSGTYPE" label="帧格式" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.MSGTYPE==1" size="medium">本地帧</el-tag>
          <el-tag v-else size="medium" type="success">远程帧</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="danger"
            :disabled="!connected"
            @click="handleDelete(scope.$index)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
//import PCANTP from './../../build/Release/PCANTP.node'

const { ipcRenderer } = require("electron");
export default {
  data: function() {
    return {
      tpConfig: {
        name: "defaultName1",
        SA: "0",
        TA: "0",
        RA: "0",
        txId: "0",
        rxId: "0",
        MSGTYPE: 1,
        FORMAT: 1,
        IDTYPE: 1,
        TA_TYPE: 1
      },
      formatList: [
        {
          label: "Normal addressing",
          value: 1
        },
        {
          label: "Normal fixed addressing",
          value: 2
        },
        {
          label: "Extended addressing",
          value: 3
        },
        {
          label: "Mixed addressing",
          value: 4
        }
        // {
        //   label: 'Enhance addressing',
        //   value: PCANTP.PCANTP_FORMAT_ENHANCED
        // }
      ],
      canfd: false,
      device: 0x51,
      deviceList: [
        {
          label: "PEAK-USB1",
          value: 0x51
        },
        {
          label: "PEAK-USB2",
          value: 0x52
        }
      ],
      speed: 0x001c,
      canSpeed: [
        {
          label: "250 KBit/s",
          value: 0x011c
        },
        {
          label: "500 KBit/s",
          value: 0x001c
        },
        {
          label: "1 MBit/s",
          value: 0x0014
        }
      ],
      nomSpeed:
        "f_clock=80000000,nom_brp=10,nom_tseg1=12,nom_tseg2=3,nom_sjw=1,",
      canNomSpeed: [
        {
          label: "500 KBit/s",
          value:
            "f_clock=80000000,nom_brp=10,nom_tseg1=12,nom_tseg2=3,nom_sjw=1,"
        },
        {
          label: "1 MBit/s",
          value:
            "f_clock=80000000,nom_brp=10,nom_tseg1=5,nom_tseg2=2,nom_sjw=1,"
        }
      ],
      dataSpeed: "data_brp=4,data_tseg1=7,data_tseg2=4,data_sjw=1",
      canDataSpeed: [
        {
          label: "2 MBit/s",
          value: "data_brp=4,data_tseg1=7,data_tseg2=4,data_sjw=1"
        },
        {
          label: "4 MBit/s",
          value: "data_brp=2,data_tseg1=7,data_tseg2=4,data_sjw=1"
        }
      ]
    };
  },
  computed: {
    addrTable: function() {
      return this.$store.state.canAddrTable;
    },
    connected: function() {
      return this.$store.state.canConnect;
    },
    txId: function() {
      let ta = parseInt(this.tpConfig.TA, 16) << 8;
      let sa = parseInt(this.tpConfig.SA, 16);
      if (this.tpConfig.FORMAT === 2) {
        return (0x18db0000 + ta + sa).toString(16);
      } else {
        return (0x18ce0000 + ta + sa).toString(16);
      }
    },
    rxId: function() {
      let ta = parseInt(this.tpConfig.SA, 16) << 8;
      let sa = parseInt(this.tpConfig.TA, 16);
      if (this.tpConfig.FORMAT === 2) {
        return (0x18db0000 + ta + sa).toString(16);
      } else {
        return (0x18ce0000 + ta + sa).toString(16);
      }
    },
    bit29Must: function() {
      if (this.tpConfig.FORMAT === 2) {
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.tpConfig.IDTYPE = 2;
        return true;
      } else {
        return false;
      }
    },
    mapMust: function() {
      if (this.tpConfig.FORMAT === 2) {
        return false;
      }
      if (this.tpConfig.FORMAT === 4 && this.tpConfig.IDTYPE === 2) {
        return false;
      }
      return true;
    }
  },
  methods: {
    handleDelete(index) {
      if (this.addrTable[index].maped) {
        let err = ipcRenderer.sendSync("canDeleteMap", [
          this.addrTable[index].txId,
          this.addrTable[index].rxId
        ]);
        if (err.err !== 0) {
          this.$notify.error({
            title: "Error",
            message: err.msg
          });
          return;
        }
      }
      this.$store.commit("canAddrDelete", index);
    },
    setupAddr() {
      var item = JSON.parse(JSON.stringify(this.tpConfig));
      for (var i in this.addrTable) {
        if (this.addrTable[i].name === item.name) {
          this.$notify.error({
            title: "Error",
            message: "Name重复"
          });
          return;
        }
      }
      if (item.SA === item.TA) {
        this.$notify.error({
          title: "Error",
          message: "SA不能等于TA"
        });
        return;
      }
      if (item.txId === item.rxId) {
        this.$notify.error({
          title: "Error",
          message: "TX_ID不能等于RX_ID"
        });
        return;
      }
      item.maped = this.mapMust;
      item.SA = parseInt(item.SA, 16);
      item.TA = parseInt(item.TA, 16);
      item.RA = parseInt(item.RA, 16);

      if (this.mapMust === false) {
        item.rxId = this.rxId;
        item.txId = this.txId;
      } else {
        item.rxId = parseInt(item.rxId, 16);
        item.txId = parseInt(item.txId, 16);
        let err = ipcRenderer.sendSync("canAddMap", item);
        if (err.err !== 0) {
          this.$notify.error({
            title: "Error",
            message: err.msg
          });
          return;
        }
      }
      this.$store.commit("canAddrAdd", item);
    },
    connect() {
      let err;
      if (this.canfd) {
        err = ipcRenderer.sendSync("canConnectFd", [
          this.device,
          this.nomSpeed + this.dataSpeed
        ]);
      } else {
        err = ipcRenderer.sendSync("canConnect", [this.device, this.speed]);
      }
      if (err.err === 0) {
        // this.connected = true
        this.$store.commit("canChange", true);
      } else {
        this.$notify.error({
          title: "Error",
          message: err.msg
        });
      }
    },
    disconnect() {
      let err = ipcRenderer.sendSync("canDisconnect", this.device);
      if (err.err === 0) {
        this.$store.commit("canChange", false);
        this.$store.commit("canAddrLoad", []);
      } else {
        this.$notify.error({
          title: "Error",
          message: err.msg
        });
      }
    }
  }
};
</script>
<style>
.el-row {
  margin-bottom: 10px;
}
.header {
  font-weight: 800;
}
.title {
  line-height: 18px;
  font-weight: 550;
}
</style>
