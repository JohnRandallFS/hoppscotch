<template>
  <SmartModal
    v-if="show"
    dialog
    :title="t('collection.open')"
    @close="hideModal"
  >
    <template #body>

    </template>
  </SmartModal>
</template>

<script lang="ts">
import { useToast } from "@composables/toast"
import { useI18n } from "@composables/i18n"
import { defineComponent } from "vue"
import { RESTCollectionImporters } from "~/helpers/import-export/import/importers"

  export default defineComponent({
    props: {
      show: Boolean,
      loadingState: Boolean,
      },
    emits: ["submit", "hide-modal"],
    setup() {
      return {
        toast: useToast(),
        t: useI18n(),
        }
    },
    data() {
      return {
        name: null
        }
    },
    watch: {
      show(isShowing: boolean) {
        if (!isShowing) {
          this.name = null
        }
      },
    },
    methods: {
      hideModal() {
        this.name = null
        this.$emit("hide-modal")
      },
    },
  })
</script>
