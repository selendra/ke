<template>
  <div class="page">
    <div class="card" v-loading="loading">
      <h1>Please input PIN Code</h1>
      <el-form>
        <div class="pin_code">
          <h4>PIN Code</h4>
          <el-form-item>
            <VuePin v-model="pin" :onlyNumber="true" />
          </el-form-item>
          <h4>Confirm PIN</h4>
          <el-form-item>
            <VuePin v-model="pin1" />
          </el-form-item>
        </div>
      </el-form>
      <div class="respone">
        <span>{{ this.res_msg }}</span>
      </div>
      <div class="button">
        <el-button type="primary" @click="getWallet()">Get Wallet</el-button>
      </div>
      <el-dialog type="text" :visible.sync="dialogVisible" width="80%" :before-close="handleClose">
        <span slot="title" style="font-size:25px">
          <i class="el-icon-warning" style="color:orange"></i> Warning
        </span>
        <p class="warning">
          Please keep your key secure. This secret key will only
          be showed to you once.
          <br />KE will not be able to help you recover it if lost.
        </p>
        <br/>
        <div class="key">
          <h4 class="public_key">
            Public key (will be Wallet):
            <div style="padding-top: 0.6rem"></div>
          </h4>
          <span style="background-color:yellow; color:  #191819">{{ this.value.wallet }}</span>
        </div>
        <br />
        <div class="key">
          <h4 class="secret_key">
            Seed (SAVE THIS AND KEEP THIS SECURE):
            <div style="padding-top: 0.6rem"></div>
            <span
              style="background-color:yellow"
            >{{ this.value.seed }}</span>
          </h4>
        </div>
        <el-button type="danger" @click="handleClose()">Close</el-button>
        <el-button type="primary" @click="exportPDF()">Export PDF</el-button>
      </el-dialog>
    </div>
  </div>
</template>
 
<script>
import axios from 'axios';
import Cookie from 'js-cookie';
import VuePin from "@/components/VuePin";
import { mixinMsg } from '@/plugins/mixins/mixin_msg';

export default {
  middleware: ["auth"],
  mixins: [mixinMsg],
  components: {
    VuePin
  },
  data() {
    return {
      loading: false,
      pin: "",
      pin1: "",
      dialogVisible: false,
      value: [],
      res_msg: ""
    };
  },
  methods: {
    getWallet() {
      const pin = this.pin;
      const pin1 = this.pin1;
      if (pin === "") {
        this.res_msg = "PIN must be filled out!";
        return false;
      } else if (pin !== pin1) {
        this.res_msg = "PIN does not match!";
        return false;
      } else {
        this.loading = true;
        this.$store.dispatch('users/handleGetWallet', {
          pin: this.pin
        })
        .then(_=> {
          if(this.apiMsg !== 'Opp! look like you already had a wallet') {
            this.value = this.apiMsg;
            this.dialogVisible = true;
          } else {
            this.$notify({
              title: "Opp!",
              message: this.apiMsg,
              type: 'error'
            });
          }
          this.loading = false;
        })
      }
    },
    handleClose(done) {
      this.$confirm("Are you sure to close this dialog?")
        .then(_ => {
          this.dialogVisible = false;
          this.$router.push("/profile");
        })
        .catch(_ => {});
    },
    exportPDF() {
      if (process.client) {
        const jsPDF = require("jspdf");
        const doc = new jsPDF({
          orientation: "landscape"
        });
        doc.setFontSize(14);
        doc.text("Wallet: " + this.value.wallet, 10, 10);
        doc.text("Private Key: " + this.value.seed, 10, 20);
        doc.save("YourKey.pdf");
      }
    }
  }
};
</script>

<style lang="less" scoped>
h1,
h4 {
  color: white;
}
.card {
  text-align: center;
  padding: 2rem;
}
.pin_code {
  padding: 3rem 0 0 0;
}
.button {
  padding: 2rem 0 0 0;
}
.respone {
  padding: 2rem 0 0 0;
  span {
    color: red;
  }
}
// Dialog
.el-dialog p {
  font-size: 15px;
  color: red;
}
.el-dialog h4 {
  color: black;
}
.key {
  // border: 1px solid red;
  padding: 5px;
}
.key span {
  background-color:yellow;
  border-radius: 2px;
  color: #191819;
  font-size: 16px;
}
.el-dialog .el-button {
  margin: 1rem 0 0 0;
}
/** SmartPhone Tablet */
@media only screen and (max-width: 768px) {
  .warning {
    font-size: 16px!important;
  }
}
/* Normal */
@media only screen and (min-width: 769px) and (max-width: 1199px) {
  .warning {
    font-size: 18px!important;
  }
  .public_key,
  .secret_key {
    font-size: 16px!important;
  }
}
/* Large monitor */
@media only screen and (min-width: 1200px) and (max-width: 1919px) {
 .warning {
    font-size: 18px!important;
  }
  .public_key,
  .secret_key {
    font-size: 16px!important;
  }
}
/* Landscape */
@media only screen and (max-height: 500px) {
}
/* Widescreen */
@media only screen and (min-width: 1920px) {
  .warning {
    font-size: 18px!important;
  }
  .public_key,
  .secret_key {
    font-size: 16px!important;
  }
}
</style>