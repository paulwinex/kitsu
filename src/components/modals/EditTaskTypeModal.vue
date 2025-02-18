<template>
<div :class="{
  'modal': true,
  'is-active': active
}">
  <div class="modal-background" @click="$emit('cancel')" ></div>

  <div class="modal-content">

    <div class="box">

      <h1 class="title" v-if="isEditing">
        {{ $t("task_types.edit_title") }} {{ taskTypeToEdit.name }}
      </h1>
      <h1 class="title" v-else>
        {{ $t("task_types.new_task_type") }}
      </h1>

      <form v-on:submit.prevent>
        <text-field
          ref="nameField"
          :label="$t('task_types.fields.name')"
          v-model="form.name"
          @enter="confirmClicked"
          v-focus
        />
        <combobox
          :label="$t('task_types.fields.dedicated_to')"
          :options="dedicatedToOptions"
          @enter="confirmClicked"
          v-model="form.for_entity"
           v-if="!isEditing"
        />
        <combobox-boolean
          :label="$t('task_types.fields.allow_timelog')"
          @enter="confirmClicked"
          v-model="form.allow_timelog"
        />
        <combobox-department
          :label="$t('task_types.fields.department')"
          @enter="confirmClicked"
          v-model="form.department_id"
        />
        <color-field
          ref="colorField"
          :label="$t('task_types.fields.color')"
          v-model="form.color"
        />
      </form>

      <modal-footer
        :error-text="$t('task_types.create_error')"
        :is-loading="isLoading"
        :is-error="isError"
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

import Combobox from '@/components/widgets/Combobox.vue'
import ComboboxBoolean from '@/components/widgets/ComboboxBoolean.vue'
import ComboboxDepartment from '@/components/widgets/ComboboxDepartment.vue'
import ColorField from '@/components/widgets/ColorField'
import ModalFooter from '@/components/modals/ModalFooter'
import TextField from '@/components/widgets/TextField'

export default {
  name: 'edit-task-type-modal',
  mixins: [modalMixin],
  components: {
    Combobox,
    ComboboxBoolean,
    ComboboxDepartment,
    ColorField,
    ModalFooter,
    TextField
  },

  props: [
    'active',
    'onConfirmClicked',
    'entries',
    'isLoading',
    'isError',
    'taskTypeToEdit',
    'text'
  ],

  watch: {
    taskTypeToEdit () {
      if (this.taskTypeToEdit) {
        this.form = {
          name: this.taskTypeToEdit.name,
          color: this.taskTypeToEdit.color,
          for_entity: this.taskTypeToEdit.for_entity,
          allow_timelog: String(this.taskTypeToEdit.allow_timelog === true),
          department_id: this.taskTypeToEdit.department_id
        }
      }
    }
  },

  data () {
    return {
      form: {
        name: '',
        color: '$grey',
        for_entity: 'Asset',
        allow_timelog: 'false',
        department_id: null
      },
      dedicatedToOptions: [
        { label: this.$t('assets.title'), value: 'Asset' },
        { label: this.$t('shots.title'), value: 'Shot' },
        { label: this.$t('edits.title'), value: 'Edit' }
      ]
    }
  },

  computed: {
    ...mapGetters([
      'taskTypes',
      'taskTypeStatusOptions',
      'departments'
    ]),
    isEditing () {
      return this.taskTypeToEdit && this.taskTypeToEdit.id
    }
  },

  methods: {
    ...mapActions([
    ]),

    newPriority (forEntity) {
      return this.entries.filter(taskType => taskType.for_entity === forEntity).length + 1
    },

    confirmClicked () {
      if (!this.isEditing) {
        this.form.priority = this.newPriority(this.form.for_entity)
      }
      this.$emit('confirm', this.form)
    }
  }
}
</script>

<style lang="scss" scoped>
.modal-content .box p.text {
  margin-bottom: 1em;
}
.is-danger {
  color: #ff3860;
  font-style: italic;
}
</style>
