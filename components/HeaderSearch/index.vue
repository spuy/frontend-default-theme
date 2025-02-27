<template>
  <div :class="{'show':show}" class="header-search">
    <svg-icon class-name="search-icon" icon-class="search" @click.stop="click" />
    <el-select
      ref="headerSearchSelect"
      v-model="search"
      :remote-method="querySearch"
      filterable
      default-first-option
      remote
      :popper-class="styleInput"
      class="header-search-select"
      :placeholder="$t('table.dataTable.search')"
      @focus="openSearch"
      @change="change"
    >
      <el-option
        v-for="element in options"
        :key="element.item.path"
        :value="element.item"
        :label="element.item.title.join(' > ')"
      >
        <span v-if="isMobile">
          <svg-icon :icon-class="element.item.meta.icon" />
          {{ element.item.title.join(' > ') }}
        </span>
        <span v-else>
          {{ element.item.title.join(' > ') }}
          <svg-icon :icon-class="element.item.meta.icon" />
        </span>
      </el-option>
    </el-select>
  </div>
</template>

<script>
// fuse is a lightweight fuzzy-search module
// make search results more in line with expectations
import Fuse from 'fuse.js'
import path from 'path'
import { generateTitle } from '@/utils/i18n'
// import i18n from '@/lang'

export default {
  name: 'HeaderSearch',
  data() {
    return {
      search: '',
      options: [],
      searchPool: [],
      show: false,
      fuse: undefined
    }
  },
  computed: {
    isMobile() {
      return this.$store.state.app.device === 'mobile'
    },
    routes() {
      return this.$store.getters.permission_routes
    },
    lang() {
      return this.$store.getters.language
    },
    supportPinyinSearch() {
      return this.$store.state.settings.supportPinyinSearch
    },
    styleInput() {
      if (this.isMobile) {
        return 'search-menu-mobile'
      }
      return ''
    }
  },
  watch: {
    lang() {
      this.searchPool = this.generateRoutes(this.routes)
    },
    routes() {
      this.searchPool = this.generateRoutes(this.routes)
    },
    searchPool(list) {
      // Support pinyin search
      if (this.lang === 'zh' && this.supportPinyinSearch) {
        this.addPinyinField(list)
      }
      this.initFuse(list)
    },
    show(value) {
      if (value) {
        document.body.addEventListener('click', this.close)
      } else {
        document.body.removeEventListener('click', this.close)
      }
    }
  },
  mounted() {
    this.searchPool = this.generateRoutes(this.routes)
  },
  methods: {
    openSearch() {
      if (this.show) {
        this.querySearch('document')
      }
    },

    generateTitle,
    async addPinyinField(list) {
      const { default: pinyin } = await import('pinyin')
      if (Array.isArray(list)) {
        list.forEach(element => {
          const title = element.title
          if (Array.isArray(title)) {
            title.forEach(v => {
              v = pinyin(v, {
                style: pinyin.STYLE_NORMAL
              }).join('')
              element.pinyinTitle = v
            })
          }
        })
        return list
      }
    },
    click() {
      this.show = !this.show
      if (this.show) {
        this.$refs.headerSearchSelect && this.$refs.headerSearchSelect.focus()
      }
    },
    close() {
      this.$refs.headerSearchSelect && this.$refs.headerSearchSelect.blur()
      this.options = []
      this.show = false
    },
    change(val) {
      if (val.name) {
        const query = {}

        this.$router.push({
          name: val.name,
          params: {
            childs: val.meta.childs
          },
          query
        }, () => {})
      } else {
        this.$router.push({
          path: val.path
        }, () => {})
      }
      this.search = ''
      this.options = []
      this.$nextTick(() => {
        this.show = false
      })
    },
    initFuse(list) {
      this.fuse = new Fuse(list, {
        shouldSort: true,
        threshold: 0.4,
        location: 0,
        distance: 100,
        maxPatternLength: 32,
        minMatchCharLength: 1,
        keys: [{
          name: 'title',
          weight: 0.7
        }, {
          name: 'pinyinTitle',
          weight: 0.3
        }, {
          name: 'path',
          weight: 0.3
        }]
      })
    },
    // Filter out the routes that can be displayed in the sidebar
    // And generate the internationalized title
    generateRoutes(routes, basePath = '/', prefixTitle = []) {
      let res = []
      for (const router of routes) {
        // skip hidden router
        // if (router.meta && router.meta.isIndex) { continue }
        const data = {
          path: path.resolve(basePath, router.path),
          title: [...prefixTitle],
          meta: router.meta,
          name: router.name
        }
        if (router.meta && router.meta.title) {
          // generate internationalized title
          const i18ntitle = this.generateTitle(router.meta.title)
          data.title = [...data.title, i18ntitle]
          if (router.redirect !== 'noRedirect' && router.name !== 'Report Viewer' && !router.meta.isIndex) {
            // only push the routes with title
            // special case: need to exclude parent router without redirect
            res.push(data)
          }
        }
        // recursive child routes
        if (router.children) {
          const tempRoutes = this.generateRoutes(router.children, data.path, data.title)
          if (tempRoutes.length >= 1) {
            res = [...res, ...tempRoutes]
          }
        }
      }
      return res
    },
    querySearch(query) {
      if (query !== '') {
        this.options = this.fuse.search(query)
        this.options.map(menu => {
          if (this.isMobile && menu.item.title.length > 0) {
            return {
              ...menu,
              title: menu.item.title.reverse()
            }
          }
          return menu
        })
      } else {
        this.options = []
      }
    }
  }
}
</script>
<style>
.search-menu-mobile {
  position: absolute;
  top: 46px;
  left: -444px;
  transform-origin: center top;
  z-index: 2002;
  min-width: 150px;
  max-width: 350px;
}
.search-scroll-mobile {
  max-width: 350px;
}
</style>

<style lang="scss" scoped>
.header-search {
  font-size: 0 !important;
  .search-icon {
    cursor: pointer;
    font-size: 18px;
    vertical-align: middle;
  }
  .header-search-select {
    font-size: 18px;
    transition: width 0.2s;
    width: 0;
    overflow: hidden;
    background: transparent;
    border-radius: 0;
    display: inline-block;
    vertical-align: middle;

    ::v-deep .el-input__inner {
      border-radius: 0;
      border: 0;
      padding-left: 0;
      padding-right: 0;
      box-shadow: none !important;
      border-bottom: 1px solid #d9d9d9;
      vertical-align: middle;
    }
  }
  &.show {
    .header-search-select {
      width: 150px;
      margin-left: 10px;
    }
  }
}
</style>
