<template lang='pug'>
  v-container(fluid, grid-list-lg)
    v-layout(row wrap)
      v-flex(xs12)
        .admin-header
          img.animated.fadeInUp(src='/svg/icon-paint-palette.svg', alt='Theme', style='width: 80px;')
          .admin-header-title
            .headline.primary--text.animated.fadeInLeft {{$t('admin:theme.title')}}
            .subheading.grey--text.animated.fadeInLeft.wait-p2s {{$t('admin:theme.subtitle')}}
          v-spacer
          v-btn.animated.fadeInRight(color='success', depressed, @click='save', large, :loading='loading')
            v-icon(left) check
            span {{$t('common:actions.apply')}}
        v-form.pt-3
          v-layout(row wrap)
            v-flex(lg6 xs12)
              v-card.wiki-form.animated.fadeInUp
                v-toolbar(color='primary', dark, dense, flat)
                  v-toolbar-title
                    .subheading {{$t('admin:theme.title')}}
                v-card-text
                  v-select(
                    :items='themes'
                    outline
                    background-color='grey lighten-2'
                    prepend-icon='palette'
                    v-model='config.theme'
                    :label='$t(`admin:theme.siteTheme`)'
                    persistent-hint
                    :hint='$t(`admin:theme.siteThemeHint`)'
                    )
                    template(slot='item', slot-scope='data')
                      v-list-tile-avatar
                        v-icon.blue--text(dark) filter_frames
                      v-list-tile-content
                        v-list-tile-title(v-html='data.item.text')
                        v-list-tile-sub-title(v-html='data.item.author')
                  v-select.mt-3(
                    :items='iconsets'
                    outline
                    background-color='grey lighten-2'
                    prepend-icon='pets'
                    v-model='config.iconset'
                    :label='$t(`admin:theme.iconset`)'
                    persistent-hint
                    :hint='$t(`admin:theme.iconsetHint`)'
                    )
                  v-divider.mt-3
                  v-switch(
                    v-model='darkMode'
                    :label='$t(`admin:theme.darkMode`)'
                    color='primary'
                    persistent-hint
                    :hint='$t(`admin:theme.darkModeHint`)'
                    )

              v-card.wiki-form.mt-3.animated.fadeInUp.wait-p2s
                v-toolbar(color='primary', dark, dense, flat)
                  v-toolbar-title
                    .subheading {{$t(`admin:theme.codeInjection`)}}
                v-card-text
                  v-textarea(
                    v-model='config.injectCSS'
                    :label='$t(`admin:theme.cssOverride`)'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    :hint='$t(`admin:theme.cssOverrideHint`)'
                    auto-grow
                    )
                  i18next.caption.pl-2.ml-1(path='admin:theme.cssOverrideWarning', tag='div')
                    strong.red--text(place='caution') {{$t('admin:theme.cssOverrideWarningCaution')}}
                    code(place='cssClass') .contents
                  v-textarea.mt-3(
                    v-model='config.injectHead'
                    :label='$t(`admin:theme.headHtmlInjection`)'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    :hint='$t(`admin:theme.headHtmlInjectionHint`)'
                    auto-grow
                    )
                  v-textarea.mt-2(
                    v-model='config.injectBody'
                    :label='$t(`admin:theme.bodyHtmlInjection`)'
                    outline
                    background-color='grey lighten-1'
                    color='primary'
                    persistent-hint
                    :hint='$t(`admin:theme.bodyHtmlInjectionHint`)'
                    auto-grow
                    )
            v-flex(lg6 xs12)
              v-card.animated.fadeInUp.wait-p2s
                v-toolbar(color='teal', dark, dense, flat)
                  v-toolbar-title
                    .subheading {{$t('admin:theme.downloadThemes')}}
                  v-spacer
                  v-chip(label, color='white', small).teal--text coming soon
                v-data-table(
                  :headers='headers',
                  :items='themes',
                  hide-actions,
                  item-key='value',
                  :rows-per-page-items='[-1]'
                )
                  template(v-slot:items='thm')
                    td
                      strong {{thm.item.text}}
                    td
                      span {{ thm.item.author }}
                    td.text-xs-center
                      v-progress-circular(v-if='thm.item.isDownloading', indeterminate, color='blue', size='20', :width='2')
                      v-btn(v-else-if='thm.item.isInstalled && thm.item.installDate < thm.item.updatedAt', icon)
                        v-icon.blue--text cached
                      v-btn(v-else-if='thm.item.isInstalled', icon)
                        v-icon.green--text check
                      v-btn(v-else, icon)
                        v-icon.grey--text cloud_download
</template>

<script>
import _ from 'lodash'
import { sync } from 'vuex-pathify'

import themeConfigQuery from 'gql/admin/theme/theme-query-config.gql'
import themeSaveMutation from 'gql/admin/theme/theme-mutation-save.gql'

export default {
  data() {
    return {
      loading: false,
      themes: [
        { text: 'Default', author: 'requarks.io', value: 'default', isInstalled: true, installDate: '', updatedAt: '' }
      ],
      iconsets: [
        { text: 'Material Icons (default)', value: 'md' },
        { text: 'Material Design Icons', value: 'mdi' },
        { text: 'Font Awesome 5', value: 'fa' },
        { text: 'Font Awesome 4', value: 'fa4' },
      ],
      config: {
        theme: 'default',
        darkMode: false,
        iconset: '',
        injectCSS: '',
        injectHead: '',
        injectBody: ''
      },
      darkModeInitial: false
    }
  },
  computed: {
    darkMode: sync('site/dark'),
    headers() {
      return [
        {
          text: this.$t('admin:theme.downloadName'),
          align: 'left',
          value: 'text'
        },
        {
          text: this.$t('admin:theme.downloadAuthor'),
          align: 'left',
          value: 'author'
        },
        {
          text: this.$t('admin:theme.downloadDownload'),
          align: 'center',
          value: 'value',
          sortable: false,
          width: 100
        }
      ]
    }
  },
  mounted() {
    this.darkModeInitial = this.darkMode
  },
  beforeDestroy() {
    this.darkMode = this.darkModeInitial
  },
  methods: {
    async save () {
      this.loading = true
      this.$store.commit(`loadingStart`, 'admin-theme-save')
      try {
        const respRaw = await this.$apollo.mutate({
          mutation: themeSaveMutation,
          variables: {
            theme: this.config.theme,
            iconset: this.config.iconset,
            darkMode: this.darkMode,
            injectCSS: this.config.injectCSS,
            injectHead: this.config.injectHead,
            injectBody: this.config.injectBody
          }
        })
        const resp = _.get(respRaw, 'data.theming.setConfig.responseResult', {})
        if (resp.succeeded) {
          this.darkModeInitial = this.darkMode
          this.$store.commit('showNotification', {
            message: 'Theme settings updated successfully.',
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-theme-save')
      this.loading = false
    }
  },
  apollo: {
    config: {
      query: themeConfigQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.theming.config,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-theme-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
