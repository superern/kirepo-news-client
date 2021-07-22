<template>
  <div class="column is-one-third" v-if="is_published">
    <div class="card">
      <header class="card-header">
        <p class="card-header-title has-text-grey">
          {{ title }}
        </p>
        <div class="mx-1 my-2" v-for="tag in tags" :key="tag.id">
          <b-tag type="is-primary is-primary">
            {{ tag.name }}
          </b-tag>
        </div>
      </header>
      <div class="card-content">
        <div class="content has-text-centered">
          <div v-html="content"></div>
        </div>
      </div>
      <footer class="card-footer">
        <div class="card-footer-item">
          <span>
            <slot />
          </span>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      required: true,
    },
    content: {
      type: String,
      required: true,
    },
    is_published: {
      type: Boolean,
    },
    tags: {
      type: Array,
    },
  },
}
</script>
