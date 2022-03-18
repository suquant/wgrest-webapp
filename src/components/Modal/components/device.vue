<template>
  <div>
    <p class="device_form modal_title">Add new Device</p>
    <el-input
      class="device_form"
      placeholder="Name"
      v-model="newDeviceForm.name"
    ></el-input>
    <p>Listen Port</p>
    <el-input-number
      class="device_form"
      placeholder="Listen Port"
      v-model="newDeviceForm.listen_port"
    ></el-input-number>
    <el-input
      class="device_form"
      placeholder="Private Key"
      v-model="newDeviceForm.private_key"
    ></el-input>
    <p>Firewall Mark</p>
    <el-input-number
      class="device_form"
      placeholder="Firewall Mark"
      v-model="newDeviceForm.firewall_mark"
    ></el-input-number>
    <el-input
      class="device_form"
      placeholder="List Networks"
      v-model="newDeviceForm.networks"
    ></el-input>
    <div class="modal__buttons">
      <el-button @click="$emit('close')">Cancel</el-button>
      <el-button type="primary" @click="createDevice">Create</el-button>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { deviceApi } from '@/api/interface'
import {DeviceCreateOrUpdateRequest} from "wgrest/models/device-create-or-update-request";
import {emitter} from "@/utils/emmiter";

@Component({
  name: 'DeviceModal'
})
export default class DeviceModal extends Vue {
  private newDeviceForm: DeviceCreateOrUpdateRequest = {
    name: null,
    listen_port: null,
    private_key: null,
    firewall_mark: null,
    networks: null
  }

  public async createDevice(): Promise<void> {
    const savedItem = JSON.parse(JSON.stringify(this.newDeviceForm))

    Object.keys(savedItem).forEach(key => {
      if (!savedItem[key]) {
        delete savedItem[key]
      }
    })

    if(Object.keys(savedItem).includes('networks')) {
      savedItem.networks = savedItem.networks.replace(/\s+/g, '').split(',').filter(Boolean)
    }

    await deviceApi.createDevice(savedItem)

    this.$emit('close')

    emitter.emit('updateDevice')

    this.$message({
      type: 'success',
      message: 'New device create'
    })
  }
}
</script>

<style scoped>
.device_form {
  margin-bottom: 20px;
}

.modal_title {
  text-align: center;
}

.modal__buttons {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>
