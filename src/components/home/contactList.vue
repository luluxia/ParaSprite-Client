<template>
  <div class="contact-list">
    <ul class="type">
      <li @click="showType(0)" :class="[nowShow == 0 ? 'type-active' : '']"><i class="iconfont icon-xiaoxi"></i></li>
      <li @click="showType(1)" :class="[nowShow == 1 ? 'type-active' : '']"><i class="iconfont icon-lianxiren"></i></li>
      <li @click="showType(2)" :class="[nowShow == 2 ? 'type-active' : '']"><i class="iconfont icon-qunliao"></i></li>
    </ul>
    <!-- 消息列表 -->
    <div :style="{transform: 'translate(' + (0 - 0.5 * nowShow) + 'em, 0)'}" :class="[nowShow == 0 ? '' : 'type-hide', 'item-list']">
      <ul v-for="(lists, n) in messageList" :key="lists.id" :class="['list', lists.show ? '' : 'list-hide']">
        <li @click="show(n)" class="title">
          <i class="iconfont icon-zhankai"></i>
          {{lists.name}}
        </li>
        <li @contextmenu.prevent='onUserMenu(item.id)'
            @click="choose(item.id, item.type)"
            v-for="item in lists.list"
            :key="item.type + item.id"
            :class="[
              'item',
              nowChatId == item.id ? 'active-item' : '',
              item.lastMsgNum ? 'status-new': '',
              item.type == 'user' && !include.user[item.id].online ? 'status-offline' : ''
            ]"
        >
          <div class="avatar">
            <img v-if="item.type=='user'" :src="include.user[item.id].avatar ? '' : `https://api.multiavatar.com/${item.id}.png`" alt="">
            <img v-else :src="include.group[item.id].avatar ? 'http://192.168.26.96:7001/public/' + include.group[item.id].avatar : `https://api.multiavatar.com/${item.id}.png`" alt="">
          </div>
          <div class="info">
            <p class="nick">{{item.type == 'user' ? include.user[item.id].nick : include.group[item.id].nick}}</p>
            <p class="content">{{item.lastMsg}}</p>
          </div>
          <div v-if="item.lastMsgNum" class="num">
            <p>+{{item.lastMsgNum}}</p>
          </div>
        </li>
      </ul>
    </div>
    <!-- 联系人列表 -->
    <div :style="{transform: 'translate(' + (0.5 - 0.5 * nowShow) + 'em, 0)'}" :class="[nowShow == 1 ? '' : 'type-hide', 'item-list']">
      <ul v-for="(lists, n) in contactList" :key="lists.id" :class="['list', lists.show ? '' : 'list-hide']">
        <li @click="show(n)" class="title"><i class="iconfont icon-zhankai"></i>{{lists.name}}</li>
        <li @click="choose(item.id, 'user')"
          v-for="item in lists.list"
          :key="item.id"
          :class="[
            'item',
            nowChatId == item.id ? 'active-item' : '',
            include.user[item.id].online ? '' : 'status-offline'
          ]"
        >
          <div class="avatar">
            <img :src="include.user[item.id].avatar ? '' : `https://api.multiavatar.com/${item.id}.png`" alt="">
          </div>
          <div class="info">
            <p class="nick">{{include.user[item.id].nick}}</p>
            <p class="content">{{include.user[item.id].sign ? include.user[item.id].sign : '这个人很懒，什么都没有写'}}</p>
            <div>
              <p class="content">{{include.user[item.id].emoji}}{{include.user[item.id].sign}} </p>
            </div>
          </div>
        </li>
      </ul>
    </div>
    <!-- 群列表 -->
    <div :style="{transform: 'translate(' + (1 - 0.5 * nowShow) + 'em, 0)'}" :class="[nowShow == 2 ? '' : 'type-hide', 'item-list']">
      <ul v-for="(lists, n) in groupList" :key="n" :class="['list', lists.show ? '' : 'list-hide']">
        <li @click="show(n)" class="title"><i class="iconfont icon-zhankai"></i>{{lists.name}}</li>
        <li @click="choose(item.id, 'group')"
          v-for="(item, m) in lists.list" :key="m"
          :class="[
            'item',
            nowChatId == item.id ? 'active-item' : '',
          ]"
        >
          <div class="avatar">
            <img :src="include.group[item.id].avatar ? 'http://192.168.26.96:7001/public/' + include.group[item.id].avatar : `https://api.multiavatar.com/${item.id}.png`" alt="">
          </div>
          <div class="info">
            <p class="nick">{{include.group[item.id].nick}}</p>
            <p class="content">{{item.lastMsg}}</p>
          </div>
          <div v-if="item.lastMsgNum" class="num">
            <p>+{{item.lastMsgNum}}</p>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import _ from 'lodash'
export default {
  data () {
    return {
      // 当前展示的类别
      nowShow: 0,
      // 消息列表
      messageList: [],
      // 联系人列表
      contactList: {},
      // 群聊列表
      groupList: {}
    }
  },
  computed: {
    ...mapState(['relationship', 'include', 'nowChatId'])
  },
  watch: {
    relationship () {
      this.getData()
    }
  },
  methods: {
    ...mapActions(['showChat', 'setRelationInfo']),
    show (n) {
      if (this.nowShow === 0) {
        this.messageList[n].show = !this.messageList[n].show
      } else if (this.nowShow === 1) {
        this.contactList[n].show = !this.contactList[n].show
      } else if (this.nowShow === 2) {
        this.groupList[n].show = !this.groupList[n].show
      }
    },
    showType (n) {
      this.nowShow = n
    },
    choose (id, type) {
      console.log(id)
      this.showChat({
        id: id,
        type: type
      })
      const index = _.findIndex(this.relationship, { id: id })
      if (this.relationship[index].lastMsgNum) {
        this.$socket.emit('clearLastMsgNum', { id: id })
        this.setRelationInfo({
          id: id,
          content: {
            lastMsgNum: 0
          }
        })
      }
    },
    onUserMenu (id) {
      this.$contextmenu({
        items: [
          { label: '置顶调整', onClick: () => { this.axios.get(`/api/${id}/top`) } },
          { label: '修改备注' },
          { label: '移出消息列表', divided: true, onClick: () => { this.axios.get(`/api/${id}/remove`) } },
          { label: '删除联系人', onClick: () => { this.axios.get(`/api/${id}/delete`); this.showChat({ id: null, type: null }) } }
        ],
        customClass: 'contextmenu',
        event
      })
    },
    getData () {
      console.log('update')
      // 初始化关系表
      const contactList = {}
      const groupList = {}
      const messageList = [
        {
          id: 0,
          name: '置顶',
          show: 1,
          list: []
        },
        {
          id: 1,
          name: '消息',
          show: 1,
          list: []
        },
        {
          id: 2,
          name: '服务',
          show: 1,
          list: []
        }
      ]
      this.relationship.forEach(item => {
        // 初始化消息列表
        const data = {
          id: item.id,
          type: item.type,
          lastMsg: item.lastMsg,
          lastMsgNum: item.lastMsgNum,
          lastActiveTime: item.lastActiveTime
        }
        if (item.inChat) {
          if (item.top) {
            messageList[0].list.push(data)
          } else if (item.group === '我的服务') {
            messageList[2].list.push(data)
          } else {
            messageList[1].list.push(data)
          }
        }
        // 初始化联系人
        if (item.type === 'user') {
          if (!contactList[item.groupId]) {
            contactList[item.groupId] = {
              name: item.group,
              show: 1,
              list: []
            }
          }
          contactList[item.groupId].list.push({
            id: item.id
          })
        } else {
          // 初始化群组列表
          if (!groupList[item.groupId]) {
            groupList[item.groupId] = {
              name: item.group,
              show: 1,
              list: []
            }
          }
          groupList[item.groupId].list.push({
            id: item.id,
            lastMsg: item.lastMsg,
            lastMsgNum: item.lastMsgNum
          })
        }
      })
      this.messageList = messageList
      this.contactList = contactList
      this.groupList = groupList
      this.sortMessageList()
    },
    sortMessageList () {
      this.messageList[0].list = this._.orderBy(this.messageList[0].list, 'lastActiveTime', 'desc')
      this.messageList[1].list = this._.orderBy(this.messageList[1].list, 'lastActiveTime', 'desc')
      this.messageList[2].list = this._.orderBy(this.messageList[2].list, 'lastActiveTime', 'desc')
    }
  }
}
</script>

<style lang="sass" scoped>
  .contact-list
    width: 100%
    height: calc(100% - 2em)
  .type
    display: flex
    justify-content: space-between
    border-bottom: 1px solid var(--hover-bg)
    li
      display: inline-block
      width: calc( 100% / 3 )
      height: 2.5em
      text-align: center
      line-height: 2.5em
      color: var(--text)
      transition: color 0.3s
      cursor: pointer
      &:hover
        color: var(--main)
  .type-active
    color: var(--main) !important
    border-bottom: 1px solid

  .item-list
    position: absolute
    width: 100%
    height: calc(100% - 4em)
    margin-top: 1em
    padding: 0 1em
    overflow-y: auto
    transition: all 0.3s
    &::-webkit-scrollbar
      width: 0.2em
    &::-webkit-scrollbar-thumb
      background: rgba(0, 0, 0, 0)
      transition: background 0.3s
    &:hover
      &::-webkit-scrollbar-thumb
        background: rgba(0, 0, 0, 0.1)
  .type-hide
    opacity: 0
    pointer-events: none
  .list
    margin-bottom: 0.5em
  .list-hide
    .icon-zhankai
      transform: rotate(90deg)
    .item
      display: none
  .title
    position: sticky
    display: flex
    align-items: center
    list-style: none
    font-weight: bold
    color: var(--text)
    background: var(--hover-bg)
    top: 0
    padding: 0.5em
    opacity: 0.8
    font-size: 0.8em
    margin-bottom: 0.5em
    backdrop-filter: blur(5px)
    border-radius: 0.2em
    z-index: 1
    cursor: pointer
    i
      display: inline-block
      width: 1em
      height: 1em
      font-size: 0.8em
      margin-right: 0.5em
      transform: rotate(180deg)
  .item
    display: flex
    padding: 0.5em
    align-items: center
    border-radius: 0.2em
    transition: background 0.3s
    cursor: pointer
    &:hover
      background: var(--hover-bg)
  .active-item
    display: flex !important
    background: var(--active-bg) !important
  .avatar
    width: 2.5em
    height: 2.5em
    background: var(--hover-bg)
    border-radius: 2em
    img
      width: 100%
      height: 100%
      border-radius: 2em
  .info
    margin-left: 0.5em
    width: calc(100% - 3em)
  .nick
    color: var(--text)
    margin-bottom: 0.1em
  .content
    font-size: 0.8em
    opacity: 0.6
    color: var(--text)
    white-space: nowrap
    text-overflow: ellipsis
    overflow: hidden
  .num
    position: absolute
    right: 0.5em
    color: var(--main)
  .status-new
    .content
      color: var(--main)
      opacity: 0.8
  .status-offline
    opacity: 0.5
</style>
