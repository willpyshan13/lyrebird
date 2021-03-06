<template>
  <div>
    <Layout class="main-layout">
      <Sider
        ref="mainSider"
        class="sider-bar"
        hide-trigger
        collapsible
        :collapsed-width="50"
        v-model="isCollapsed"
      >
        <div class="logo">
          <img src="@/assets/lyrebird.logo.png" />
          <span>{{logo}}</span>
        </div>
        <Divider class="sider-bar-divider" />
        <Menu theme="dark" width="auto" :class="menuitemClasses" :active-name="activeName">
          <MenuItem
            v-for="(menuItem, index) in menu"
            :key="index"
            :name="menuItem.title"
            @click.native="menuItemOnClick(menuItem)"
          >
            <Tooltip :content="menuItem.title" placement="right" :disabled="!isCollapsed">
              <b>{{menuItemTitle(menuItem)}}</b>
            </Tooltip>
          </MenuItem>
        </Menu>
      </Sider>
      <Layout>
        <Header class="main-header" inline>
          <Icon type="md-menu" color="white" size="24" @click.native="collapsedSider"></Icon>
          <notice-center></notice-center>
        </Header>
        <Content>
          <div class="main-container">
            <router-view></router-view>
          </div>
        </Content>
        <Footer class="main-footer">
          <span v-show="activatedGroupName" class="main-footer-mock-status">
            <b>Activated mock group: {{activatedGroupName}}</b>
            <Icon type="md-close-circle" style="cursor:pointer;" @click="resetActivatedData"/>
          </span>
          <span class="main-footer-right">
            <span class="main-footer-copyright">
              <strong style="color:#f8f8f9">
                Copyright &copy; 2018-present
                <a
                  href="https://meituan-dianping.github.io/lyrebird"
                  target="_blank"
                >Meituan</a>. All rights reserved.
              </strong>
            </span>
            <Poptip
              v-if="status"
              content="content"
              placement="top-end"
              class="main-footer-status"
              width="220"
            >
              <a>
                <Icon type="ios-arrow-up" style="color:#f8f8f9" />
                <b style="color:#f8f8f9">&nbsp;&nbsp;Version {{status.version}}</b>
              </a>
              <div slot="title">
                <b>💡Status</b>
              </div>
              <div slot="content">
                <Row v-for="key in showedStatus" :key="key">
                  <i-col span="11">
                    <b style="float: right">{{key.toUpperCase()}}</b>
                  </i-col>
                  <i-col span="12" offset="1">{{status[key]}}</i-col>
                </Row>
              </div>
            </Poptip>
            <a
              href="https://github.com/Meituan-Dianping/lyrebird/issues/new?assignees=&labels=&template=bug_report.md&title="
              target="_blank"
              class="main-footer-status"
            >
              <Icon type="ios-bug" color="white" />
            </a>
          </span>
        </Footer>
      </Layout>
    </Layout>
  </div>
</template>

<script>
import NoticeCenter from '@/views/notice/NoticeCenter.vue'

export default {
  name: 'MainLayout',
  components: {
    NoticeCenter
  },
  data () {
    return {
      isCollapsed: true,
      showedStatus: ["ip", "mock.port", "proxy.port", "version"]
    }
  },
  mounted () {
    this.$store.dispatch('loadMenu')
    this.$store.dispatch('loadStatus')
    this.$store.dispatch('loadManifest')
    this.$store.dispatch('loadActivatedGroup')
  },
  created () {
    this.$bus.$on('msg.success', this.successMessage)
    this.$bus.$on('msg.info', this.infoMessage)
    this.$bus.$on('msg.error', this.errorMessage)
  },
  computed: {
    menuitemClasses () {
      return ["menu-item", this.isCollapsed ? "collapsed-menu" : "menu"];
    },
    logo () {
      if (this.isCollapsed) {
        return ''
      } else {
        return 'Lyrebird'
      }
    },
    menu () {
      return this.$store.state.menu
    },
    status () {
      return this.$store.state.status
    },
    manifest () {
      return this.$store.state.manifest
    },
    activeName () {
      return this.$store.state.activeName
    },
    activatedGroupName () {
      const activatedGroups = this.$store.state.inspector.activatedGroup
      if (activatedGroups === null) {
        return null
      }
      if (Object.keys(activatedGroups) === 0) {
        return null
      }
      let text = ''
      for (const groupId in activatedGroups) {
        text = text + activatedGroups[groupId].name + ' '
      }
      return text
    }
  },
  methods: {
    collapsedSider () {
      this.$refs.mainSider.toggleCollapse();
    },
    menuItemTitle (menuItem) {
      if (this.isCollapsed) {
        return menuItem.title.substring(0, 1)
      } else {
        return menuItem.title
      }
    },
    menuItemOnClick (menuItem) {
      this.$store.commit('setActiveName', menuItem.title)
      if (menuItem.type === 'router') {
        if (menuItem.name === 'plugin-view' || menuItem.name === 'plugin-container') {
          this.$store.commit('plugin/setSrc', menuItem.params.src);
        }
        this.$router.push({ name: menuItem.name, params: menuItem.params });
      } else {
        window.open(menuItem.path, '_self');
      }
    },
    resetActivatedData () {
      this.$store.dispatch('deactivateGroup')
    },
    successMessage (msg) {
      this.$Message.success({
        content: msg,
        duration: 3,
        closable: true
      })
    },
    infoMessage (msg) {
      this.$Message.info({
        content: msg,
        duration: 3,
        closable: true
      })
    },
    errorMessage (msg) {
      this.$Message.error({
        content: msg,
        duration: 0,
        closable: true
      })
    }
  }
};
</script>

<style scoped>
.main-layout {
  height: 100vh;
}
.sider-bar {
  background-color: #515a6e;
}
.sider-bar-divider {
  height: 1px;
  margin: 0;
  background: #6c6c6c;
}
.logo {
  height: 38px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.logo span {
  color: white;
  font-size: 25px;
  font-weight: bolder;
  font-style: italic;
}
.logo img {
  width: 32px;
}
.main-header {
  height: 38px;
  line-height: 38px;
  padding: 0;
  margin: 0;
  /* background-color: #fec142; */
  background-color: #0fccbf;
}
.main-footer {
  height: 28px;
  line-height: 28px;
  padding: 0;
  /* background-color: #fec142; */
  background-color: #0fccbf;
}
.main-footer-mock-status {
  margin-left: 15px;
  font-size: 12px;
  color: #f8f8f8;
}
.main-footer-copyright {
  font-size: 12px;
  margin-left: 10px;
  margin-right: 10px;
  color: #f8f8f9;
}
.main-footer-status {
  font-size: 11px;
  margin-right: 10px;
}
.main-footer-right {
  float: right;
  margin-right: 10px;
}
.collapsed-menu span {
  width: 0px;
  transition: width 0.2s ease;
}
.menu-item span {
  display: inline-block;
  overflow: hidden;
  width: 69px;
  text-overflow: ellipsis;
  white-space: nowrap;
  vertical-align: bottom;
  transition: width 0.2s ease 0.2s;
}
.main-container {
  padding-left: 5px;
  padding-right: 5px;
  height: calc(100vh - 66px);
  background: #fff;
}
</style>

