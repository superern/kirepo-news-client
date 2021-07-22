<template>
  <section class="section">
    <div class="columns is-mobile" style="display: flex; flex-wrap: wrap">
      <card
        :title="article.title"
        :content="article.content"
        v-for="article in articles"
        :key="article.id"
        :is_published="article.is_published"
      >
        {{ $moment(article.published_at).fromNow() }}
      </card>
    </div>
  </section>
</template>

<script>
import Card from '~/components/Card'

export default {
  name: 'HomePage',

  components: {
    Card,
  },
  data() {
    return {
      articles: [],
    }
  },
  mounted() {
    this.$axios.get('/news?is_published=1').then(({ data }) => {
      this.articles = data.data
    })
  },
}
</script>
