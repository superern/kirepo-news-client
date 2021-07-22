<template>
  <section class="section">
    <div class="columns is-mobile">
      <card
        :title="article.title"
        :content="article.content"
        v-for="article in articles"
        :key="article.id"
        :is_published="article.is_published"
      >
        {{ article.published_at }}
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
