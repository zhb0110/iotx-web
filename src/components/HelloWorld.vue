<template>
  <div class="hello">
    <!--    <h1>{{ msg }}</h1>-->
    <!--    <p>-->
    <!--      For a guide and recipes on how to configure / customize this project,<br>-->
    <!--      check out the-->
    <!--      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.-->
    <!--    </p>-->
    <!--    <h3>Installed CLI Plugins</h3>-->
    <!--    <ul>-->
    <!--      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-babel" target="_blank" rel="noopener">babel</a></li>-->
    <!--      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint" target="_blank" rel="noopener">eslint</a></li>-->
    <!--    </ul>-->
    <!--    <h3>Essential Links</h3>-->
    <!--    <ul>-->
    <!--      <li><a href="https://vuejs.org" target="_blank" rel="noopener">Core Docs</a></li>-->
    <!--      <li><a href="https://forum.vuejs.org" target="_blank" rel="noopener">Forum</a></li>-->
    <!--      <li><a href="https://chat.vuejs.org" target="_blank" rel="noopener">Community Chat</a></li>-->
    <!--      <li><a href="https://twitter.com/vuejs" target="_blank" rel="noopener">Twitter</a></li>-->
    <!--      <li><a href="https://news.vuejs.org" target="_blank" rel="noopener">News</a></li>-->
    <!--    </ul>-->
    <!--    <h3>Ecosystem</h3>-->
    <!--    <ul>-->
    <!--      <li><a href="https://router.vuejs.org" target="_blank" rel="noopener">vue-router</a></li>-->
    <!--      <li><a href="https://vuex.vuejs.org" target="_blank" rel="noopener">vuex</a></li>-->
    <!--      <li><a href="https://github.com/vuejs/vue-devtools#vue-devtools" target="_blank" rel="noopener">vue-devtools</a></li>-->
    <!--      <li><a href="https://vue-loader.vuejs.org" target="_blank" rel="noopener">vue-loader</a></li>-->
    <!--      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank" rel="noopener">awesome-vue</a></li>-->


    <!--    </ul>-->
    <!--    <p>-->
    <!--    < to="/test">测试</>-->
    <!--    </p>-->

    <!--    <a href="/test">测试</a>|-->
    <!--    <a href="#/test">Home1</a> |-->

    <div>
      <input type="text" placeholder="请输入你想传输的内容" id="text" class="col-lg-12"/>
      <!--    <input type="button" value="连接" class="btn btn-info" id="connect" onclick="connect()"/>-->
      <input type="button" value="发送" class="btn btn-success" id="sent" @click="sent"/>
      <!--    <input type="button" value="断开" class="btn btn-danger" id="disconnect" disabled="disabled"-->
      <!--           onclick="disconnect()"/>-->
      <div>
        <!--    <input type="button" value="连接" class="btn btn-info" id="connect" onclick="connect()"/>-->
        <input type="button" value="获得用户" class="btn btn-success" id="getUser" @click="getUser"/>
        <input type="button" value="获得所有用户" class="btn btn-success" id="getUser" @click="getAllUser"/>
        <!--    <input type="button" value="断开" class="btn btn-danger" id="disconnect" disabled="disabled"-->
        <!--           onclick="disconnect()"/>-->
      </div>
    </div>

    <div id="log">
      <p>沟通记录:</p>
      <textarea id="chattingRecords" style="white-space: pre-wrap;width: 100%;height: 300px;" value="">
      </textarea>
    </div>
  </div>
</template>

<script>

import axios from 'axios'

import Test from './test.vue'

const routes = {
  '/test': Test,
}

// websocket连接对象
let websocket = null;
// TODO:心跳机制
// var heartCheck = {
//   timeout: 55000,        // 在接近断开的情况下以通信的方式去重置连接时间。超时55秒
//   timeoutObj: null,
//   serverTimeoutObj: null,
//   reset: function(){
//     clearTimeout(this.timeoutObj);
//     clearTimeout(this.serverTimeoutObj);
//     return this;
//   },
//   start: function(){
//     this.serverTimeoutObj = setInterval(function(){
//       if(websocket.readyState == 1){
//         console.log("连接状态，发送消息保持连接");
//         websocket.send("ping");
//         heartCheck.reset().start();    // 如果获取到消息，说明连接是正常的，重置心跳检测
//       }else{
//         console.log("断开状态，尝试重连");
//       }
//     }, this.timeout)
//   }
// }


export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      currentPath: window.location.hash,
      textareaValue: "test"
    }
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || '/']
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
      this.currentPath = window.location.hash
    })
  },

  created() {
    websocket = null
    if ('WebSocket' in window) {
      var protocol = window.location.protocol === 'http:' ? 'ws://' : 'wss://'
      websocket = new WebSocket(protocol + 'localhost:8090/ws/1')
    } else {
      alert('该浏览器不支持WebSocket')
    }

    websocket.onopen = function (event) {
      // heartCheck.reset().start();
      console.log('建立WebSocket连接', event)
    }

    websocket.onclose = function (event) {
      console.log('断开WebSocket连接', event)
    }

    websocket.onmessage = function (event) {
      console.log('收到设备消息' + event.data)
      let text = document.querySelector('#chattingRecords');
      if (text.value === '') {
        text.value = '收到设备消息：' + event.data;
      } else {
        text.value = text.value + "\n" + '收到设备消息：' + event.data;
      }
      // this.textareaValue = this.textareaValue + "<br/>" + event.data;

    }

    websocket.onerror = function (event) {
      console.log('websocket通信发生错误', event)
    }

    window.onbeforeunload = function (event) {
      console.log('页面关闭前', event)
      websocket.close()
    }
  },
  methods: {
    // handleClickOutside() {
    //   this.$store.dispatch('app/closeSideBar', {withoutAnimation: false})
    // },
    sent() {
      // console.log('测试')
      if (websocket != null) {
        let text = document.querySelector('#text');
        // TODO:发送消息给服务端；
        websocket.send(text.value);
        console.log('客户端说：' + text.value)
        // log('客户端说：' + text.value);

        // 下发控制设备命令-get方式
        // axios.get('http://localhost:8090/device/rpc_get?controlValue=' + text.value).then(res => {
        //   console.log("指令下发成功", res);
        // })
        // 下发控制设备命令-post方式
        // axios.post('http://localhost:8090/device/rpc_post', {"controlValue": text.value}, {
        //   headers: {
        //     'Content-Type': 'application/json'
        //   }
        // }).then(res => {
        //   console.log("指令下发成功", res);
        // }).catch(error => {
        //   if (error.response) {
        //     // 请求已发出，但服务器响应的状态码不在2xx范围内
        //     console.log(error.response.data);
        //     console.log(error.response.status);
        //     console.log(error.response.headers);
        //   } else if (error.request) {
        //     // 发出了请求，但没有收到响应
        //     console.log(error.request);
        //   } else {
        //     // 在设置请求过程中发生了错误
        //     console.log('Error', error.message);
        //   }
        //   console.log(error.config);
        // });
        // 下发控制设备命令-post方式+本地ems
        axios.post('http://localhost:8090/device/rpc_post_ems', {"controlValue": text.value}, {
          headers: {
            'Content-Type': 'application/json'
          }
        }).then(res => {
          console.log("指令下发成功", res);
          let text = document.querySelector('#chattingRecords');
          if (text.value === '') {
            text.value = 'rpc控制状态：' + res.status;
          } else {
            text.value = text.value + "\n" + 'rpc控制状态：' + res.status;
          }
        }).catch(error => {
          if (error.response) {
            // 请求已发出，但服务器响应的状态码不在2xx范围内
            console.log(error.response.data);
            console.log(error.response.status);
            console.log(error.response.headers);
          } else if (error.request) {
            // 发出了请求，但没有收到响应
            console.log(error.request);
          } else {
            // 在设置请求过程中发生了错误
            console.log('Error', error.message);
          }
          console.log(error.config);
        });


      } else {
        console.log('请先建立连接！')
        // log('请先建立连接！')
      }
    },
    getUser() {
      axios.get('http://localhost:8090/user/getUser?id=1').then(res => {
        console.log("查询用户", res);
        let text = document.querySelector('#chattingRecords');
        if (text.value === '') {
          text.value = '获得用户信息：' + JSON.stringify(res.data);
        } else {
          text.value = text.value + "\n" + '获得用户信息：' + JSON.stringify(res.data);
        }
      })
    },
    getAllUser() {
      axios.get('http://localhost:8090/user/getUser').then(res => {
        console.log("查询所有用户", res);
        let text = document.querySelector('#chattingRecords');
        if (text.value === '') {
          text.value = '获得用户信息：' + JSON.stringify(res.data);
        } else {
          text.value = text.value + "\n" + '获得用户信息：' + JSON.stringify(res.data);
        }
      })
    }

  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
