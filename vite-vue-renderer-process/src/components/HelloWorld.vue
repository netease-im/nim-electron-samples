<script setup lang="ts">
import { ref } from 'vue'
import { v2 } from 'node-nim'
import {
  V2NIMError,
  V2NIMMessage
} from 'node-nim/types/v2_def/v2_nim_struct_def'
import {
  V2NIMLoginStatus,
  V2NIMKickedOfflineReason,
  V2NIMConnectStatus,
  V2NIMDataSyncType,
  V2NIMDataSyncState
} from 'node-nim/types/v2_def/v2_nim_enum_def'

const appKey = ref('')
const accountId = ref('')
const accountPassword = ref('')
const receiverId = ref('')
const messageBody = ref('')

const handleInit = () => {
  const key = appKey.value
  if (!key) {
    alert('Please input your app key')
    return
  }
  const error = v2.init({ appkey: key })
  if (error) {
    alert('Failed to initialize NIM SDK: ' + error)
  } else {
    v2.loginService?.on('loginStatus', (status: V2NIMLoginStatus) => {
      console.log(`[nim] login status changed: ${status}`)
    })
    v2.loginService?.on('loginFailed', (error: V2NIMError) => {
      console.error(`[nim] login failed: ${JSON.stringify(error)}`)
    })
    v2.loginService?.on('kickedOffline', (details: V2NIMKickedOfflineReason) => {
      console.warn(`[nim] kicked offline: ${details}`)
    })
    v2.loginService?.on('connectStatus', (connectStatus: V2NIMConnectStatus) => {
      console.log(`[nim] connect status changed: ${connectStatus}`)
    })
    v2.loginService?.on('disconnected', (error: V2NIMError) => {
      console.warn(`[nim] disconnected: ${JSON.stringify(error)}`)
    })
    v2.loginService?.on('connectFailed', (error: V2NIMError) => {
      console.error(`[nim] connect failed: ${JSON.stringify(error)}`)
    })
    v2.loginService?.on('dataSync', (syncType: V2NIMDataSyncType, syncState: V2NIMDataSyncState, error: V2NIMError) => {
      console.log(`[nim] data sync: ${syncType}, ${syncState}, ${JSON.stringify(error)}`)
    })
    v2.messageService?.on('sendMessage', (message: V2NIMMessage) => {
      console.log(`[nim] send message listener: `, message)
    })
    console.info('[nim] Initialize NIM SDK successfully')
  }
}

const handleUnInit = () => {
  const error = v2.uninit()
  if (error) {
    alert('Failed to cleanup NIM SDK: ' + error)
  } else {
    console.info('[nim] Cleanup NIM SDK successfully')
  }
}

const handleLogin = async () => {
  const account = accountId.value
  const password = accountPassword.value
  if (!account || !password) {
    alert('Please input your account ID and password')
    return
  }
  try {
    await v2.loginService?.login(account, password, {})
  } catch (error) {
    alert('Failed to login: ' + error)
  }
}

const handleLogout = async () => {
  try {
    await v2.loginService?.logout()
  } catch (error) {
    alert('Failed to logout: ' + error)
  }
}

const handleSendMessage = async () => {
  try {
    const message: V2NIMMessage = v2.messageCreator?.createTextMessage(messageBody.value)
    const to = v2.conversationIdUtil?.p2pConversationId(receiverId.value)
    if (to) {
      const result = await v2.messageService?.sendMessage(message, to, {}, null)
      console.log('[nim] send message result: ', result)
    } else {
      alert('Invalid receiver ID: ' + receiverId.value)
    }
  } catch (error) {
    alert('Failed to send message: ' + error)
  }
}

</script>

<template>
  <h2>NetEase IM Electron-Vite Sample</h2>
  <h4>import node-nim under renderer process</h4>
  <div class="card">
    <input id="app-key" type="text" placeholder="Your app key" v-model="appKey">
    <button class="fixed-button" style="margin-right: 5px;" @click="handleInit">Init</button>
    <button class="fixed-button" @click="handleUnInit">UnInit</button>
    <input id="account-id" type="text" placeholder="Your account ID" v-model="accountId">
    <input id="account-pwd" type="password" placeholder="Your password" v-model="accountPassword">
    <button class="fixed-button" style="margin-right: 5px;" @click="handleLogin">Login</button>
    <button class="fixed-button" @click="handleLogout">Logout</button>
    <input id="conversation-id" type="text" placeholder="Receiver ID" v-model="receiverId">
    <input id="message-body" type="text" placeholder="Message body" v-model="messageBody">
    <button class="fixed-button" @click="handleSendMessage">Send</button>
  </div>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.fixed-button {
  width: 100px;
}
</style>
