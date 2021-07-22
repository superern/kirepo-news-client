<template>
  <section class="m-5">
    <b-field name="Search">
      <b-input
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
export default {
  name: 'tags',
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
    }
  },
  methods: {
    loadAsyncData(url) {
      this.loading = true
      this.$axios
        .get(url)
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
      this.meta.page = page
      this.loadAsyncData(
        `/tags?page=${page}&sort=${this.sortField},${this.sortOrder}`
      )
    },
    onSort(field, order) {
      this.loadAsyncData(`/tags?page=${this.meta.page}&sort=${field},${order}`)
    },
    onSearch(query) {
      this.$buefy.toast.open(`Searching data with key word ${query}`)
      this.loadAsyncData(
        `/tags?like[0]=name,${query}&like[1]=description,${query}`
      )
    },
  },
  mounted() {
    this.loadAsyncData(`/tags?sort=${this.sortField},${this.sortOrder}`)
  },
}
</script>

<style scoped></style>
