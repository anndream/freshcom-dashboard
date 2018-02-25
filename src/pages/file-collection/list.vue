<template>
<div class="page-wrapper">
  <div>
    <el-menu :router="true" default-active="/file_collections" mode="horizontal" class="secondary-nav">
      <el-menu-item :route="{ name: 'ListFile' }" index="/files">
        Files
      </el-menu-item>
      <el-menu-item :route="{ name: 'ListFileCollection' }" index="/file_collections">
        Collections
      </el-menu-item>
    </el-menu>
    <locale-selector @change="searchFileCollection()" class="pull-right"></locale-selector>
  </div>

  <div>
    <el-card class="main-card">
      <div slot="header">
        <div v-if="isViewingTestData" class="test-data-banner">
          <div class="banner-content">TEST DATA</div>
        </div>

        <el-button plain size="small"><icon name="filter" scale="0.7" class="v-middle"></icon> Filter</el-button>

        <div class="search">
          <el-input :value="searchKeyword" @input="updateSearchKeyword" placeholder="Search..." size="small">
            <template slot="prepend"><icon name="search" scale="1" class="v-middle"></icon></template>
          </el-input>
        </div>

        <el-button @click="newFileCollection()" plain size="small" class="pull-right">
          <icon name="plus" scale="0.7" class="v-middle"></icon> New
        </el-button>
      </div>

      <div class="data full" v-loading="isLoading">
        <p v-if="noSearchResult" class="search-notice text-center">
          There is no result that matches "{{searchKeyword}}"
        </p>
        <el-table v-if="hasSearchResult" @row-click="viewFileCollection" :data="fileCollections" class="full">
          <el-table-column prop="name" label="Name" width="200"></el-table-column>
          <el-table-column prop="label" label="Label" width="200"></el-table-column>
          <el-table-column prop="id" label="ID"></el-table-column>
        </el-table>

        <div v-if="hasSearchResult" class="footer">
          <span class="total">around {{totalCount}} results</span>
          <pagination :number="page.number" :size="page.size" :total="totalCount"></pagination>
        </div>
      </div>
    </el-card>
  </div>
</div>
</template>

<script>
import 'vue-awesome/icons/search'

import _ from 'lodash'
import freshcom from '@/freshcom-sdk'

import PageMixin from '@/mixins/page'
import Pagination from '@/components/pagination'
import { idLastPart } from '@/helpers/filters'

export default {
  name: 'ListFileCollection',
  mixins: [PageMixin],
  components: {
    Pagination
  },
  filters: {
    idLastPart
  },
  props: {
    searchKeyword: {
      type: String,
      default: ''
    },
    page: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      fileCollections: [],
      isLoading: false,
      allCount: 0,
      totalCount: 0,

      errors: {}
    }
  },
  created () {
    this.searchFileCollection()
  },
  computed: {
    noSearchResult () {
      return !this.isLoading && this.totalCount === 0 && this.allCount > 0
    },
    hasSearchResult () {
      return !this.isLoading && this.totalCount !== 0
    },
    currentRoutePath () {
      return this.$store.state.route.fullPath
    }
  },
  watch: {
    isViewingTestData () {
      this.searchFileCollection()
    },
    searchKeyword (newKeyword) {
      this.searchFileCollection()
    },
    page (newPage, oldPage) {
      if (_.isEqual(newPage, oldPage)) {
        return
      }
      this.searchFileCollection()
    }
  },
  methods: {
    updateSearchKeyword: _.debounce(function (newSearchKeyword) {
      let q = _.merge({}, _.omit(this.$route.query, ['page[number]']), { search: newSearchKeyword })
      this.$router.replace({ name: this.$store.state.route.name, query: q })
    }, 300),

    searchFileCollection () {
      this.isLoading = true

      freshcom.listFileCollection({
        search: this.searchKeyword,
        page: this.page
      }).then(response => {
        this.fileCollections = response.data
        this.allCount = response.meta.allCount
        this.totalCount = response.meta.totalCount

        this.isLoading = false
      }).catch(errors => {
        this.isLoading = false
      })
    },

    viewFileCollection (efc) {
      this.$store.dispatch('pushRoute', { name: 'ShowFileCollection', params: { id: efc.id, callbackPath: this.currentRoutePath } })
    },

    newFileCollection () {
      this.$store.dispatch('pushRoute', { name: 'NewFileCollection' })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main-card .footer {
  text-align: right;
  border-top: 0;
}
.total {
  float: left;
  display: inline-block;
  font-size: 13px;
  min-width: 28px;
  height: 28px;
  line-height: 28px;
}
</style>