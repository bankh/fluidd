<template>
  <div class="logo-wrapper">
    <inline-svg
      :src="logoSrc"
      @loaded="handleLogoLoad"
    />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Watch } from 'vue-property-decorator'

@Component({})
export default class AppIcon extends Vue {
  @Watch('theme')
  onTheme () {
    this.setData()
  }

  get theme () {
    return this.$store.getters['config/getTheme']
  }

  get logoSrc () {
    return `${import.meta.env.BASE_URL}${this.theme.logo.src}`
  }

  handleLogoLoad () {
    this.setData()
  }

  setData () {
    const supportedQueries = [
      { query: '.color-primary', color: this.theme.currentTheme.primary },
      { query: '.color-primary-offset', color: this.theme.currentTheme.primaryOffset }
    ]

    // If this logo should not change with theme, then set it based on
    // our base theme being dark or light.
    if (!this.theme.logo.dynamic) {
      supportedQueries.forEach((query) => {
        const dark = this.theme.logo.colorDark || '#000000'
        const light = this.theme.logo.colorLight || '#ffffff'
        query.color = (this.theme.isDark) ? light : dark
      })
    }

    supportedQueries.forEach((query) => {
      this.$el.querySelectorAll(query.query).forEach((e) => {
        if (
          e.tagName === 'path' ||
          e.tagName === 'polygon' ||
          e.tagName === 'g'
        ) {
          const path = e as SVGPathElement
          path.style.fill = query.color
        }
      })
    })
  }
}
</script>

<style lang="scss" scoped>
  .logo-wrapper {
    display: flex;
  }
</style>
