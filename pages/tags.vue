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
        icon="search"
        size="is-small"
        v-debounce:300ms="onSearch"
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
      meta: [],
      query: '',
      isAddModalActive: false,
      form: {
        name: '',
        description: '',
      },
    }
  },
  methods: {
    loadAsyncData(filter = `sort=${this.sortField},${this.sortOrder}`) {
      this.loading = true
      this.$axios
        .get(`/tags?${filter}`)
        .then(({ data }) => {
          this.meta = data.meta
          let currentTotal = data.meta.total
          if (data.meta.total / this.perPage > this.filterLimit) {
            currentTotal = this.perPage * this.filterLimit
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
      this.meta.current_page = page
      this.loadAsyncData(
        `page=${page}&sort=${this.sortField},${this.sortOrder}`
      )
    },
    onSort(field, order) {
      this.sortField = field
      this.sortOrder = order
      this.loadAsyncData(
        `page=${this.meta.current_page}&sort=${field},${order}`
      )
    },
    onSearch(query) {
      this.$buefy.toast.open(`Searching data with key word ${query}`)
      this.loadAsyncData(`like[0]=name,${query}&like[1]=description,${query}`)
    },
    onAddTag(form) {
      this.$axios.$post('/tags', form).then(({ data, message }) => {
        this.$buefy.toast.open({
          message: message,
          type: 'is-success',
        })
        this.loadAsyncData()
        this.isAddModalActive = false
      })
    },
  },
  mounted() {
    this.loadAsyncData()
  },
}
</script>

<style scoped>
.field {
  display: flex;
  justify-content: flex-end;
}
</style>
