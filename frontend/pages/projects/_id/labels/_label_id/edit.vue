<template>
  <form-create
    v-slot="slotProps"
    v-bind.sync="editedItem"
    :items="items"
  >
    <v-btn
      :disabled="!slotProps.valid"
      color="primary"
      class="text-capitalize"
      @click="save"
    >
      Save
    </v-btn>
  </form-create>
</template>

<script lang="ts">
import Vue from 'vue'
import { LabelDTO } from '~/services/application/label/labelData'
import { ProjectDTO } from '~/services/application/project/projectData'
import FormCreate from '~/components/label/FormCreate.vue'

export default Vue.extend({
  components: {
    FormCreate,
  },

  layout: 'project',

  validate({ params, query, app }) {
    if (!['category', 'span', 'relation'].includes((query.type as string))) {
      return false
    }
    if (/^\d+$/.test(params.id)) {
      return app.$services.project.findById(params.id)
      .then((res:ProjectDTO) => {
        return res.canDefineLabel
      })
    }
    return false
  },

  data() {
    return {
      editedItem: {
        text: '',
        prefixKey: null,
        suffixKey: null,
        backgroundColor: '#73D8FF',
        textColor: '#ffffff'
      } as LabelDTO,
      items: [] as LabelDTO[]
    }
  },

  computed: {
    projectId(): string {
      return this.$route.params.id
    },

    labelId(): string {
      return this.$route.params.label_id
    },

    service(): any {
      const type = this.$route.query.type
      if (type === 'category') {
        return this.$services.categoryType
      } else if (type === 'span'){
        return this.$services.spanType
      } else {
        return this.$services.relationType
      }
    },
  },

  async created() {
    this.items = await this.service.list(this.projectId)
    this.editedItem = await this.service.findById(this.projectId, this.labelId)
  },

  methods: {
    async save() {
      await this.service.update(this.projectId, this.editedItem)
      this.$router.push(`/projects/${this.projectId}/labels`)
    }
  }
})
</script>
