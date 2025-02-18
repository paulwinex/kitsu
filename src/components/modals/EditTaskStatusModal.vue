<template>
<div :class="{
  'modal': true,
  'is-active': active
}">
  <div class="modal-background" @click="$emit('cancel')" ></div>

  <div class="modal-content">
    <div class="box">

      <h1 class="title" v-if="isEditing()">
        {{ $t("task_status.edit_title") }} {{ taskStatusToEdit.name }}
      </h1>
      <h1 class="title" v-else>
        {{ $t("task_status.new_task_status") }}
      </h1>

      <form v-on:submit.prevent>
        <text-field
          ref="nameField"
          input-class="task-status-name"
          :label="$t('task_status.fields.name')"
          @enter="confirmClicked"
          v-model="form.name"
          v-focus
        />
        <text-field
          ref="shortNameField"
          input-class="task-status-short-name"
          :label="$t('task_status.fields.short_name')"
          :maxlength="8"
          @enter="confirmClicked"
          v-model="form.short_name"
          v-focus
        />
        <combobox-boolean
          :label="$t('task_status.fields.is_done')"
          @enter="confirmClicked"
          v-model="form.is_done"
          v-if="form.is_default === 'false'"
        />
        <combobox-boolean
          :label="$t('task_status.fields.is_retake')"
          @enter="confirmClicked"
          v-model="form.is_retake"
          v-if="form.is_default === 'false'"
        />
        <combobox-boolean
          :label="$t('task_status.fields.is_artist_allowed')"
          @enter="confirmClicked"
          v-model="form.is_artist_allowed"
        />
        <combobox-boolean
          :label="$t('task_status.fields.is_client_allowed')"
          @enter="confirmClicked"
          v-model="form.is_client_allowed"
        />
        <combobox-boolean
          :label="$t('task_status.fields.is_feedback_request')"
          @enter="confirmClicked"
          v-model="form.is_feedback_request"
          v-if="form.is_default === 'false'"
        />

        <color-field
          ref="colorField"
          :label="$t('task_status.fields.color')"
          :colors="colors"
          v-model="form.color"
        />
      </form>

      <modal-footer
        :error-text="$t('task_status.create_error')"
        :is-error="isError"
        :is-loading="isLoading"
        @confirm="confirmClicked"
        @cancel="$emit('cancel')"
      />
    </div>

  </div>
</div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { modalMixin } from '@/components/modals/base_modal'
import ComboboxBoolean from '@/components/widgets/ComboboxBoolean'
import ColorField from '@/components/widgets/ColorField'
import ModalFooter from '@/components/modals/ModalFooter'
import TextField from '@/components/widgets/TextField'

export default {
  name: 'edit-task-status-modal',
  mixins: [modalMixin],
  components: {
    ColorField,
    ComboboxBoolean,
    ModalFooter,
    TextField
  },

  props: [
    'onConfirmClicked',
    'text',
    'active',
    'cancelRoute',
    'isLoading',
    'isError',
    'taskStatusToEdit'
  ],

  data () {
    return {
      form: {
        name: '',
        short_name: '',
        color: '$grey999',
        is_done: 'false',
        is_feedback_request: 'false',
        is_default: 'false'
      },
      isRetakeOptions: [
        { label: this.$t('main.yes'), value: 'true' },
        { label: this.$t('main.no'), value: 'false' }
      ],
      isDoneOptions: [
        { label: this.$t('main.yes'), value: 'true' },
        { label: this.$t('main.no'), value: 'false' }
      ],
      isArtistAllowedOptions: [
        { label: this.$t('main.yes'), value: 'true' },
        { label: this.$t('main.no'), value: 'false' }
      ],
      colors: [
        '#000000',
        '#FF3860',
        '#E81123',
        '#E74C3C',
        '#FF5722',
        '#FF7043',
        '#FFA000',
        '#FBC02D',
        '#AFB42B',
        '#8BC34A',
        '#66BB6A',
        '#22D160',
        '#4DB6AC',
        '#03A9F4',
        '#3273DC',
        '#3498DB',
        '#2980B9',
        '#607D8B',
        '#8764B8',
        '#AB26FF',
        '#E040FB',
        '#FF80AB'
      ]
    }
  },

  computed: {
    ...mapGetters([
      'taskStatus',
      'taskStatusStatusOptions'
    ])
  },

  methods: {
    ...mapActions([
    ]),

    confirmClicked () {
      this.$emit('confirm', this.form)
    },

    isEditing () {
      return this.taskStatusToEdit && this.taskStatusToEdit.id
    },

    resetForm () {
      if (this.taskStatusToEdit) {
        this.form = {
          name: this.taskStatusToEdit.name,
          short_name: this.taskStatusToEdit.short_name,
          color: this.taskStatusToEdit.color,
          is_done: String(this.taskStatusToEdit.is_done),
          is_retake: String(this.taskStatusToEdit.is_retake || false),
          is_artist_allowed: String(this.taskStatusToEdit.is_artist_allowed),
          is_client_allowed: String(this.taskStatusToEdit.is_client_allowed),
          is_default: String(this.taskStatusToEdit.is_default || false),
          is_feedback_request:
            String(this.taskStatusToEdit.is_feedback_request || false)
        }
      }
    }
  },

  watch: {
    taskStatusToEdit () {
      this.resetForm()
    },

    active () {
      if (this.active) {
        this.resetForm()
        setTimeout(() => {
          this.$refs.nameField.focus()
        }, 100)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.modal-content .box p.text {
  margin-bottom: 1em;
}
</style>
