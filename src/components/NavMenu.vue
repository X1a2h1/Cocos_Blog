<template>
  <div class="top-nav-wrapper">
    <div class="top-nav">
      <div class="left">
        <router-link class="pc-logo" to="/">
          <div  class="logo">
            <img
                width="50"
                :src="cocos_config.other.images.logo.day.big || 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-90b49633-e3ed-4910-b066-7bcc14cbf4b0/031ac7ad-c779-40a8-9b14-d6e963fcbf85.svg'"
                alt
                srcset
            />
<!--                            :src="cocos_config.other.images.logo.day.big || 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-90b49633-e3ed-4910-b066-7bcc14cbf4b0/031ac7ad-c779-40a8-9b14-d6e963fcbf85.svg'"-->

            <div class="title">{{ cocos_config.basic.site.title}}</div>
<!--            {{ geek_config.site_config.title }}-->
          </div>
        </router-link>
        <div  class="logo mobile-logo">
          <img width="35" src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-90b49633-e3ed-4910-b066-7bcc14cbf4b0/031ac7ad-c779-40a8-9b14-d6e963fcbf85.svg" alt srcset />
        </div>
      </div>
      <div class="right">
        <div class="right-links">
<!--          导航开始-->
          <div class="child">
            <router-link to="/">首页</router-link>
          </div>
          <div  class="child">
            <router-link to="/photos"><div>相册</div></router-link>
          </div>
          <div class="child">
            <router-link to="/links"><div>友链</div></router-link>
          </div>
<!--          导航结束-->
<!--          登陆按钮-->

<div>
          <div class="child" v-if="!is_login && login_show" @click="showLogin">
            <a-avatar src="https://joeschmoe.io/api/v1/random" :size="32" />

          </div>
  <div v-else-if="is_login" >
    <a-dropdown placement="bottom"  :trigger="['click']">
      <a class="ant-dropdown-link" @click.prevent>
        <a-avatar :src="cocos_config.basic.site.head_img" :size="32">

        </a-avatar>
      </a>
      <template #overlay>
        <a-menu>
          <a-menu-item key="0">
            <a href="#">个人信息</a>
          </a-menu-item>
          <a-menu-item key="1">
            <a href="#">系统配置</a>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item v-on:click="methods.loginOut()" key="3">退出登陆</a-menu-item>
        </a-menu>
      </template>
    </a-dropdown>
  </div>
        </div>
<!--          登陆按钮-->
<!--模态框-->
          <a-modal id="login-modal" v-model:visible="visible" title="登录你的账户" :closable="false" centered width="360px" @ok="handleOk">
            <template #footer>
              <a-button key="back" @click="handleOk">取消</a-button>
              <a-button key="submit" type="primary"  v-on:click="methods.login()">登陆</a-button>
            </template>
            <a-input
                  v-model:value="account"
                  name="username"
                  type="text"
                  style="width: 280px"
                  :allowClear="true"
                  placeholder="账号"
              />
            <br/><br/>
              <a-input-password
                  v-model:value="password"
                  name="password"
                  v-on:keyup.enter="methods.login()"
                  type="password"
                  style="width: 280px"
                  placeholder="密码"
              />
          </a-modal>
<!--模态框-->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {POST} from "@/utils/http/request";
import { ref, toRefs, reactive,defineComponent } from "vue";
import {mapState, useStore} from 'vuex'
import {inisHelper} from "@/utils/helper/helper.js";
import {Dropdown,Button,notification,Modal,Input,Avatar,Menu,Switch} from "ant-design-vue";

const InputPassword = Input.Password
const MenuItem = Menu.Item
const MenuDivider = Menu.Divider
export default defineComponent( {
  name: "NavMenu",
components:{
  AButton: Button,
    ADropdown: Dropdown,
    Button,
  AModal: Modal,
  AInput: Input,
  AAvatar: Avatar,
  AMenu: Menu,
  ASwitch:Switch,
  AInputPassword: InputPassword,
  AMenuItem: MenuItem,
  AMenuDivider: MenuDivider

},

  setup () {
    const store = useStore()
    const visible = ref(false)
    const showLogin = () => {
      visible.value = true
    }
    const handleOk = e => {
      console.log(e);
      visible.value = false
    }
    const state = reactive({
      account: null, // 帐号
      password: null, // 密码
      user: [], // 用户数据
      is_login: false, // 是否登录
      login_show: true,
    })
        // 获取缓存中的登录信息
    const login_storage = inisHelper.get.storage('login')
    // 判断缓存是否存在且未过期
    if (login_storage != 'expire' && login_storage != false) {
      state.user = login_storage.user
      state.is_login = true;
      store.dispatch('commitLogin');
    } else {
      inisHelper.clear.storage('login')
    }

    const methods = {

      // 登录
      login () {
        console.log(state.account)
        console.log(state.password)
        if (inisHelper.is.empty(state.account))notification.warning({message:'登陆有误！',description:'账号不能为空！'})
        else if (inisHelper.is.empty(state.password))notification.warning({message:'登陆有误！',description:'密码不能为空！'})
        else {
          let params = {
            mode: 'login',
            account: state.account,
            password: state.password
          }
          console.log(params)

          // 登录动画
          // state.login_is_load = true

          POST('users', params).then(res => {
            if (res.data.code === 200) {
              // 设置登录用户信息
              notification.success({message:'登陆成功！',description:'你已经赢了！'})
              state.user = res.data.data.user
              // 有效时间
              res.data.data.time = 7200
              // 登录信息存储到缓存中
              inisHelper.set.storage('login', res.data.data)
              // 关闭登录框
              visible.value = false

              // 更新登录状态
              state.is_login = true
              // 重载页面
              location.reload()
            } else  notification.error({message: '好像有点小问题',description: res.data.msg})
            // 登录动画
            // state.login_is_load = false
          })
        }
      },
      // 退出登录
      loginOut () {
        // 更新登录状态
        state.is_login = false
        // 删除缓存中的登录信息
        window.localStorage.removeItem('login')
        // 重载页面
        // location.reload()
      },
    }

    // onMounted(() => {
    //   methods.initData()
    // })

return{
  ...toRefs(state),
      methods,
      visible,
  showLogin,
  handleOk,
    };
  },
  methods:{

  },
  computed: {
    ...mapState(['cocos_config']),
  },
})

</script>

<style lang="less" scoped>

.modal-body {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 40px;
  box-sizing: border-box;
}

.top-nav-wrapper {
  width: 100%;
  height: 100%;

  .top-nav {
    max-width: calc(1300px - 14px);
    margin: 0 auto;
    width: 100%;
    height: 50px;
    display: flex;
    justify-content: space-between;
    border-radius: 7px;
    align-items: center;
    a {
      background-image: none;
    }
    a:hover {
      color: unset;
    }

    .left {
      display: flex;
      align-items: center;

      .logo {
        display: flex;
        align-items: center;
        margin-right: 20px;

        .title {
          margin-left: 15px;
          font-size: 22px;
          font-weight: 700;
          margin-right: 10px;
        }
        img {
          transition: all 0.25s;
        }
        img:active {
          transform: scale(0.9);
        }
      }
      .pc-logo {
        display: block;
      }
      .mobile-logo {
        display: none;
      }
    }
    .right {
      display: flex;
    }
    .right-links {
      display: flex;
      align-items: center;
      .child {
        display: flex;
        align-items: center;
        cursor: pointer;
        white-space: nowrap;
        font-size: 22px;
        margin-right: 30px;
        transition: all 0.25s;
      }
      .child:last-child {
        margin-right: 0;
      }
      .child:hover {
        transform: scale(1.1);
      }
      .child:active {
        transform: scale(0.9);
      }
      .nickname {
        font-size: 14px;
      }
    }
    .search {
      position: relative;
      margin-right: 30px;
      .feather-search {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        right: 15px;
        color: #999 !important;
        z-index: 99999;
      }
      input {
        position: relative;
        z-index: 99;
        width: 250px;
        height: 30px;
        padding-left: 20px;
        font-size: 14px;
        border-radius: 7px;
        border: none;
        background: #eee;
        transition: all 0.25s;
      }
      .res-box {
        position: absolute;
        left: 0;
        top: 0;
        border-radius: 7px;
        width: 272px;
        height: 0;
        overflow-y: scroll;
        padding: 0 10px 0 10px;
        box-sizing: border-box;
        line-height: 30px;
        color: #000;
        transition: all 0.25s;
        .title {
          width: 100%;
          color: #999;
          font-size: 14px;
          padding: 5px 5px 5px 10px;
        }
        ul {
          display: flex;
          flex-direction: column;
          a {
            position: relative;
            width: 100%;
            padding: 3px 10px;
            border-radius: 7px;
            font-size: 14px;
            color: #666;
          }
          a::before {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 1px;
            background: #eee;
            content: "";
          }
          a:hover {
            //background: rgba(@color: #000000, @alpha: 0.05);
          }
          a:hover::before {
            display: none;
          }
          a:hover + a::before {
            display: none;
          }
        }
      }
      .res-box::-webkit-scrollbar {
        width: 0;
        height: 0;
        display: none;
      }
      input:hover {
        //background: rgba(color: #000000, alpha: 0.1);
      }
      input:focus {
        //background: rgba(color: #ffffff, alpha: 1);
      }
      .is-show-res-box {
        height: 450px;
        padding: 40px 10px 12px 10px;
        //background: rgba(color: #ffffff, alpha: 1);
        box-shadow: 0 0 13px 0 rgb(0 0 0 / 15%);
      }
    }
  }
}
.form-group {
  margin: 0;
}

@media screen and (max-width: 1200px) {
  .top-nav-wrapper {
    width: calc(100% - 8px);
  }
}

// 移动端适配
@media screen and (max-width: 800px) {
  .top-nav-wrapper {
    background: none;
    -webkit-backdrop-filter: none;
    box-shadow: none;
    backdrop-filter: none;
    .search {
      display: none;
    }
    .top-nav {
      height: 45px;
      padding: 0 10px;
      background: none;
      -webkit-backdrop-filter: saturate(200%) blur(20px);
      box-shadow: none;
      backdrop-filter: saturate(200%) blur(20px);
      .left {
        .pc-logo {
          display: none;
        }
        .mobile-logo {
          display: block;
          margin-right: 10px;
        }
        .logo {
          display: flex;
          align-items: center;
          .title {
            display: none;
          }
          img {
            width: auto;
            height: 18px;
          }
        }
      }

      .right-links {
        a {
          font-weight: bold;
          font-size: 14px;
          margin: 0 0;
        }
        .child {
          margin-right: 15px;
        }
      }
    }
  }
}
</style>