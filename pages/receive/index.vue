<template>
  <div class="page">
    <el-row>
      <el-col>
        <el-card class="card" v-loading="loading">
          <h2>Wallet:</h2>
          <div class="DialogReceive">
            <div class="key">
              <div class="qrcode">
                <no-ssr>
                <vue-qr 
                  :size="250" 
                  :dotScale="0.5" 
                  :logoSrc="zee" 
                  :margin="10" 
                  :text="user_profile.wallet">
                </vue-qr>
                </no-ssr>
              </div>
              <span>{{ user_profile.wallet }}</span>
            </div>
          </div>
          <input type="text" id="myInput" v-model="user_profile.wallet" />
          <el-button
            v-if="user_profile.wallet !== null"
            type="success"
            plain
            style="margin-top:1rem;"
            @click="onCopy()"
          >Copy to clipboard</el-button>
          <nuxt-link to="/getwallet" v-else>
            <el-button type="success" plain style="margin-top:1rem">Get Wallet</el-button>
          </nuxt-link>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';
import Cookie from 'js-cookie';

export default {
  middleware: ["auth"],
  data() {
    return {
      zee: require("~/assets/KE.png"),
      loading: true,
    };
  },
  asyncData({req, res, error, redirect}) {
    let token;
    if (process.server) {
      const jwtCookie = req.headers.cookie
        .split(";")
        .find(c => c.trim().startsWith("jwt="));
      if (!jwtCookie) {
        return;
      }
      token = jwtCookie.split("=")[1];
    }
    if (process.client) {
      token = Cookie.get("jwt");
    }
    const config = {
      headers: {
        Authorization: "Bearer " + token
      }
    };
    return axios.get(process.env.apiUrl + "/userprofile", config)
      .then((res) => {
        return { user_profile: res.data, loading: false }
      })
      .catch((e) => {
        redirect({
          name: 'login'
        })
      })
  },
  methods: {
    onCopy() {
      /* Get the text field */
      var copyText = document.getElementById("myInput");
      /* Select the text field */
      copyText.select();
      copyText.setSelectionRange(0, 99999); /*For mobile devices*/
      /* Copy the text inside the text field */
      document.execCommand("copy");
      /* Alert the copied text */
      alert("Copied the text: " + copyText.value);
    }
  }
};
</script>

<style scoped>
h2,
h4 {
  color: #fafafa;
}
#myInput {
  position: absolute;
  color: #fff;
  opacity: 0;
  width: 1px;
  height: 1px;
  font-size: 1px;
  left: 0;
  box-sizing: content-box;
  -webkit-user-select: initial;
  outline: none;
  margin-left: -9999px;
}
.DialogReceive {
  text-align: center;
  padding: 1rem;
}
.key span {
  word-wrap: break-word;
  overflow-wrap: anywhere;
  background: red;
  border-radius: 2px;
}
.qrcode {
  padding: 1rem;
}
.el-button {
  margin: 1rem;
}
/* // RESPONSIVE */
/* //SmartPhone */
@media only screen and (max-width: 768px) {
  .key span {
    color: #fafafa !important;
  }
}
</style>