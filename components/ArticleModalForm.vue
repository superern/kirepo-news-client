<template>
  <form action="">
    <div class="modal-card" style="width: auto; height: 100vh">
      <header class="modal-card-head">
        <p class="modal-card-title">Article</p>
        <button type="button" class="delete" @click="$emit('close')" />
      </header>
      <section class="modal-card-body">
        <b-field label="Title">
          <b-input
            type="text"
            v-model="form.title"
            placeholder="Article Title"
            required
          >
          </b-input>
        </b-field>
        <client-only>
          <tiptap v-model="form.content" class="my-5 input wysiwyg-container" />
        </client-only>
        <b-field>
          <b-switch v-model="form.is_published" type="is-success">
            {{ publishLabel }}
          </b-switch>
        </b-field>
      </section>
      <footer class="modal-card-foot">
        <b-button label="Cancel" @click="$emit('close')" />
        <b-button
          :label="actionType"
          type="is-primary"
          @click="$emit('onSubmit', form)"
        />
      </footer>
    </div>
  </form>
</template>

<script>
import Tiptap from './Tiptap'

export default {
  name: 'ArticleModalForm',
  components: {
    Tiptap,
  },
  props: {
    title: String,
    content: String,
    is_published: Boolean,
    actionType: String,
  },
  computed: {
    publishLabel() {
      return this.form.is_published ? 'Publish' : 'Draft'
    },
  },
  data() {
    return {
      form: {
        title: this.title,
        content: this.content,
        is_published: this.is_published,
      },
    }
  },
}
</script>

<style scoped></style>
