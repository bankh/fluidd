<template>
  <v-card
    :class="{ 'no-pointer-events': overlay }"
    @dragover="handleDragOver"
    @dragenter.self.prevent
    @dragleave.self.prevent="handleDragLeave"
    @drop.self.prevent="handleDrop"
  >
    <job-queue-toolbar
      v-if="selected.length === 0"
      :headers="headers"
      @remove-all="handleRemoveAll"
      @refresh="handleRefresh"
    />

    <job-queue-bulk-actions
      v-else
      @remove="handleRemove(selected)"
    />

    <job-queue-browser
      v-model="selected"
      :headers="visibleHeaders"
      :dense="dense"
      :bulk-actions="bulkActions"
      @row-click="handleRowClick"
    />

    <app-drag-overlay
      v-model="overlay"
      :message="$t('app.file_system.overlay.drag_files_enqueue')"
      icon="$enqueueJob"
      absolute
    />

    <job-queue-context-menu
      v-if="contextMenuState.open"
      v-model="contextMenuState.open"
      :job="contextMenuState.job"
      :position-x="contextMenuState.x"
      :position-y="contextMenuState.y"
      @remove="handleRemove"
    />
  </v-card>
</template>

<script lang="ts">
import { SocketActions } from '@/api/socketActions'
import type { QueuedJob } from '@/store/jobQueue/types'
import { Component, Prop, Vue } from 'vue-property-decorator'
import JobQueueToolbar from './JobQueueToolbar.vue'
import JobQueueBulkActions from './JobQueueBulkActions.vue'
import JobQueueBrowser from './JobQueueBrowser.vue'
import JobQueueContextMenu from './JobQueueContextMenu.vue'
import type { AppTableHeader } from '@/types'

@Component({
  components: {
    JobQueueToolbar,
    JobQueueBulkActions,
    JobQueueBrowser,
    JobQueueContextMenu
  }
})
export default class JobQueue extends Vue {
  contextMenuState: any = {
    open: false,
    x: 0,
    y: 0,
    job: null
  }

  selected: QueuedJob[] = []
  overlay = false

  @Prop({ type: Boolean, default: false })
  readonly dense!: boolean

  @Prop({ type: Boolean, default: false })
  readonly bulkActions!: boolean

  get headers (): AppTableHeader[] {
    const headers = [
      { text: '', value: 'handle', sortable: false, width: '24px' },
      { text: this.$tc('app.general.table.header.name'), value: 'filename', sortable: false },
      { text: this.$tc('app.general.table.header.time_added'), value: 'time_added', configurable: true, sortable: false },
      { text: this.$tc('app.general.table.header.time_in_queue'), value: 'time_in_queue', configurable: true, sortable: false }
    ]
    const key = 'job_queue'
    return this.$store.getters['config/getMergedTableHeaders'](headers, key)
  }

  get visibleHeaders (): AppTableHeader[] {
    return this.headers.filter(header => header.visible || header.visible === undefined)
  }

  handleRowClick (item: QueuedJob, event: MouseEvent) {
    if (this.contextMenuState.open) {
      this.contextMenuState.open = false

      if (event.type !== 'contextmenu') {
        return
      }
    }

    if (
      this.selected.length !== 0 &&
      !this.selected.some(x => x.filename === item.filename)
    ) {
      return
    }

    // Open the context menu
    this.contextMenuState.x = event.clientX
    this.contextMenuState.y = event.clientY
    this.contextMenuState.job = this.selected.length > 1
      ? this.selected
      : item
    this.$nextTick(() => {
      this.contextMenuState.open = true
    })
  }

  async handleRemoveAll () {
    const res = await this.$confirm(
      this.$tc('app.job_queue.msg.confirm'),
      { title: this.$tc('app.general.label.confirm'), color: 'card-heading', icon: '$error' }
    )

    if (res) {
      SocketActions.serverJobQueueDeleteJobs(['all'])
    }
  }

  handleRefresh () {
    SocketActions.serverJobQueueStatus()
  }

  handleRemove (jobs: QueuedJob | QueuedJob[]) {
    const jobIds = Array.isArray(jobs)
      ? jobs.map(job => job.job_id)
      : [jobs.job_id]

    SocketActions.serverJobQueueDeleteJobs(jobIds)
  }

  handleDragOver (event: DragEvent) {
    if (event.dataTransfer?.types.includes('x-fluidd-jobs')) {
      event.preventDefault()

      event.dataTransfer.dropEffect = 'link'

      this.overlay = true
    }
  }

  handleDragLeave () {
    this.overlay = false
  }

  handleDrop (event: DragEvent) {
    this.overlay = false

    if (event.dataTransfer?.types.includes('x-fluidd-jobs')) {
      const data = event.dataTransfer.getData('x-fluidd-jobs')
      const files: { path: string, jobs: string[] } = JSON.parse(data)
      const filePath = files.path ? `${files.path}/` : ''
      const filenames = files.jobs
        .map(file => `${filePath}${file}`)

      SocketActions.serverJobQueuePostJob(filenames)
    }
  }
}
</script>
