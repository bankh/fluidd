<template>
  <v-layout class="console-item">
    <span
      v-if="value.time"
      class="secondary--text mr-3 d-none d-sm-block"
    >
      {{ itemTime }}&nbsp;
    </span>
    <span
      :class="itemClass"
      @click.capture="itemClick"
      v-html="itemMessage"
    />
  </v-layout>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { Globals } from '@/globals'
import type { ConsoleEntry } from '@/store/console/types'

@Component({})
export default class ConsoleItem extends Vue {
  @Prop({ type: Object, default: {} })
  readonly value!: ConsoleEntry

  get knownCommands () {
    const availableCommands = this.$store.getters['console/getAllGcodeCommands']
    return new Set(Object.keys(availableCommands))
  }

  get itemMessage () {
    let message = this.value.message
    if (this.value.type === 'response') {
      message = this.value.message.replace(/([A-Z0-9_]{2,})/gm, (match, command) => {
        if (this.knownCommands.has(command)) return `<a class="primary--text text--lighten-1">${command.toUpperCase()}</a>`
        return match
      })
    }
    return (this.value.type === 'command')
      ? `${Globals.CONSOLE_SEND_PREFIX}<a class="primary--text text--lighten-1">${message}</a>`
      : message
  }

  get itemTime () {
    return (this.value.time)
      ? this.$filters.formatTimeWithSeconds(this.value.time * 1000)
      : ''
  }

  get itemClass () {
    if (this.value.message.startsWith('!!')) {
      return { 'error--text': true }
    }

    // if (this.value.message.startsWith('//')) {
    //   return { 'secondary--text': true }
    // }

    if (this.value.type === 'command') {
      return { 'primary--text': true }
    }

    return { 'secondary--text': true }
  }

  itemClick (event: Event) {
    if (event.target instanceof HTMLAnchorElement) {
      const command = event.target.innerHTML
        .replace(/<br>/g, '\n')
        .replace(/^\s+|\s+$/gm, '')

      this.$emit('click', command)
    }
  }
}
</script>

<style lang="scss" scoped>
  .console-item {
    flex: 0 0 auto;
  }
</style>
