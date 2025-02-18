<template>
<div class="data-list">
  <div class="datatable-wrapper">
    <table class="datatable">
      <thead class="datatable-head">
        <tr>
          <th scope="col" class="name">
            {{ $t('task_status.fields.name') }}
          </th>
          <th scope="col" class="short-name">
            {{ $t('task_status.fields.short_name') }}
          </th>
          <th scope="col" class="is-done">
            {{ $t('task_status.fields.is_done') }}
          </th>
          <th scope="col" class="is-default">
            {{ $t('task_status.fields.is_default') }}
          </th>
          <th scope="col" class="is-retake">
            {{ $t('task_status.fields.is_retake') }}
          </th>
          <th scope="col" class="is-artist-allowed">
            {{ $t('task_status.fields.is_artist_allowed') }}
          </th>
          <th scope="col" class="is-client-allowed">
            {{ $t('task_status.fields.is_client_allowed') }}
          </th>
          <th scope="col" class="is-feedback-request">
            {{ $t('task_status.fields.is_feedback_request') }}
          </th>
          <th scope="col" class="actions"></th>
        </tr>
      </thead>
      <tbody class="datatable-body">
        <tr class="datatable-row" v-for="entry in entries" :key="entry.id">
          <td class="name">
            {{ entry.name }}
          </td>
          <task-status-name class="short-name" :entry="entry" />
          <td class="is-done">
            {{ formatBoolean(entry.is_done) }}
          </td>
          <td class="is-default">
            {{ formatBoolean(entry.is_default) }}
          </td>
          <td class="is-retake">
            {{ formatBoolean(entry.is_retake) }}
          </td>
          <td class="is-artist-allowed">
            {{ formatBoolean(entry.is_artist_allowed) }}
          </td>
          <td class="is-client-allowed">
            {{ formatBoolean(entry.is_client_allowed) }}
          </td>
          <td class="is-feedback-request">
            {{ formatBoolean(entry.is_feedback_request) }}
          </td>
          <row-actions-cell
            :entry-id="entry.id"
            :hide-delete="entry.is_default === true"
            @edit-clicked="$emit('edit-clicked', entry)"
            @delete-clicked="$emit('delete-clicked', entry)"
          />
        </tr>
      </tbody>
    </table>
  </div>

  <table-info
    :is-loading="isLoading"
    :is-error="isError"
  />

  <p class="has-text-centered nb-task-status">
    {{ entries.length }} {{ $tc('task_status.number', entries.length) }}
  </p>

</div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { formatListMixin } from '@/components/mixins/format'
import RowActionsCell from '@/components/cells/RowActionsCell'
import TableInfo from '@/components/widgets/TableInfo'
import TaskStatusName from '@/components/cells/TaskStatusName'

export default {
  name: 'task-status-list',
  mixins: [formatListMixin],
  props: [
    'entries',
    'isLoading',
    'isError'
  ],
  data () {
    return {}
  },
  components: {
    RowActionsCell,
    TableInfo,
    TaskStatusName
  },
  computed: {
    ...mapGetters([
    ])
  },
  methods: {
    ...mapActions([
    ])
  }
}
</script>

<style lang="scss" scoped>

.datatable-body tr:first-child th,
.datatable-body tr:first-child td {
  border-top: 0;
}

.name {
  width: 200px;
  min-width: 200px;
}

.short-name {
  width: 150px;
  min-width: 150px;
}

.is-reviewable,
.is-done,
.is-default,
.is-retake,
.is-artist-allowed,
.is-client-allowed {
  width: 140px;
  min-width: 140px;
}
</style>
