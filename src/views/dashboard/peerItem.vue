<template>
    <el-card class="box-card" shadow="hover">
     <div @click="openPeerModal">
       <div slot="header" class="card__header">
         <span>public key: {{ item.url_safe_public_key }}</span>
         <i class="el-icon-setting" @click="openPeerModal"></i>
       </div>
       <div class="text item">
         <div class="info__peer">
           <span>receive: {{ item.receive_bytes }}</span>
           <span>transmit: {{ item.transmit_bytes }}</span>
         </div>
       </div>
       <div class="peer__buttons">
         <el-button
           type="primary"
           @click.stop="getQrCode"
         >
           qr code
         </el-button>
         <el-button
           icon="el-icon-download"
           type="info"
           @click.stop="getQuickConf"
         >
           quick.conf
         </el-button>
       </div>
     </div>
      <el-drawer
        :title="`${item.url_safe_public_key}`"
        :visible.sync="drawer"
        direction="rtl"
        >
        <div class="detail__info">

          <span>Private key</span>
          <el-input v-model="editedPeer.private_key" class="detail__info-input"></el-input>

          <span>Public key</span>
          <el-input v-model="editedPeer.public_key" class="detail__info-input" :disabled="editedPeer.private_key.length > 0"></el-input>
          <span>Persistent keepalive interval</span>
          <el-input v-model="editedPeer.persistent_keepalive_interval" class="detail__info-input"></el-input>
          <span>endpoint:</span>
          <el-input v-model="editedPeer.endpoint" class="detail__info-input"></el-input>
          <span>Allowed ips</span>
          <el-input type="textarea" rows="4" v-model="editedPeer.allowed_ips" class="detail__info-input"></el-input>
        </div>
       <div class="detail__info-buttons">
         <el-button type="primary" @click="savePeer">Save</el-button>
         <el-button type="danger" @click="deletePeer">Delete</el-button>
       </div>
      </el-drawer>
      <el-dialog
        title="QR Code"
        :visible.sync="dialogVisible"
        width="30%"
      >
        <div class="qr__dialog">
          <img :src="qrCode" alt="">
        </div>
      </el-dialog>
    </el-card>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { Peer, PeerCreateOrUpdateRequest } from 'wgrest/dist/models'
import { deviceApi } from '@/api/interface'

@Component({
  name: 'peerItem',

  filters: {
    date(value: string) {
      const time = new Date(value)

      const year = time.getFullYear()
      const month = time.getMonth()
      const day = time.getDate()

      const hours = time.getHours()
      const minutes = time.getMinutes()

      return `${day < 10 ? `0${day}` : day}-${month < 10 ? `0${month}` : month}-${year} at ${hours < 10 ? `0${hours}` : hours}:${minutes < 10 ? `0${minutes}` : minutes}`
    }
  }
})
export default class peerItem extends Vue {
  @Prop({ default: {} }) item!: Peer

  private editedPeer: PeerCreateOrUpdateRequest = {
    public_key: '',

    private_key: '',

    preshared_key: '',

    allowed_ips: '',

    persistent_keepalive_interval: '',

    endpoint: '',

  }

  private drawer = false
  private dialogVisible = false

  private qrCode: string | unknown = ''

  public async getQrCode(): Promise<void> {
    const { data } = await deviceApi.getDevicePeerQuickConfigQRCodePNG(this.$route.params.id, this.item.url_safe_public_key, '', { responseType: 'arraybuffer' })
    const blob = new Blob([data])
    this.qrCode = URL.createObjectURL(blob)
    this.dialogVisible = true
  }

  public async getQuickConf(): Promise<void> {
    const { data } = await deviceApi.getDevicePeerQuickConfig(this.$route.params.id, this.item.url_safe_public_key)
    const blob = new Blob([data])
    const link = document.createElement('a')
    link.href = window.URL.createObjectURL(blob)
    link.download = `${this.item.url_safe_public_key}.conf`
    link.click()
  }

  private async savePeer(): Promise<void> {
    await deviceApi.updateDevicePeer(this.$route.params.id, this.item.url_safe_public_key, {
      ...this.editedPeer,
      allowed_ips: peerItem.prepareAllowedIps(this.editedPeer.allowed_ips)
    })

    this.$message({
      type: 'success',
      message: 'Peer updated'
    })
  }

  private static prepareAllowedIps(item: string): string[] | string {
    if (item.length > 0) {
      return item.split(',').map((item: string) => item.replace(/\n|\r/g, ''))
    }
    return []
  }

  private async deletePeer(): Promise<void> {
    await this.$msgbox({
      title: 'Delete peer',
      message: `Delete ${this.item.url_safe_public_key} ?`,
      showCancelButton: true,
      confirmButtonText: 'Confirm',
      cancelButtonText: 'Cancel',
      beforeClose: async (action, instance, done) => {
        if (action === 'confirm') {
          instance.confirmButtonLoading = true;
          instance.confirmButtonText = 'Loading...';
          const answer = await deviceApi.deleteDevicePeer(this.$route.params.id, this.item.url_safe_public_key)

          if (answer.status.toString().startsWith('2')) {
            this.$message({
              type: 'success',
              message: 'Peer deleted'
            })
          }

          done()
          instance.confirmButtonLoading = false;
          this.$emit('delete', this.item.public_key)

          this.drawer = false
        } else {
          done();
        }
      }
    })
  }

  private openPeerModal() {
    this.editedPeer = JSON.parse(JSON.stringify(this.item))

    if (!Object.keys(this.editedPeer).includes('private_key')) {
      this.$set(this.editedPeer, 'private_key', '')
    }

    if (this.editedPeer.allowed_ips.length) {
      this.editedPeer.allowed_ips = this.editedPeer.allowed_ips.join(',')
    }else {
      this.editedPeer.allowed_ips = ''
    }

    this.drawer = true;
  }
}
</script>

<style lang="scss" scoped>
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.box-card {
  width: 480px;

  cursor: pointer;
  margin: 0 50px 50px 0;

  @media (max-width: 850px) {
    width: 100%;
    margin-right: 0;
  }
}

.card__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  margin-bottom: 20px;
}

.qr__code {
  width: 200px;
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;

  img {
    width: 100%;
    height: 100%;
  }
}

.info__peer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.detail__info {
  display: flex;
  flex-direction: column;

  padding: 20px;

  span {
    margin-bottom: 5px;
    font-weight: bold;
  }

  &-input {
    margin-bottom: 20px;
  }
}

.peer__buttons {
  margin-top: 30px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.qr__dialog {
  display: flex;
  align-items: center;
  justify-content: center;
}

.detail__info-buttons {
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>
