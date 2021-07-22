<template>
  <section class="m-5">
    <b-field>
      <b-button
        class="add-button"
        label="Create Article"
        type="is-primary"
        size="is-small"
        @click="isAddModalActive = true"
      />
    </b-field>
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
      @details-open="(row) => $buefy.toast.open(`Expanded ${row.title}`)"
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

      <!-- Title -->
      <b-table-column field="title" label="Title" sortable v-slot="props">
        {{ props.row.title }}
      </b-table-column>

      <!-- Published At-->
      <b-table-column
        field="published_at"
        label="Published At"
        sortable
        centered
        v-slot="props"
      >
        <span class="tag is-success">
          {{
            props.row.published_at
              ? $moment(props.row.published_at).fromNow()
              : 'Not yet published'
          }}
        </span>
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
          <figure class="media-left">
            <p class="image is-64x64">
              <img src="https://via.placeholder.com/150" />
            </p>
          </figure>
          <div class="media-content">
            <div class="content">
              <p>
                <strong>{{ props.row.title }} </strong>
                <br />
                {{ props.row.content }}
              </p>

              <!-- Update Modal Button -->
              <b-button
                class="update-button"
                label="Update"
                type="is-primary"
                size="is-small"
                @click="showUpdateModal(props.row)"
              />
            </div>
          </div>
        </article>
      </template>
    </b-table>

    <!-- Add Modal -->
    <b-modal
      v-model="isAddModalActive"
      has-modal-card
      full-screen
      :destroy-on-hide="true"
      :can-cancel="false"
      aria-role="dialog"
      aria-label="Create Article"
      aria-modal
    >
      <template #default="props">
        <modal-form
          v-bind="form"
          @close="onCancelModal(props)"
          @onSubmit="onAddArticle"
          action-type="Add"
        ></modal-form>
      </template>
    </b-modal>

    <!-- Update Modal -->
    <b-modal
      v-model="isUpdateModalActive"
      has-modal-card
      full-screen
      :destroy-on-hide="true"
      :can-cancel="false"
      aria-role="dialog"
      aria-label="Update Article"
      aria-modal
    >
      <template #default="props">
        <modal-form
          v-bind="form"
          @close="onCancelModal(props)"
          @onSubmit="onUpdateArticle"
          action-type="Update"
        ></modal-form>
      </template>
    </b-modal>
  </section>
</template>

<script>
import { debounce } from 'vue-debounce'
import ModalForm from '../components/ArticleModalForm'

export default {
  name: 'news',
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
      filter: '',
      isAddModalActive: false,
      isUpdateModalActive: false,
      form: {
        title: '',
        content: '',
        is_published: false,
      },
    }
  },
  methods: {
    loadAsyncData() {
      this.loading = true
      this.$axios
        .get(`/news?${this.filter}`)
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
        vm.filter += `&like[0]=title,${query}&like[1]=content,${query}`
      }
      vm.loadAsyncData()
    }, 300),
    onAddArticle(form) {
      this.$axios
        .$post('/news', form)
        .then(({ data, message }) => {
          this.$buefy.toast.open({
            message: message,
            type: 'is-success',
          })
        })
        .catch((err) => {
          console.log(err.response)
          this.$buefy.toast.open({
            message: 'Failed to create your Article',
            type: 'is-danger',
          })
        })
        .finally(() => {
          this.form = {}
          this.filter = `${this.sortFilter}`
          this.loadAsyncData()
          this.isAddModalActive = false
        })
    },
    showUpdateModal(form) {
      this.form = form
      this.isUpdateModalActive = true
    },
    onUpdateArticle(form) {
      this.$axios
        .$put(`/news/${this.form.id}`, form)
        .then(({ data, message }) => {
          this.$buefy.toast.open({
            message: message,
            type: 'is-success',
          })
        })
        .catch((err) => {
          console.log(err.response)
          this.$buefy.toast.open({
            message: 'Failed to update your Article',
            type: 'is-danger',
          })
        })
        .finally(() => {
          this.form = {}
          this.filter = `${this.sortFilter}`
          this.loadAsyncData()
          this.isUpdateModalActive = false
        })
    },
    onCancelModal(props) {
      props.close()
      this.form = {}
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
