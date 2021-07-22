<template>
  <form action="">
    <div class="modal-card" style="width: auto">
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

        <b-field label="Content">
          <b-input
            maxlength="200"
            type="textarea"
            v-model="form.content"
            password-reveal
            placeholder="Your Content"
          >
          </b-input>
        </b-field>
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
export default {
  name: 'ArticleModalForm',
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
