<template>
  <div>
    <div>
      <el-radio-group v-model="type" class="choose" @change="itemChange1">
        <el-radio-button label="uds">UDS</el-radio-button>
        <!-- <el-radio-button label="doip" v-if="mode==='doip'">DOIP</el-radio-button> -->
      </el-radio-group>
    </div>
    <div class="subheader">Serivce</div>
    <el-select
      v-model="itemIndex"
      placeholder="请选择"
      class="choose"
      @change="itemChange"
      style="width:100%"
    >
      <el-option v-for="(item,key) in config[type]" :key="key" :label="item.name" :value="key">
        <span style="float: left">{{ item.name }}</span>
        <span style="float: right; color: #8492a6; font-size: 13px">0x{{ item.value.toString(16) }}</span>
      </el-option>
    </el-select>
    <div class="subheader">Address</div>
    <el-select v-model="addrIndex" placeholder="请选择" class="choose" style="width:100%">
      <el-option v-for="(item,key) in addrTable" :key="key" :label="item.name" :value="key">
        <span style="float: left">{{ item.name }}</span>
        <span style="float: right; color: #8492a6; font-size: 13px">SA:{{ item.SA}},TA:{{item.TA}}</span>
      </el-option>
    </el-select>

    <Service :config="config[type][itemIndex]" @additem="addItem" v-if="refresh" />
  </div>
</template>

<script>
import config from "./service.js";
import Service from "./service.vue";
export default {
  components: {
    Service
  },
  data() {
    return {
      config: config,
      type: "uds",
      itemIndex: 0,
      refresh: true,
      addrIndex: ""
    };
  },
  props: ["mode"],
  computed: {
    addrTable: function() {
      if (this.mode === "doip") {
        return this.$store.state.doipAddrTable;
      } else if (this.mode === "can") {
        return this.$store.state.canAddrTable;
      } else {
        return [];
      }
    }
  },
  methods: {
    addItem(val) {
      //console.log(val)
      var item = JSON.parse(JSON.stringify(val));
      item.addr = this.addrTable[this.addrIndex];
      if (!this.addrTable[this.addrIndex]) {
        this.$message.error("请选择正确的地址");
        this.addrIndex = "";
        return;
      }
      if (this.mode === "doip") {
        this.$store.commit("doipTableAdd", item);
      } else if (this.mode === "can") {
        this.$store.commit("canTableAdd", item);
      }
      this.$emit("additem");
    },
    itemChange1() {
      this.itemIndex = "";
      this.itemChange();
    },
    itemChange() {
      this.refresh = false;
      this.$nextTick(() => {
        this.refresh = true;
      });
    }
  }
};
</script>

<style>
.choose {
  margin-bottom: 20px;
}
.subheader {
  font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
  font-size: 25px;
  margin-bottom: 10px;
}
</style>