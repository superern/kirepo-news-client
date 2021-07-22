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
