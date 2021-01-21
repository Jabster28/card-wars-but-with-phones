<template>
  <div class="container">
    <div v-if="!choose">
      <Logo />
      <h1 class="title">card-wars-but-with-phones</h1>
      <div class="links">
        <button :disabled="choose" class="button--green" @click="client">
          client
        </button>
        <button :disabled="choose" class="button--green" @click="server">
          server
        </button>
      </div>
    </div>
    <div v-else-if="status == 'server'">
      server? based.

      <h1>{{ code ? 'code is ' + code : '' }}</h1>
      <textarea
        id="chat"
        v-model="chat"
        name="chat"
        cols="30"
        rows="10"
        readonly
      ></textarea>
    </div>
    <div v-else>
      client? cringe.
      <form v-if="!connected" @submit="cringe">
        <input
          v-model="code"
          type="number"
          :disabled="req"
          placeholder="enter server code here"
        />
      </form>
      <div v-else>
        gamers. we have done it.<br />
        <textarea
          id="chat"
          v-model="chat"
          name="chat"
          cols="30"
          rows="10"
          readonly
        ></textarea>
        <form @submit="sendMsg">
          <input v-model="msg" type="text" placeholder="send a msg" />
        </form>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import socket from 'socket.io-client'
export type ConnectionRequest = {
  code: string
  id: string
}
export default Vue.extend({
  data() {
    const x: {
      io: socket.Socket | undefined
      choose: boolean
      status: 'client' | 'server' | null
      code: number
      req: boolean
      connected: boolean
      chat: string
      msg: string
    } = {
      chat: '',
      msg: '',
      io: undefined,
      choose: false,
      req: false,
      status: null,
      code: 0,
      connected: false,
    }
    return x
  },
  mounted() {
    const io = socket.io('https://well-nifty-game.glitch.me/')
    this.io = io
  },
  methods: {
    sendMsg(event: Event) {
      event.preventDefault()
      this.io?.emit('chat', this.msg)
      this.msg = ''
    },
    cringe(event: Event) {
      this.req = true
      this.code = +this.code
      event.preventDefault()
      console.log('choosing')
      this.io?.emit('choose', this.code)
      console.log('registering connection thing')
      this.io?.on('connectCon', (e: ConnectionRequest) => {
        console.log("something's been confirmed")
        console.log(e)
        if (+e.code === this.code) {
          console.log('accepting cause same code')
          this.connected = true
          this.io?.on('chat', (e: string) => {
            this.chat = this.chat + '\n' + e
          })
        }
      })
    },
    client() {
      if (this.choose) return
      console.log('client')
      this.io?.emit('client')
      this.choose = true
      this.status = 'client'
    },
    server() {
      if (this.choose) return
      console.log('server')
      this.io?.emit('server')
      this.choose = true
      this.io?.on('connectRequest', (e: ConnectionRequest) => {
        console.log(e)
        if (+e.code === this.code) {
          console.log('confirming because same code')
          this.io?.emit('connectCon', e)
        }
      })

      this.io?.on('chat', (e: string) => {
        this.chat = this.chat + '\n' + e
      })
      this.io?.on('clientdc', (e: string) => console.log(e))
      this.io?.on('code', (e: number) => (this.code = e))
      this.status = 'server'
    },
  },
})
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
