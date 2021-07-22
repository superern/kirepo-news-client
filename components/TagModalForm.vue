<template>
  <form action="">
    <div class="modal-card" style="width: auto">
      <header class="modal-card-head">
        <p class="modal-card-title">Tag</p>
        <button type="button" class="delete" @click="$emit('close')" />
      </header>
      <section class="modal-card-body">
        <b-field label="Name">
          <b-input type="text" v-model="form.name" placeholder="Name" required>
          </b-input>
        </b-field>

        <b-field label="Description">
          <b-input
            maxlength="200"
            type="textarea"
            v-model="form.description"
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
    name: String,
    description: String,
    actionType: String,
  },
  data() {
    return {
      form: {
        name: '',
        description: '',
      },
    }
  },
}
</script>

<style scoped></style>
