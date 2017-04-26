<template>
<section class="index">
  <article id="status-box">
    <span id="status"></span>
    <span id="station"></span>
  </article>
  <article id="message-box">
    <article id="name"></article>
    <article id="message"></article>
  </article>
  <article id="button-box">
    <div id="authenticate-button" class="button-set">
      <div class="button-cell">
        <button id="accept" disabled>確認驗證</button>
      </div>
      <div class="button-cell">
        <button id="reject" disabled>終止驗證</button>
      </div>
    </div>
    <div id="clear-button" class="button-set">
      <div class="button-cell">
        <button id="dismiss">清除訊息</button>
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

const clear_message = () => {
  const xs = $$('#message-box article')
  for (let i = 0; i < xs.length; i++) {
    xs[i].innerHTML = ''
  }
}

const set_attr = (selector, attr) => {
  const xs = $$(selector)
  for (let i = 0; i < xs.length; i++) {
    xs[i].setAttribute(attr, true)
  }
}

const rm_attr = (selector, attr) => {
  const xs = $$(selector)
  for (let i = 0; i < xs.length; i++) {
    xs[i].removeAttribute(attr)
  }
}

const change_button_state = s => {
  if (s === 'default') {
    set_attr('#authenticate-button button', 'disabled')
    rm_attr('#clear-button button', 'disabled')
  }
  else if (s === 'authenticated') {
    set_attr('#clear-button button', 'disabled')
    rm_attr('#authenticate-button button', 'disabled')
  }
  else if (s === 'offline') {
    set_attr('#authenticate-button button', 'disabled')
    set_attr('#clear-button button', 'disabled')
  }
}

let timeout = null

const socket = io(document.URL)

export default {
  name: 'index',
  data () {
    return {}
  },
  created() {
    const app = this

    socket.on('authenticated', data => {
      $('#name').innerHTML    = data.id
      $('#message').innerHTML = `${data.type} ${data.department}`
      change_button_state('authenticated')
    })

    socket.on('confirmed', data => {
      $('#name').innerHTML    = data.student_id
      $('#message').innerHTML = `${l('please go to station')} ${data.slot}`
      change_button_state('default')
    })

    socket.on('message', data => {
      $('#name').innerHTML    = ''
      $('#message').innerHTML = l(data)
      change_button_state('default')
      if (timeout !== null) {
        clearTimeout(timeout)
      }
      timeout = setTimeout(clear_message, 5000)
    })

    socket.on('station', data => {
      $('#station').innerHTML = data
    })

    // Connection ON
    socket.on('connect', () => {
      $('#status').innerHTML     = l('online')
      $('#status-box').className = ''
      change_button_state('default')
    })
    socket.on('reconnect', () => {
      $('#status').innerHTML     = l('online')
      $('#status-box').className = ''
      change_button_state('default')
    })

    // Connection OFF
    socket.on('disconnect', () => {
      $('#status').innerHTML     = l('offline')
      $('#status-box').className = 'offline'
      change_button_state('offline')
      clear_message()
    });

    $('#accept').addEventListener('click', ev => {
      socket.emit('accept')
      change_button_state('default')
    })

    $('#reject').addEventListener('click', ev => {
      socket.emit('reject')
      change_button_state('default')
    })

    $('#dismiss').addEventListener('click', ev => {
      change_button_state('default')
      clear_message()
      clearTimeout(timeout)
    })
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