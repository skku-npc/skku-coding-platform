<template>
  <div id="header">
    <Menu
      theme="light"
      mode="horizontal"
      :active-name="activeMenu"
      class="oj-menu"
      @on-select="handleRoute"
    >
      <div class="logo">
        <span><img src="https://www.skku.edu/_res/skku//img/common/logo.png"></span>
      </div>
      <Menu-item name="/">
        <Icon type="home" />
        {{ $t('m.Home') }}
      </Menu-item>
      <Menu-item name="/problem">
        <Icon type="ios-keypad" />
        {{ $t('m.NavProblems') }}
      </Menu-item>
      <Menu-item name="/contest">
        <Icon type="trophy" />
        {{ $t('m.Contests') }}
      </Menu-item>
      <Menu-item name="/status">
        <Icon type="ios-pulse-strong" />
        {{ $t('m.NavStatus') }}
      </Menu-item>
      <Submenu name="rank">
        <template slot="title">
          <Icon type="podium" />
          {{ $t('m.Rank') }}
        </template>
        <Menu-item name="/acm-rank">
          {{ $t('m.ACM_Rank') }}
        </Menu-item>
        <Menu-item name="/oi-rank">
          {{ $t('m.OI_Rank') }}
        </Menu-item>
      </Submenu>
      <Submenu name="about">
        <template slot="title">
          <Icon type="information-circled" />
          {{ $t('m.About') }}
        </template>
        <Menu-item name="/about">
          {{ $t('m.Judger') }}
        </Menu-item>
        <Menu-item name="/FAQ">
          {{ $t('m.FAQ') }}
        </Menu-item>
      </Submenu>
      <template v-if="!isAuthenticated">
        <div class="btn-menu">
          <Button
            ref="loginBtn"
            type="ghost"
            shape="circle"
            @click="handleBtnClick('login')"
          >
            {{ $t('m.Login') }}
          </Button>
          <Button
            v-if="website.allow_register"
            type="ghost"
            shape="circle"
            style="margin-left: 5px;"
            @click="handleBtnClick('register')"
          >
            {{ $t('m.Register') }}
          </Button>
        </div>
      </template>
      <template v-else>
        <Dropdown
          class="drop-menu"
          placement="bottom"
          trigger="click"
          @on-click="handleRoute"
        >
          <Button
            type="text"
            class="drop-menu-title"
          >
            {{ user.username }}
            <Icon type="arrow-down-b" />
          </Button>
          <Dropdown-menu slot="list">
            <Dropdown-item name="/user-home">
              {{ $t('m.MyHome') }}
            </Dropdown-item>
            <Dropdown-item name="/status?myself=1">
              {{ $t('m.MySubmissions') }}
            </Dropdown-item>
            <Dropdown-item name="/setting/profile">
              {{ $t('m.Settings') }}
            </Dropdown-item>
            <Dropdown-item
              v-if="isAdminRole"
              name="/admin"
            >
              {{ $t('m.Management') }}
            </Dropdown-item>
            <Dropdown-item
              divided
              name="/logout"
            >
              {{ $t('m.Logout') }}
            </Dropdown-item>
          </Dropdown-menu>
        </Dropdown>
      </template>
    </Menu>
    <Modal
      v-model="modalVisible"
      :width="400"
    >
      <div
        slot="header"
        class="modal-title"
      >
        {{ $t('m.Welcome_to') }} {{ website.website_name }}
      </div>
      <component
        :is="modalStatus.mode"
        v-if="modalVisible"
      />
      <div
        slot="footer"
        style="display: none"
      />
    </Modal>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import login from '@oj/views/user/Login'
import register from '@oj/views/user/Register'

export default {
  components: {
    login,
    register
  },
  mounted () {
    this.getProfile()
  },
  methods: {
    ...mapActions(['getProfile', 'changeModalStatus']),
    handleRoute (route) {
      if (route && route.indexOf('admin') < 0) {
        this.$router.push(route)
      } else {
        window.open('/admin/')
      }
    },
    handleBtnClick (mode) {
      if (mode === 'register') {
        this.$alert('아이디(Username)는 자신의 학번으로 설정해주세요')
      }
      this.changeModalStatus({
        visible: true,
        mode: mode
      })
    }
  },
  computed: {
    ...mapGetters(['website', 'modalStatus', 'user', 'isAuthenticated', 'isAdminRole']),
    // Follow routing changes
    activeMenu () {
      return '/' + this.$route.path.split('/')[1]
    },
    modalVisible: {
      get () {
        return this.modalStatus.visible
      },
      set (value) {
        this.changeModalStatus({ visible: value })
      }
    }
  }
}
</script>

<style lang="less" scoped>
  #header {
    min-width: 300px;
    position: fixed;
    top: 0;
    left: 0;
    height: auto;
    width: 100%;
    z-index: 1000;
    background-color: #fff;
    box-shadow: 0 1px 5px 0 rgba(0, 0, 0, 0.1);
    .oj-menu {
      background: #fdfdfd;
    }

    .logo {
      margin-left: 2%;
      margin-right: 2%;
      float: left;
      height: 100%;
    }

    .drop-menu {
      float: right;
      margin-right: 30px;
      position: absolute;
      right: 10px;
      &-title {
        font-size: 18px;
      }
    }
    .btn-menu {
      font-size: 16px;
      float: right;
      margin-right: 10px;
    }
  }
  .modal {
    &-title {
      font-size: 18px;
      font-weight: 600;
    }
  }
</style>
