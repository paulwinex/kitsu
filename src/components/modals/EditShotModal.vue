<template>
<div :class="{
  'modal': true,
  'is-active': active
}">
  <div class="modal-background" @click="$emit('cancel')" ></div>

  <div class="modal-content">
    <div class="box">

      <h1 class="title" v-if="shotToEdit && this.shotToEdit.id">
        {{ $t("shots.edit_title") }} {{ shotToEdit.name }}
      </h1>
      <h1 class="title" v-else>
        {{ $t("shots.new_shot") }}
      </h1>

      <form v-on:submit.prevent>
        <combobox
          :label="$t('shots.fields.sequence')"
          :options="getSequenceOptions"
          v-model="form.sequence_id"
        />
        <text-field
          ref="nameField"
          :label="$t('shots.fields.name')"
          v-model="form.name"
          @enter="runConfirmation"
          v-focus
        />
        <textarea-field
          ref="descriptionField"
          :label="$t('shots.fields.description')"
          v-model="form.description"
          @keyup.ctrl.enter="runConfirmation"
          @keyup.meta.enter="runConfirmation"
        />
        <text-field
          ref="nbFramesField"
          :label="$t('shots.fields.nb_frames')"
          type="number"
          v-model="form.nb_frames"
          @enter="runConfirmation"
          v-focus
        />
        <text-field
          ref="frameInField"
          :label="$t('shots.fields.frame_in')"
          v-model="form.frameIn"
          type="number"
        />
        <text-field
          ref="frameOutField"
          :label="$t('shots.fields.frame_out')"
          v-model="form.frameOut"
          type="number"
          @enter="runConfirmation"
        />
        <text-field
          ref="fpsField"
          :label="$t('shots.fields.fps')"
          v-model="form.fps"
          type="number"
          @enter="runConfirmation"
        />
        <div
          :key="descriptor.id"
          v-for="descriptor in shotMetadataDescriptors"
        >
          <combobox
            v-if="descriptor.choices.length > 0"
            :label="descriptor.name"
            :options="getDescriptorChoicesOptions(descriptor)"
            v-model="form.data[descriptor.field_name]"
          />
          <text-field
            :label="descriptor.name"
            v-model="form.data[descriptor.field_name]"
            @enter="runConfirmation"
            v-else
          />
        </div>
      </form>

      <modal-footer
        :error-text="$t('shots.edit_fail')"
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
import Combobox from '@/components/widgets/Combobox'
import ModalFooter from '@/components/modals/ModalFooter'
import TextField from '@/components/widgets/TextField'
import TextareaField from '@/components/widgets/TextareaField'

export default {
  name: 'edit-shot-modal',
  mixins: [modalMixin],

  components: {
    Combobox,
    ModalFooter,
    TextField,
    TextareaField
  },

  props: [
    'onConfirmClicked',
    'text',
    'active',
    'cancelRoute',
    'isError',
    'isLoading',
    'isLoadingStay',
    'isSuccess',
    'shotToEdit',
    'errorText'
  ],

  data () {
    return {
      form: {
        data: {}
      },
      shotSuccessText: ''
    }
  },

  mounted () {
    this.resetForm()
  },

  computed: {
    ...mapGetters([
      'currentProduction',
      'getSequenceOptions',
      'getOpenProductionOptions',
      'openProductions',
      'sequences',
      'shots',
      'shotCreated',
      'shotMetadataDescriptors'
    ]),

    frameIn () {
      return this.shotToEdit.data ? this.shotToEdit.data.frame_in : ''
    },

    frameOut () {
      return this.shotToEdit.data ? this.shotToEdit.data.frame_out : ''
    },

    fps () {
      return this.shotToEdit.data ? this.shotToEdit.data.fps : ''
    }
  },

  methods: {
    ...mapActions([
      'loadSequences'
    ]),

    runConfirmation () {
      if (this.isEditing()) {
        this.confirmClicked()
      } else {
        this.confirmAndStayClicked()
      }
    },

    confirmAndStayClicked () {
      this.$emit('confirmAndStay', this.form)
    },

    confirmClicked () {
      this.$emit('confirm', this.form)
    },

    getDescriptorChoicesOptions (descriptor) {
      const values = descriptor.choices.map(c => ({ label: c, value: c }))
      return [{ label: '', value: '' }, ...values]
    },

    isEditing () {
      return this.shotToEdit && this.shotToEdit.id
    },

    resetForm () {
      this.shotSuccessText = ''
      if (!this.isEditing()) {
        if (this.openProductions.length > 0) {
          this.form.project_id =
            this.currentProduction ? this.currentProduction.id : ''
        }
        if (this.sequences.length > 0) {
          this.form.sequence_id = this.sequences[0].id
        }
        this.form.name = ''
        this.form.description = ''
        this.form.nb_frames = 0
        this.form.data = {}
      } else {
        this.form = {
          sequence_id: this.shotToEdit.sequence_id,
          project_id: this.shotToEdit.project_id,
          name: this.shotToEdit.name,
          description: this.shotToEdit.description,
          nb_frames: this.shotToEdit.nb_frames,
          frameIn: this.frameIn,
          frameOut: this.frameOut,
          fps: this.fps,
          data: { ...this.shotToEdit.data } || {}
        }
      }
    }
  },

  watch: {
    active () {
      this.shotSuccessText = ''
      this.resetForm()
      if (this.sequences.length === 0) {
        this.loadSequences()
      }
      if (this.active) {
        setTimeout(() => {
          this.$refs.nameField.focus()
        }, 100)
      }
    },

    shotToEdit () {
      this.resetForm()
    },

    shotCreated () {
      if (this.isEditing()) {
        this.shotSuccessText = this.$t('shots.edit_success', {
          name: this.shotCreated
        })
      } else {
        this.shotSuccessText = this.$t('shots.new_success', {
          name: this.shotCreated
        })
      }
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
.title {
  border-bottom: 2px solid #DDD;
  padding-bottom: 0.5em;
  margin-bottom: 1.2em;
}
.info-message {
  margin-top: 1em;
}

.modal-content {
  max-height: 80%;
}
</style>
