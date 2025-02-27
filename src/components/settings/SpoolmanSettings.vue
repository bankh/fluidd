<template>
  <div>
    <v-subheader id="spoolman">
      {{ $t('app.spoolman.title.spoolman') }}
    </v-subheader>
    <v-card
      :elevation="5"
      dense
      class="mb-4"
    >
      <app-setting
        :title="$t('app.spoolman.setting.show_spool_selection_dialog_on_print_start')"
      >
        <v-switch
          v-model="autoSpoolSelectionDialog"
          hide-details
          class="mt-0 mb-4"
        />
      </app-setting>

      <v-divider />
      <app-setting
        :title="$tc('app.spoolman.setting.auto_open_qr_camera')"
      >
        <v-select
          v-model="autoOpenQRDetectionCameraId"
          filled
          dense
          single-line
          hide-details="auto"
          :items="supportedCameras"
        />
      </app-setting>

      <v-divider />
      <app-setting
        :title="$t('app.spoolman.setting.prefer_device_camera')"
      >
        <v-switch
          v-model="preferDeviceCamera"
          hide-details
          class="mt-0 mb-4"
        />
      </app-setting>

      <v-divider />
      <app-setting
        :title="$t('app.spoolman.setting.auto_select_spool_on_match')"
      >
        <v-switch
          v-model="autoSelectSpoolOnMatch"
          hide-details
          class="mt-0 mb-4"
        />
      </app-setting>

      <v-divider />
      <app-setting
        :title="$t('app.spoolman.setting.warn_on_not_enough_filament')"
      >
        <v-switch
          v-model="warnOnNotEnoughFilament"
          hide-details
          class="mt-0 mb-4"
        />
      </app-setting>

      <v-divider />
      <app-setting
        :title="$t('app.spoolman.setting.warn_on_filament_type_mismatch')"
      >
        <v-switch
          v-model="warnOnFilamentTypeMismatch"
          hide-details
          class="mt-0 mb-4"
        />
      </app-setting>

      <v-divider />
      <app-setting :title="$t('app.setting.label.reset')">
        <app-btn
          outlined
          small
          color="primary"
          @click="handleReset"
        >
          {{ $t('app.setting.btn.reset') }}
        </app-btn>
      </app-setting>
    </v-card>
  </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import { defaultState } from '@/store/config/state'
import StateMixin from '@/mixins/state'
import type { CameraConfig } from '@/store/cameras/types'

@Component({
  components: {}
})
export default class SpoolmanSettings extends Mixins(StateMixin) {
  get autoSpoolSelectionDialog (): boolean {
    return this.$store.state.config.uiSettings.spoolman.autoSpoolSelectionDialog
  }

  set autoSpoolSelectionDialog (value: boolean) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.autoSpoolSelectionDialog',
      value,
      server: true
    })
  }

  get supportedCameras () {
    return [
      { text: this.$tc('app.setting.label.none', 0), value: null },
      ...this.$store.getters['cameras/getEnabledCameras']
        .map((camera: CameraConfig) => ({ text: camera.name, value: camera.id, disabled: !camera.enabled || camera.service === 'iframe' }))
    ]
  }

  get autoOpenQRDetectionCameraId (): string {
    return this.$store.state.config.uiSettings.spoolman.autoOpenQRDetectionCamera
  }

  set autoOpenQRDetectionCameraId (value: string) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.autoOpenQRDetectionCamera',
      value,
      server: true
    })
  }

  get preferDeviceCamera () {
    return this.$store.state.config.uiSettings.spoolman.preferDeviceCamera
  }

  set preferDeviceCamera (value: boolean) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.preferDeviceCamera',
      value,
      server: true
    })
  }

  get autoSelectSpoolOnMatch () {
    return this.$store.state.config.uiSettings.spoolman.autoSelectSpoolOnMatch
  }

  set autoSelectSpoolOnMatch (value: boolean) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.autoSelectSpoolOnMatch',
      value,
      server: true
    })
  }

  get warnOnNotEnoughFilament () {
    return this.$store.state.config.uiSettings.spoolman.warnOnNotEnoughFilament
  }

  set warnOnNotEnoughFilament (value: boolean) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.warnOnNotEnoughFilament',
      value,
      server: true
    })
  }

  get warnOnFilamentTypeMismatch () {
    return this.$store.state.config.uiSettings.spoolman.warnOnFilamentTypeMismatch
  }

  set warnOnFilamentTypeMismatch (value: boolean) {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman.warnOnFilamentTypeMismatch',
      value,
      server: true
    })
  }

  handleReset () {
    this.$store.dispatch('config/saveByPath', {
      path: 'uiSettings.spoolman',
      value: defaultState().uiSettings.spoolman,
      server: true
    })
  }
}
</script>
