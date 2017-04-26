<template>
<section class="index">
  <article id="status-box" :class="buttonState === 'offline' ? 'offline' : ''">
    <span id="status">{{ statusBox.status }}</span>
    <span id="station">{{ statusBox.station }}</span>
  </article>
  <article id="message-box">
    <article id="name">{{ messageBox.name }}</article>
    <article id="message">{{ messageBox.msg }}</article>
  </article>
  <article id="button-box">
    <div id="authenticate-button" class="button-set">
      <div class="button-cell">
        <button id="accept"
          :disabled="buttonState !== 'authenticated'"
          @click="acceptListener">確認驗證</button>
      </div>
      <div class="button-cell">
        <button id="reject"
          :disabled="buttonState !== 'authenticated'"
          @click="rejectListener">終止驗證</button>
      </div>
    </div>
    <div id="clear-button" class="button-set">
      <div class="button-cell">
        <button id="dismiss"
          :disabled="buttonState !== 'default'"
          @click="dismissListener">清除訊息</button>
      </div>
    </div>
  </article>
</section>
</template>

<script>
import io from 'socket.io-client'

require('../assets/l10n.min.js')
require('../assets/localisation.js')

const $  = s => document.querySelector(s)
const $$ = s => document.querySelectorAll(s)

const l  = str => {
  if (str)
    return str.toLocaleString('zh_tw')
  else
    return str
}

let timeout = null

const socket = io(document.URL)

export default {
  name: 'index',
  data () {
    return {
      messageBox: {
        name: '',
        msg:  ''
      },
      statusBox: {
        station: '',
        status:  l('offline')
      },
      buttonState: 'default'
    }
  },
  created() {
    const app = this

    socket.on('authenticated', data => {
      app.messageBox.name = data.id
      app.messageBox.msg  = `${data.type} ${data.department}`
      app.buttonState     = 'authenticated'
    })

    socket.on('confirmed', data => {
      app.messageBox.name = data.student_id
      app.messageBox.msg  = `${l('please go to station')} ${data.slot}`
      app.buttonState     = 'default'
    })

    socket.on('message', data => {
      app.messageBox.name = ''
      app.messageBox.msg  = l(data)
      app.buttonState     = 'default'

      if (timeout !== null) {
        clearTimeout(timeout)
      }
      timeout = setTimeout(app.cleanMessage, 5000)
    })

    socket.on('station', data => {
      app.statusBox.station = data
    })

    // Connection ON
    socket.on('connect', () => {
      app.statusBox.status = l('online')
      app.buttonState      = 'default'
    })
    socket.on('reconnect', () => {
      app.statusBox.status = l('online')
      app.buttonState      = 'default'
    })

    // Connection OFF
    socket.on('disconnect', () => {
      app.statusBox.status = l('offline')
      app.buttonState      = 'offline'
      app.cleanMessage()
    })
  },
  methods: {
    acceptListener(ev) {
      this.buttonState = 'default'
      socket.emit('accept')
    },
    rejectListener(ev) {
      this.buttonState = 'default'
      socket.emit('reject')
    },
    dismissListener(ev) {
      const app = this
      app.buttonState = 'default'
      app.cleanMessage()
      clearTimeout(timeout)
    },
    cleanMessage() {
      const app = this
      app.messageBox.name = ''
      app.messageBox.msg  = ''
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  color: inherit;
  font-family: Helvetica, 'Source Han Sans', 'Microsoft JhengHei', sans-serif;
  font-size: 24px;
  line-height: 1;
  text-align: center;
}

body {
  color: #434A54;
  margin: 0 auto;
  max-width: 36em;
  min-width: 24em;
  position: relative;
  width: 100%;
}

.button-set {
  display: table;
  width: 100%;
}

.button-cell {
  display: table-cell;
  padding: 0.25em;
}

button {
  border: none;
  border-radius: 5px;
  color: #FFF;
  padding: 0.5em 0.75em;
  width: 100%;
}

button#accept {
  background: #8CC152;
}

button#accept.active {
  background: #A0D468;
}

button#reject {
  background: #DA4453;
}

button#reject.active {
  background: #ED5565;
}

button#dismiss {
  background: #3BAFDA;
}

button#dismiss.active {
  background: #4FC1E9;
}

button[disabled] {
  background: #BBB !important;
}

article {
  padding: 1em;
}

article#status-box {
  border-bottom: #DDD 1px solid;
}

article#status-box.offline {
  color: #DA4453;
}

article#message-box {
  height: 8em;
}
</style>