<template>
<div>
<section class="index">
  <article id="status-box" :class="status">
    <span id="status">{{ status_message }}</span>
    <span id="station">{{ store.station.name }}</span>
  </article>
  <article id="info-box">
    <article id="title">{{ info.title }}</article>
    <article id="message">{{ info.message }}</article>
  </article>
  <article id="button-box">
    <div id="authenticate-button" class="button-set">
      <div class="button-cell">
        <button id="accept"
          :disabled="status !== 'authenticated'"
          @click="acceptListener">確認驗證</button>
      </div>
      <div class="button-cell">
        <button id="reject"
          :disabled="status !== 'authenticated'"
          @click="rejectListener">終止驗證</button>
      </div>
    </div>
    <div id="clear-button" class="button-set">
      <div class="button-cell">
        <button id="dismiss"
          :disabled="status !== 'online'"
          @click="dismissListener">清除訊息</button>
      </div>
    </div>
  </article>
  <article id="station-box">
      <div class="tablet" 
          v-for="tablet in store.tablets"
          v-bind:class="tablet.status">
         {{ tablet.slot }}
      </div>
  </article>
</section>
<article id="station-box" v-for="(tablet, idx) in store.tablets">
  <div :id="`station${idx + 1}`" :class="`station ${tablet}`">{{ idx + 1 }}</div>
</article>
</div>
</template>

<script>
import _  from 'lodash'
import io from 'socket.io-client'

require('../assets/l10n.min.js')
require('../assets/localisation.js')

const l  = str => {
  if (str) return str.toLocaleString('zh_tw')
  else     return str
}

const socket = io(document.URL)

const code = {
  online:        'online',
  offline:       'offline',
  authenticated: 'authenticated',
  confirmed:     'confirmed',
  message:       'message',
  clean:         'clean'
}

let timeout = null

export default {
  name: 'index',
  /**
   *  status: 'online', 'authenticated', 'offline'
   *  store.msg_status: 'message', 'authenticated', 'confirmed', 'clean'
   *  tablets: {@id, @status}
   */
  data () {
    return {
      status: code.offline,
      store: {
        msg_status: code.clean,
        student: {
          id: '',
          type: '',
          department: ''
        },
        station: {
          id:   '',
          name: ''
        },
        tablets: [
          { id: 1, status: '' }
        ],
        slot:      '',
        error_msg: ''
      },
      info: {
        title:   '',
        message: ''
      }
    }
  },
  created() {
    const app = this
    /**
     *  connection:
     *    connect, reconnect, and disconnect
     */
    // Connection ON
    socket.on('connect',    () => { app.status = code.online })
    socket.on('reconnect',  () => { app.status = code.online })
    // Connection OFF
    socket.on('disconnect', () => {
      app.status = code.offline
      app.clean_info()
    })
    /**
     *  initialise
     */
    socket.on('initialise', data => {
      if (app.status === code.offline)
        app.status = code.online
      _.merge(app.store, data)
    })
    /**
     *  update
     */
    socket.on('update', data => {
      if (app.status === code.offline)
        app.status = code.online
      _.merge(app.store, data)
    })
    /**
     *  authenticated
     */
    socket.on('authenticated', data => {
      app.status = code.authenticated
      _.merge(app.store, data)
    })
  },
  computed: {
    status_message() {
      return l(this.status)
    },
    info() {
      const app = this
      switch (app.store.msg_status) {
        // print error message
        case code.message:
          if (timeout !== null)
            clearTimeout(timeout)
          timeout = setTimeout(app.clean_info, 5000)
          return {
            title: '',
            message: l(app.store.error_msg)
          }
        // show information of authenticated student
        case code.authenticated:
          const student = app.store.student
          return {
            title: student.id,
            message: `${student.type} ${student.department}`
          }
        // confirm student and show slot
        case code.confirmed:
          return {
            title: app.store.student.id,
            message: `${l('please go to station')} ${app.store.slot}`
          }
        // clean message
        default:
          return {
            title:   '',
            message: ''
          }
      }
    }
  },
  methods: {
    acceptListener(ev) {
      socket.emit('accept')
    },
    rejectListener(ev) {
      socket.emit('reject')
    },
    dismissListener(ev) {
      this.clean_info()
      clearTimeout(timeout)
    },
    clean_info() {
      this.msg_status = code.clean
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
article#info-box {
  height: 8em;
}
article#station-box {
    height: 4em
}

.tablet {
background-color: #BBB;
height: 3em;
width: 3em;
margin: 1em;
display: inline-block;
text-align: center;
}
.tablet.free {
background-color: #A0D468;
}
.tablet.lock {
background-color: #FFD95F;
}
</style>
