<template>
  <section class="m-5">
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
              <img src="/static/img/placeholder-128x128.png" />
            </p>
          </figure>
          <div class="media-content">
            <div class="content">
              <p>
                <strong>{{ props.row.title }} </strong>
                <br />
                {{ props.content }}
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
  name: 'news',
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
      this.loadAsyncData(`/news?page=${page}&sort=updated_at,desc`)
    },
    onSort(field, order) {
      this.loadAsyncData(`/news?page=${this.meta.page}&sort=${field},${order}`)
    },
  },
  mounted() {
    this.loadAsyncData('/news?sort=updated_at,desc')
  },
}
</script>

<style scoped></style>
