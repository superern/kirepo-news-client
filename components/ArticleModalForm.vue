<template>
  <form action="">
    <div class="modal-card" style="width: auto">
      <header class="modal-card-head">
        <p class="modal-card-title">Article</p>
        <button type="button" class="delete" @click="$emit('close')" />
      </header>
      <section class="modal-card-body">
        <b-field label="Name">
          <b-input type="text" v-model="form.title" placeholder="Name" required>
          </b-input>
        </b-field>

        <b-field label="Description">
          <b-input
            maxlength="200"
            type="textarea"
            v-model="form.content"
            password-reveal
            placeholder="Your description"
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
  name: 'TagModalForm',
  props: {
    title: String,
    description: String,
    is_published: Boolean,
    actionType: String,
  },
  data() {
    return {
      form: {
        title: '',
        description: '',
        is_published: false,
      },
    }
  },
}
</script>

<style scoped></style>
