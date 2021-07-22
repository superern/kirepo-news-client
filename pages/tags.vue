<template>
  <section class="m-5">
    <b-field>
      <b-button
        class="add-button"
        label="Create Tag"
        type="is-primary"
        size="is-small"
        @click="isAddModalActive = true"
      />
    </b-field>
    <b-modal
      v-model="isAddModalActive"
      has-modal-card
      full-screen
      :destroy-on-hide="true"
      :can-cancel="false"
      aria-role="dialog"
      aria-label="Create Tag"
      aria-modal
    >
      <template #default="props">
        <modal-form
          v-bind="form"
          @close="props.close"
          @onSubmit="onAddTag"
          action-type="Add"
        ></modal-form>
      </template>
    </b-modal>
    <b-field name="Search">
      <b-input
        style="flex: 1"
        v-model="query"
        placeholder="Search..."
        icon="magnify"
        size="is-medium"
      />
    </b-field>
    <b-table
      :data="data"
      ref="table"
      :loading="loading"
      paginated
      backend-pagination
      :total="meta.total"
      :per-page="meta.per_page"
      :current-page="meta.current_page"
      @page-change="onPageChange"
      aria-next-label="Next page"
      aria-previous-label="Previous page"
      aria-page-label="Page"
      aria-current-label="Current page"
      backend-sorting
      :default-sort-direction="defaultSortOrder"
      :default-sort="[sortField, sortOrder]"
      @sort="onSort"
      detailed
      detail-key="id"
      :detail-transition="transitionName"
      @details-open="(row) => $buefy.toast.open(`Showing ${row.name}`)"
      :show-detail-icon="showDetailIcon"
    >
      <!-- ID -->
      <b-table-column
        field="id"
        label="ID"
        width="40"
        sortable
        numeric
        v-slot="props"
      >
        {{ props.row.id }}
      </b-table-column>

      <!-- name -->
      <b-table-column field="name" label="Name" sortable v-slot="props">
        {{ props.row.name }}
      </b-table-column>

      <!-- Created At-->
      <b-table-column
        field="created_at"
        label="Created At"
        sortable
        centered
        v-slot="props"
      >
        <span class="tag is-success">
          {{ $moment(props.row.created_at).fromNow() }}
        </span>
      </b-table-column>

      <!-- Updated At-->
      <b-table-column
        field="updated_at"
        label="Updated At"
        sortable
        centered
        v-slot="props"
      >
        <span class="tag is-success">
          {{ $moment(props.row.updated_at).fromNow() }}
        </span>
      </b-table-column>

      <template #detail="props">
        <article class="media">
          <div class="media-content">
            <div class="content">
              <p>
                <strong>{{ props.row.name }} </strong>
                <br />
                {{ props.row.description }}
              </p>
            </div>
          </div>
        </article>
      </template>
    </b-table>
  </section>
</template>

<script>
import ModalForm from '../components/TagModalForm'
import { debounce } from 'vue-debounce'

export default {
  name: 'tags',
  components: {
    ModalForm,
  },
  computed: {
    transitionName() {
      if (this.useTransition) {
        return 'fade'
      }
    },
    sortFilter() {
      return `sort=${this.sortField},${this.sortOrder}`
    },
  },
  data() {
    return {
      data: [],
      loading: false,
      defaultOpenedDetails: [1],
      showDetailIcon: true,
      useTransition: true,
      sortField: 'updated_at',
      sortOrder: 'desc',
      defaultSortOrder: 'desc',
      filterLimit: 20,
      meta: {
        per_page: 5,
        current_page: 1,
        total: 0,
      },
      query: '',
      isAddModalActive: false,
      form: {
        name: '',
        description: '',
      },
      filter: '',
    }
  },
  methods: {
    loadAsyncData() {
      this.loading = true
      this.$axios
        .get(`/tags?${this.filter}`)
        .then(({ data }) => {
          this.meta.per_page = data.meta.per_page

          let currentTotal = data.meta.total
          if (data.meta.total / data.meta.per_page > this.filterLimit) {
            currentTotal = data.meta.per_page * this.filterLimit
          }

          this.meta.total = currentTotal
          this.data = data.data
          this.loading = false
        })
        .catch((error) => {
          this.data = []
          this.meta.total = 0
          this.loading = false
          throw error
        })
    },
    onPageChange(page) {
      let currentPage = `&page=${page}`,
        pageQuery = `&page=${this.meta.current_page}`

      this.filter =
        this.filter.search(pageQuery) === -1
          ? this.filter + currentPage
          : this.filter.replace(pageQuery, currentPage)

      console.log(this.filter)
      this.meta.current_page = page

      this.loadAsyncData()
    },
    onSort(field, order) {
      console.log('sorting')
      let currentQuery = `sort=${field},${order}`

      this.filter = this.filter.replace(this.sortFilter, currentQuery)
      this.sortField = field
      this.sortOrder = order

      this.loadAsyncData()
    },
    onSearch: debounce((vm, query) => {
      if (!query) {
        // user clear the filters
        vm.filter = vm.sortFilter
        vm.meta.current_page = 1
      } else {
        vm.$buefy.toast.open(`Searching data with key word ${query}`)
        vm.filter += `&like[0]=name,${query}&like[1]=description,${query}`
      }
      vm.loadAsyncData()
    }, 300),
    onAddTag(form) {
      this.$axios.$post('/tags', form).then(({ data, message }) => {
        this.$buefy.toast.open({
          message: message,
          type: 'is-success',
        })
        this.filter = `${this.sortFilter}`
        this.loadAsyncData()
        this.isAddModalActive = false
      })
    },
  },
  mounted() {
    this.filter = this.sortFilter
    this.loadAsyncData()
  },
  watch: {
    query: {
      handler(newValue, oldValue) {
        this.onSearch(this, newValue)
      },
    },
  },
}
</script>

<style scoped>
.field {
  display: flex;
  justify-content: flex-end;
}
</style>
