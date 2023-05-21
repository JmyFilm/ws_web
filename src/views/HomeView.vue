<script setup>
import { nextTick, onMounted, reactive, ref } from 'vue'
import { NButton, NLayout, NInput, NInputGroup, useMessage, NTag } from 'naive-ui'
const s = reactive({
  socket: null,
  msg: [],
  sv: "",
  onlien: 0,
})
const nmsg = useMessage()
function send() {
  if (s.sv.trim() == '') {
    nmsg.warning("消息不能为空")
    return
  }

  s.socket.send(s.sv);
  s.msg.push({
    type: "self",
    name: "",
    msg: s.sv,
    time: new Date().toLocaleString()
  })
  s.sv = ""
  toBottom()
}


const ip = ref(null)
const tag = ref(null)

function addMsg(msg) {
  var ret = {type:"", name: "", msg: "", time: ""}

  if (msg.charAt(0) == ">") {
    let msgArr = msg.split("|")
    s.online = msgArr[3]
    ret.type = msgArr[1]
    ret.name = msgArr[2]
    ret.time = msgArr[4]
  } else {
    var i = msg.indexOf("|")
    ret.type = "msg"
    ret.name = msg.substr(0,i)
    let t = msg.substring(i+1)
    i = t.indexOf("|")
    ret.time = t.substr(0,i)
    ret.msg = t.substring(i+1)
  }

  s.msg.push(ret)

  const rect1 = tag.value.getBoundingClientRect();
  const rect2 = ip.value.getBoundingClientRect();
  const distance = Math.abs(rect1.top - rect2.top);
  if (distance < 100) {
    toBottom()
  }  
}

const scrollDiv = ref(null)

function toBottom(){
  nextTick(() => {
    scrollDiv.value.lastElementChild.scrollIntoView()
  });
}

onMounted(() => {
  s.socket = new WebSocket("wss://wss.jmyfilm.com");

  s.socket.addEventListener("message", ({ data }) => {
    addMsg(data)
  })
})


</script>

<template>
  <n-layout>
    <main>
      <div id="chatBox" ref="scrollDiv" class="msg">
        <section v-for="msg in s.msg">
          <!-- {{ msg }} -->
          <n-tag style="margin-top: 1rem;" class="tag" v-if="msg.type == 'online'" round  type="success">
             服务器已连接 {{ msg.time.substr(11) }}
          </n-tag>
          <n-tag class="tag" v-if="msg.type == 'login'" round  type="info">
             {{ msg.name }} 建立连接  {{ msg.time.substr(11) }}
          </n-tag>
          <n-tag class="tag" v-if="msg.type == 'logout'" round  type="warning">
             {{ msg.name }} 断开连接 {{ msg.time.substr(11) }}
          </n-tag>
          <p class="rec" v-if="msg.type=='msg'">
            <span class="name">{{ msg.name }}</span>
            <p class="data">
              <div class="content">
                <span> {{ msg.msg }}</span>
                <time class="ct">{{ msg.time.substr(11,5) }}</time>
              </div>
            </p>
          </p>
          <p class="rec self" v-if="msg.type=='self'">
            <p class="data">
              <div class="content">
                <span> {{ msg.msg }}</span>
                <time class="ct">{{ msg.time.substr(10,5) }}</time>
              </div>
            </p>
          </p>
        </section>
        <span ref="tag"></span>
      </div>
      <div class="b"></div>
      <div ref="ip" class="ip">
        <n-input-group class="ip_ng"  @keyup.enter="send">
          <n-input v-model:value="s.sv"/>
          <n-button @click="send" :disabled="s.sv.trim() == '' ? true : false">
            发送
          </n-button>
        </n-input-group>
      </div>

    </main>

  </n-layout>
</template>

<style scoped>
main {
  max-width: 35rem;
  margin: 0 auto;
  padding: 0.5rem 1rem;
}
.rec {
  text-align: left;
  margin: 0;
  padding: 0;
  margin-bottom: 0.87rem;

}
.rec span {
  display: block;
}

.rec .name {
  font-size: 13px;
}
.self {
  display: flex;
  -webkit-box-orient: horizontal;
  -webkit-box-direction: reverse;
  -ms-flex-direction: row-reverse;
  flex-direction: row-reverse;
  margin-bottom: 1rem;
}
.data {
  color: #000;
  max-width: 87%;
  font-size: 16px;
  display: flex;
  margin: 0;
  padding: 0;
}
.self .data {
  color: #fff;
}
.content {
  border: 1px solid #F2F3F5;
  background-color: #F2F3F5;
  padding: 6px 8px;
  padding-bottom: 0;
  border-radius: 5px
}
.self .content {
  border: 1px solid #4A97E7;
  background-color: #4A97E7;
}
.content span {
  min-width: 3rem;
  word-break:break-all;
  line-height: 24px;
  letter-spacing: 0.5px;
}
.ct {
  margin-top: 4px;
  margin-bottom: 2px;
  float: right;
  font-size: 10px;
  text-align: right;
}

.tag {
  margin-bottom: 1rem;
}
section {
  text-align: center;
  padding-bottom: 1px;
}
.msg {
  padding-bottom: 4rem;
  overflow:auto
}
.ip {
  background-color: #FFFFFF;
  height: 3.5rem;
  padding-top: 0.7rem;
  position: absolute;
  bottom: 0rem;
  width: 35rem;
}

@media screen and (max-width: 35rem) {
  .ip {
    width: auto;
    right: 0.5rem;
    left: 0.5rem;
  }
}

@media (prefers-color-scheme: dark) {
  .ip {
    background-color: #101014;
  }
  .data {
    color: #eee;
  }
  .content {
    border: 1px solid #28282C;
    background-color: #28282C;
  }
}
</style>