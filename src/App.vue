<script setup>
import {ref,reactive,watch} from 'vue'
localStorage.setItem('deepseek_api_key', 'sk-bfwtleampwpipgmxtulkpkrmwfxkdlyzivnfgypswmvbotiq') 
const messages=reactive([])
const usermessage=ref('')
const isListening = ref(false)
let recognition = null
const onfocus=function(e){
  e.target.style.outline='none'
}
// 自动朗读开关（默认开启，可从 localStorage 读取用户偏好）
const autoSpeak = ref(localStorage.getItem('autoSpeak') !== 'false')

// 监听变化，保存到 localStorage
watch(autoSpeak, (val) => {
  localStorage.setItem('autoSpeak', val)
})
function speakText(text) {
  if (!autoSpeak.value) return
  if (!window.speechSynthesis) return
  window.speechSynthesis.cancel()  // 避免重叠
  const utterance = new SpeechSynthesisUtterance(text)
  utterance.lang = 'zh-CN'
  utterance.rate = 1.0
  window.speechSynthesis.speak(utterance)
}
const btn=()=>{
  autoSpeak.value=!autoSpeak.value
}
const apiKey = ref(localStorage.getItem('deepseek_api_key') || '')
const conversationHistory = reactive([])   // 存储 {role, content}
const isLoading = ref(false)
// 2. 调用 DeepSeek 接口
async function callDeepSeekAPI(userMessage) {
  if (!apiKey.value) {
    messages.push({role:'assistant', content:'请先设置 API Key'})
    return null
  }

  // 构建消息历史（保留最近 6 轮对话，避免超长）
  const messagesForAPI = []
  const start = Math.max(0, conversationHistory.length - 6)
  for (let i = start; i < conversationHistory.length; i++) {
    messagesForAPI.push(conversationHistory[i])
  }
  messagesForAPI.push({ role: 'user', content: userMessage })

  try {
    const response = await fetch('https://api.siliconflow.cn/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey.value}`
      },
      body: JSON.stringify({
        model: 'deepseek-ai/DeepSeek-V3.2',
        messages: messagesForAPI,
        temperature: 0.7,
        max_tokens: 800,
        stream: false
      })
    })

    if (!response.ok) {
      const errorData = await response.json()
      throw new Error(errorData.error?.message || `HTTP ${response.status}`)
    }

    const data = await response.json()
    const assistantReply = data.choices[0].message.content

    // 保存对话历史
    conversationHistory.push(
      { role: 'user', content: userMessage },
      { role: 'assistant', content: assistantReply }
    )
    // 限制历史长度（最多 20 条）
    if (conversationHistory.length > 20) conversationHistory.splice(0, conversationHistory.length - 20)

    return assistantReply
  } catch (error) {
    console.error('API 错误:', error)
    return `❌ 请求失败：${error.message}`
  }
}
 const submit=async ()=>{
  if (!usermessage.value.trim()) return
  const userMsg = usermessage.value
  // 显示用户消息
  messages.push({ role: 'user', content: userMsg })
  usermessage.value = ''
  
  // 显示加载状态（可选）
  isLoading.value = true
  const reply = await callDeepSeekAPI(userMsg)
  isLoading.value = false

  if (reply) {
    messages.push({ role: 'assistant', content: reply })
    // 自动语音输出（如果开启）
    if (autoSpeak.value) speakText(reply)
  }
}

function startVoiceInput() {
  // 如果已经在监听中，直接返回
  if (isListening.value) {
    console.log('语音识别已启动，请勿重复点击')
    return
  }
  
  // 检查浏览器支持
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
  if (!SpeechRecognition) {
    alert('当前浏览器不支持语音识别，请使用 Chrome 或 Edge')
    return
  }
  
  // 如果已有实例且正在运行，先中止（防止残留）
  if (recognition) {
    try {
      recognition.abort()
    } catch (e) {}
  }
  
  // 创建新实例
  recognition = new SpeechRecognition()
  recognition.lang = 'zh-CN'
  recognition.continuous = false
  recognition.interimResults = false
  
  // 事件绑定
  recognition.onstart = () => {
    isListening.value = true
    // 可选：改变按钮样式
  }
  
  recognition.onend = () => {
    isListening.value = false
  }
  
  recognition.onresult = (event) => {
    const text = event.results[0][0].transcript
    usermessage.value = text
    submit()  // 自动发送消息
    isListening.value = false
  }
  
  recognition.onerror = (event) => {
    console.error('语音错误', event.error)
    isListening.value = false
    if (event.error === 'network') {
      alert('网络问题导致语音识别失败，请检查网络后重试')
    } else if (event.error !== 'no-speech') {
      alert(`语音识别失败：${event.error}`)
    }
  }
  
  // 启动识别
  recognition.start()
}

</script>

<template>
  <div class="box">
    <!-- 左侧栏 -->
    <div class="list">
      <div class="name">
        <img src="../public/head.png" alt="">豆包
      </div>
      <div class="new">
        <div class="text"><span class="iconfont icon-06"></span>新对话</div>
        <div class="key">Ctrl K</div>
      </div>
      <div class="history">
        <div class="historytitle">历史对话</div>
        <div class="historylist"></div>
      </div>
      <div class="user">
        <img src="../public/user.png" alt="">橙留湘<span>></span>
      </div>
    </div>
    <!-- 右边聊天框 -->
    <div class="content">
      <div class="contenthead">
        <div class="func1">
          <span class="iconfont icon-chuanghu-"></span><span class="iconfont icon-06"></span>
        </div>
        <div class="contenttitle">
          <p class="p1">新对话</p>
          <p class="p2">内容由豆包 AI 生成，请仔细甄别</p>
        </div>
        <div class="func2">
          <span class="iconfont icon-dianhua"></span><span class="iconfont icon-xuanfuchuangkou"></span>
        </div>
      </div>
      <div class="contentbody" >
        <div v-show="messages.length===0" class="help">有什么我能帮你的吗？</div>
        <div v-show="messages.length!==0" v-for="item in messages">
          <span v-show="item.role==='user'" class="msg1">{{ item.content }}</span>
          <span v-show="item.role==='assistant'" class="msg2">{{ item.content }}</span>
        </div>
      </div>
      <div class="input">
      <input class="questions" type="text" placeholder="发消息..." @focus="onfocus" v-model="usermessage" @keyup.enter="submit">
      <div v-show="usermessage===''" class="voice1" @click="startVoiceInput">
        <span class="iconfont icon-maikefeng"></span>
      </div>
      <div v-show="usermessage!==''" class="voice2" @click="submit">
        <span class="iconfont icon-xiangshangjiantou"></span>
      </div>
      </div>
      <div class="btn" @click="btn">朗读</div>
    </div>
  </div>
</template>

<style scoped>
@media(max-width:375px){
  .list{
    background-color: black;
  }
}
.box{
  display: flex;
  width: 100vw;
  height:100vh ;
}
.box .list{
  position: relative;
  padding: 15px;
  width: 250px;
  height: 100%;
  background-color: #f7f6f6;
  border: 1px solid #c5c4c4;
}
.box .content{
  flex:1;
  height: 100%;
  position: relative;
}
.list .name{
  width: 100%;
  
}
.list .name img{
  width: 30px;
  height: 30px;
  border-radius: 15px;
  margin-right: 10px;
  vertical-align: bottom;
}
.list .new{
  display: flex;
  justify-content: space-between;
  width: 100%;
  height: 35px;
  background-color: white;
  border-radius: 10px;
  margin-top: 15px;
  line-height: 35px;
  padding-left: 10px;
  padding-right: 10px;
  box-shadow: 2px 2px 4px rgb(152, 151, 151);
}
.list .new .text span{
  font-size: 15px;
  font-weight: 700;
  margin-right: 10px;
}
.list .new .key{
  color: #d1d0d0;
}
.list .history{
  margin-top: 20px;
  width: 100%;
  height: 452px;
}
.list .history .historytitle{
  color: #a09f9f;
  font-size: 12px;
}
.list .user{
  padding-left: 15px;
  position: absolute;
  left:0;
  bottom:0;
  width: 100%;
  height:65px;
  line-height: 65px;
  border:1px solid #c5c4c4;
}
.list .user img{
  width: 40px;
  height: 40px;
  border-radius: 20px;
  margin-right: 10px;
  vertical-align: middle;
}
.list .user span{
  margin-left: 5px;
  color: #9f9e9e;
}
.content .contenthead{
  display: flex;
  justify-content: space-between;
  width: 100%;
  height: 60px;
  border:1px solid #c5c4c4;
  position: absolute;
  top:0;
  left:0;
  z-index:2;
  background-color: white;
}
.content .contenthead .func1{
  width: 10%;
  height: 100%;
  line-height: 60px;
  text-align: center;
}
.content .contenthead .func1 span:first-child{
  font-size: 35px;
  margin-right: 15px;
  vertical-align: bottom;
}
.content .contenthead .func1 span:last-child{
  font-size: 20px;
  font-weight: 700;
}
.content .contenthead .contenttitle{
  padding-top: 10px;
  width: 20%;
  height: 100%;
  text-align: center;
}
.content .contenthead .func2{
  width: 10%;
  height: 100%;
  line-height: 60px;
  text-align: center;
}
.content .contenthead .func2 span{
  font-size: 27px;
  margin-right: 15px;
}
.content .contenthead .contenttitle .p2{
  font-size: 12px;
  color: #b7b6b6;
  margin-top: 5px;
}
.content .contentbody{
  width: 64%;
  height: 100%;
  /* background-color: pink; */
  position: absolute;
  bottom:80px;
  left:18%;
  padding-top: 150px;
  overflow:auto;
  text-align: right;
}
.content .contentbody .help{
  width: 100%;
  font-size: 35px;
  font-weight: 700;
  margin-top: 20%;
  text-align: center;
}
.content .input{
  display: flex;
  justify-content:space-between;
  width: 64%;
  height: 100px;
  position: absolute;
  bottom:10px;
  left:18%;
  border:1px solid rgb(162, 162, 249);
  border-radius: 30px;
  box-shadow: 2px 2px 10px rgb(162, 162, 249);
  padding-left: 15px;
  padding-right: 15px;
  background-color: #fff;
}
.content .input .questions{
  flex:1;
  height: 40px;
  margin-top: 30px;
  border:none;
  font-size: 18px;
}
.content .input .voice1,.content .input .voice2{
  width: 50px;
  height: 50px;
  margin-top: 25px;
  margin-left: 20px;
  line-height: 50px;
  text-align: center;
  border-radius: 50px;
}
.content .input .voice1{
  background-color: #c2c1c1;
}
.content .input .voice2{
  background-color: #7489f5;
}
.content .input .voice1 .iconfont,.content .input .voice2 .iconfont{
  font-size: 30px;
}
.content .contentbody .msg1,.content .contentbody .msg2{
  display:inline-block;
  background-color: #e8e7e7;
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 30px;
}
.content .contentbody .msg2{
  text-align: left;
}
.btn{
  width: 70px;
  height: 40px;
  background-color: rgb(143, 158, 245);
  position: absolute;
  bottom:10px;
  right:120px;
  border-radius: 20px;
  line-height: 40px;
  text-align: center;
  color: #fff;
  cursor:pointer;
}
</style>
