<template>
  <div
    class="navbar"
    :class="{'navbar_opened': menuOpened}"
  >
    <hamburger
      id="hamburger-container"
      :is-active="sidebar.opened"
      class="hamburger-container"
      @toggle-click="toggleSideBar"
    />
    <breadcrumb
      id="breadcrumb-container"
      class="breadcrumb-container"
    />
    <div class="right-menu">
      <el-button
        class="navbar__button"
        type="primary"
        icon="el-icon-refresh"
        @click="updateList"
      ></el-button>
      <el-button
        class="navbar__button"
        type="primary"
        icon="el-icon-plus"
        v-if="CurrentButton === NewAddButtonEnum.Device"
        @click="openModal('device')"
      >
        <span class="add__button-navbar">New Device</span>
      </el-button>
      <el-button
        class="navbar__button"
        type="primary"
        icon="el-icon-plus"
        @click="openModal('peer')"
        v-if="CurrentButton === NewAddButtonEnum.Peer"
      >
        <span class="add__button-navbar">New Peer</span>
      </el-button>
<!--      <el-dropdown-->
<!--        class="avatar-container right-menu-item hover-effect"-->
<!--        trigger="click"-->
<!--      >-->
<!--        <div class="avatar-wrapper">-->
<!--          <img-->
<!--            :src="avatar+'?imageView2/1/w/80/h/80'"-->
<!--            class="user-avatar"-->
<!--          >-->
<!--          <i class="el-icon-caret-bottom" />-->
<!--        </div>-->
<!--      </el-dropdown>-->
<!--      <el-dropdown-menu slot="dropdown">-->
<!--        <router-link to="/">-->
<!--          <el-dropdown-item>-->
<!--            Home-->
<!--          </el-dropdown-item>-->
<!--        </router-link>-->
<!--        <el-dropdown-item divided>-->
<!--            <span-->
<!--              style="display:block;"-->
<!--              @click="logout"-->
<!--            >LogOut</span>-->
<!--        </el-dropdown-item>-->
<!--      </el-dropdown-menu>-->
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { AppModule } from '@/store/modules/app'
import { UserModule } from '@/store/modules/user'
import { ModalModule } from '@/store/modules/modal'
import Breadcrumb from '@/components/Breadcrumb/index.vue'
import Hamburger from '@/components/Hamburger/index.vue'

import { emitter } from '@/utils/emmiter'

export enum NewAddButtonEnum {
  Device = 'Device',
  Peer = 'Peer'
}

@Component({
  name: 'Navbar',
  components: {
    Breadcrumb,
    Hamburger
  }
})
export default class extends Vue {
  NewAddButtonEnum = NewAddButtonEnum
  get sidebar() {
    return AppModule.sidebar
  }

  get device() {
    return AppModule.device.toString()
  }

  get avatar() {
    return UserModule.avatar
  }

  get menuOpened() {
    return AppModule.sidebar.opened
  }

  private toggleSideBar() {
    AppModule.ToggleSideBar(false)
  }

  public openModal(name: string): void {
    ModalModule.openModal(name)
  }

  get CurrentButton(): String {
    if (this.$route.name === 'Device') {
      return NewAddButtonEnum.Device
    }
    if (this.$route.name === 'Peer') {
      return NewAddButtonEnum.Peer
    }

    return ''
  }

  private async logout() {
    await UserModule.LogOut()
    this.$router.push(`/login?redirect=${this.$route.fullPath}`)
  }

  private updateList(): void {
    if (this.$route.name === 'Device') {
      emitter.emit('updateDevice')

      this.$message({
        type: 'success',
        message: 'Devices updated'
      })
    }
    if (this.$route.name === 'Peer') {
      emitter.emit('updatePeer')

      this.$message({
        type: 'success',
        message: 'Peers updated'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
  position: fixed;
  z-index: 100;
  width: calc(100vw - 54px);
  background: #fff;
  box-shadow: 0 1px 4px rgba(0,21,41,.08);
  transition: all .2s ease-in-out;

  &_opened {
    width: calc(100vw - 210px);
  }

  @media (max-width: 850px) {
    width: 100%;
  }

  .hamburger-container {
    line-height: 46px;
    height: 100%;
    float: left;
    padding: 0 15px;
    cursor: pointer;
    transition: background .3s;
    -webkit-tap-highlight-color:transparent;

    &:hover {
      background: rgba(0, 0, 0, .025)
    }
  }

  .breadcrumb-container {
    float: left;
  }

  .right-menu {
    float: right;
    height: 100%;
    line-height: 50px;

    @media (max-width: 850px) {
      display: flex;
      align-items: center;
      float: right;
      margin-right: 10px;
    }

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 8px;
      height: 100%;
      font-size: 18px;
      color: #5a5e66;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: background .3s;

        &:hover {
          background: rgba(0, 0, 0, .025)
        }
      }
    }

    .avatar-container {
      margin-right: 30px;

      .avatar-wrapper {
        margin-top: 5px;
        position: relative;

        .user-avatar {
          cursor: pointer;
          width: 40px;
          height: 40px;
          border-radius: 10px;
        }

        .el-icon-caret-bottom {
          cursor: pointer;
          position: absolute;
          right: -20px;
          top: 25px;
          font-size: 12px;
        }
      }
    }
  }
}

.navbar__button {
  margin-right: 30px;

  @media (max-width: 850px) {
    margin-right: 0;
  }
}

.add__button-navbar {
  @media (max-width: 850px) {
    display: none;
  }
}
</style>
