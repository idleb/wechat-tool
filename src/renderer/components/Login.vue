<template>
  <div id="login">
    <figure v-loading="loading">
      <img v-if="url" :src="url">
      <figcaption>{{message[0]}}</figcaption>
      <figcaption>{{message[1]}}</figcaption>
    </figure>
  </div>
</template>

<script>

export default {
  name: 'login',

  data () {
    return {
      url: false,
      code: false,
      loading: true
    }
  },

  computed: {
    message () {
      if (this.code === 201) {
        return ['扫描成功', '请在手机上点击确认以登录']
      }

      return ['扫码登录微信', '网页版微信需要配合你的手机登录使用']
    },
    groups () {
      return this.$store.state.Config.groups
    }
  },

  mounted () {
    const wechaty = require('electron').remote.getGlobal('wechaty')
    wechaty
      .on('scan', (url, code) => {
        console.log(`Scan QR Code to login: ${code}\n${url}`)
        this.url = url
        this.code = code
        this.loading = false
      })
      .on('login', user => {
        console.log(`User ${user} logined`)
        this.$store.dispatch('init', {wechaty, user})
        console.log(user.id)

        this.$router.replace(this.$route.query.redirect || '/')
      })
      .on('message', m => {
        console.log(`message ${m} received`)
        if (m.self()) {
          return
        }
        const contact = m.from()
        const content = m.content()
        const room = m.room()

        console.log(contact)
        console.log(content)
        console.log(room)

        this.groups.forEach(config => {
          console.log(config)
          if (config.disable) return
          const configRoom = config.rooms.find(item => item.room.id === room.id)
          console.log(configRoom)
          if (configRoom) {
            const speaker = configRoom.members.find(item => item.id === contact.id)
            console.log(speaker)
            if (speaker) {
              console.log(speaker, '===>', config.rooms)
              config.rooms.forEach(item => {
                if (item.room.id === room.id) return
                item.room.say(m)
              })
            }
          }
        })
      })
      .start()
  },

  methods: {
  }
}
</script>

<style scoped>
#login{
  overflow: hidden;
}
figure{
  width: 80%;
  margin: 20% auto;
  text-align: center;
  vertical-align: middle;
}
figure>img{
  width: 80%;
}
figcaption{
  font-size: 0.8em;
  color: rgba(0, 0, 0, 0.5);
}
figcaption:first-of-type{
  font-size: 1.2em;
  color: rgba(0, 0, 0, 1);
}
</style>
