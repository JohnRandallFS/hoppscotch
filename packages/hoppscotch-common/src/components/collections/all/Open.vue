<template>
  <SmartModal
    v-if="show"
    dialog
    :title="t('collection.open')"
    @close="hideModal"
  >
    <template #body>
      <div class="flex mt-2 sm:mt-0">
        <ButtonPrimary :icon="IconDownload"
                       :label="t('collection.save')"
                       :disabled="enableImportButton"
                       filled
                       class="flex-1 rounded min-w-20"
                       @click="openFromDisk" />
      </div>
      <span class="my-2 text-center">
        <div class="flex mt-2 sm:mt-0">
          <SmartCheckbox :on="includeEnvs"
                         class="px-2"
                         @change="includeEnvs = !includeEnvs">
            {{ t('environment.include_envs') }}
          </SmartCheckbox>
        </div>
        <div class="flex mt-2 sm:mt-0">
          <SmartCheckbox :on="includeGlobals"
                         class="px-2"
                         @change="includeGlobals = !includeGlobals">
            {{ t('environment.include_globals') }}
          </SmartCheckbox>
        </div>
      </span>
      <div class="space-y-4">
        <p class="flex items-center">
          <span class="inline-flex items-center justify-center flex-shrink-0 mr-4 border-4 rounded-full border-primary text-dividerDark"
                :class="{
                    '!text-green-500': hasFile,
                  }">
            <icon-lucide-check-circle class="svg-icons" />
          </span>
          <span>
            {{ t('environment.include_globals') }}
          </span>
        </p>
        <p class="flex flex-col ml-10 border border-dashed rounded border-dividerDark">
          <input id="inputChooseFileToOpenAll"
                 ref="inputChooseFileToOpenAll"
                 name="inputChooseFileToOpenAll"
                 type="file"
                 class="p-4 cursor-pointer transition file:transition file:cursor-pointer text-secondary hover:text-secondaryDark file:mr-2 file:py-2 file:px-4 file:rounded file:border-0 file:text-secondary hover:file:text-secondaryDark file:bg-primaryLight hover:file:bg-primaryDark"
                 :accept="acceptedFileTypes"
                 :change="onFileChange" />
        </p>
        </div>
    </template>
  </SmartModal>
</template>

<script setup lang="ts">
import { useToast } from "@composables/toast"
import { useI18n } from "@composables/i18n"
import {
  computed,
  ref,
} from "vue"
import { translateToNewRESTCollection } from "@hoppscotch/data"
import {
  restCollectionStore,
  appendRESTCollections,
  removeRESTCollection,
} from "~/newstore/collections"
import {
  environmentsStore,
  appendEnvironments,
  deleteEnvironment,
  addGlobalEnvVariable,
  setGlobalEnvVariables,
} from "~/newstore/environments"
import { size } from "lodash-es"

  const props = defineProps<{
    show: boolean
    loadingState: boolean,
  }>()

  const emit = defineEmits<{
    (e: "hide-modal"): void
  }>()

  const toast = useToast()
  const t = useI18n()
  const includeEnvs = ref(false)
  const includeGlobals = ref(false)
  const overwrite = ref(true) //Change after testing
  const inputChooseFileToOpenAll = ref<HTMLInputElement | any>()
  const acceptedFileTypes = "application/json"
  var collectionsFile: Object

  const fileImported = () => {
    toast.success(t("state.file_imported").toString())
    hideModal()
  }

  const failedImport = () => {
    toast.error(t("import.failed").toString())
  }

  const hideModal = () => {
    emit("hide-modal")
  }

  const hasFile = ref(false)

  const onFileChange = () => {
    if (!inputChooseFileToOpenAll.value[0]) {
      hasFile.value = false
      return
    }

    if (
      !inputChooseFileToOpenAll.value[0].files ||
      inputChooseFileToOpenAll.value[0].files.length === 0
    ) {
      inputChooseFileToOpenAll.value[0].value = ""
      hasFile.value = false
      toast.show(t("action.choose_file").toString())
      return
    }

    //Something here is failing, file format not recognised,
    //Directly importing file does work, so the issue is with handling here
    const reader = new FileReader()
    reader.onload = ({ target }) => {
      const content = target!.result as string | null
      if (!content) {
        hasFile.value = false
        toast.show(t("action.choose_file").toString())
        return
      }

      collectionsFile = JSON.parse(content)
      hasFile.value = !!content?.length
    }
    reader.readAsText(inputChooseFileToOpenAll.value[0].files[0])
  }

  const openFromDisk = () => {
    try {
      toast.show(String(size(collectionsFile))) //Test
      if (collectionsFile.collections) { 
        const collections = collectionsFile.collections

        if (overwrite.value) {
          const length = restCollectionStore.value.state.length

          for (let i = 0; i < length; i++) {
            removeRESTCollection(i)
          }
        }

        var hoppCollections = []

        for (let i = 0; i < size(collections); i++) {
          const hoppCollection = translateToNewRESTCollection(collections[i])
          hoppCollections.push(hoppCollection)
        }

        appendRESTCollections(hoppCollections)
      }

      if (includeEnvs.value && collectionsFile.environments) {
        const environments = collectionsFile.environments

        if (overwrite.value) {
          const length = environmentsStore.value.environments.length

          for (let i = 0; i < length; i++) {
            deleteEnvironment(i)
          }
        }

        appendEnvironments(environments)
      }

      if (includeGlobals.value && collectionsFile.globals) {
        const globals = collectionsFile.globals

        if (overwrite.value) {
          setGlobalEnvVariables(globals)
        } else {
          for (let i = 0; i < size(globals); i++) {
            addGlobalEnvVariable(globals[i])
          }
        }
        
      }

      fileImported()
    } catch (e) {
      failedImport()
    }
  }

  const enableImportButton = computed(
    () => !(hasFile)
  )
</script>
