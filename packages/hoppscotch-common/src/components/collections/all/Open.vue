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
                       filled
                       class="flex-1 rounded min-w-20"
                       @click="openFromDisk()" />
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
                 :change="getFile()" />
        </p>
        </div>
    </template>
  </SmartModal>
</template>

<script lang="ts">
import { useToast } from "@composables/toast"
import { useI18n } from "@composables/i18n"
  import {
    defineComponent,
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
  clearGlobalEnvVariables,
  setGlobalEnvVariables,
  } from "~/newstore/environments"

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
        includeEnvs: ref(false),
        includeGlobals: ref(false),
        overwrite: ref(false), //Change after testing
        acceptedFileTypes: "application/json",
        }
    },
    data() {
      return {
        name: null,
        file: {} as Blob,
        content: "" as string,
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
      openFromDisk() {

        const reader = new FileReader()

        reader.onload = (res) => {
          this.content = res.target!.result as string

          if (!this.content) {
            this.toast.show(this.t("action.choose_file").toString())
            return
          }

          const collectionsFile = JSON.parse(this.content)

          if (collectionsFile.collections) {
            const collections = collectionsFile.collections

            if (this.overwrite) {
              const length = restCollectionStore.value.state.length

              for (let i = 0; i < length; i++) {
                removeRESTCollection(i)
              }
            }

            var hoppCollections = []

            for (const collection in collections) {
              const hoppCollection = translateToNewRESTCollection(collection)
              hoppCollections.push(hoppCollection)
            }

            appendRESTCollections(hoppCollections)
          }

          if (this.includeEnvs && collectionsFile.environments) {
            const environments = collectionsFile.environments

            if (this.overwrite) {
              const length = environmentsStore.value.environments.length

              for (let i = 0; i < length; i++) {
                deleteEnvironment(i)
              }
            }

            appendEnvironments(environments)
          }

          if (this.includeGlobals && collectionsFile.globals) {
            const globals = collectionsFile.globals

            if (this.overwrite) {
              clearGlobalEnvVariables()
            }

            setGlobalEnvVariables(globals)
          }

        }

        reader.readAsText(this.file)

      },
      getFile(openElement: any) {
        //if (!openElement.target) return

        //if (
        //  !openElement.target.file[0].value ||
        //  openElement.target.file[0].value.length === 0
        //) {
        //  this.toast.show(this.t("action.choose_file").toString())
        //  return
        //}
        //this.file = openElement!.target.files[0]
      }
    },
  })


  /*const inputChooseFileToOpenAll = ref<HTMLInputElement | any>()

  const openFromDisk = (includeEnvs: boolean, includeGlobals: boolean, overwrite: boolean) => {
    if (!inputChooseFileToOpenAll.value) return

    if (
      !inputChooseFileToOpenAll.value.files ||
      inputChooseFileToOpenAll.value.files.length === 0
    ) {
      this.toast.show(this.t("action.choose_file").toString())
      return
    }

    const reader = new FileReader()

    reader.onload = ({ target }) => {
      const content = target!.result as string | null

      if (!content) {
        this.toast.show(this.t("action.choose_file").toString())
        return
      }

      const collectionsFile = JSON.parse(content)

      if (collectionsFile.collections) {
        const collections = collectionsFile.collections

        if (overwrite) {
          const length = restCollectionStore.value.state.length

          for (let i = 0; i < length; i++) {
            removeRESTCollection(i)
          }
        }

        var hoppCollections = []

        for (const collection in collections) {
          const hoppCollection = translateToNewRESTCollection(collection)
          hoppCollections.push(hoppCollection)
        }
        
        appendRESTCollections(hoppCollections)
      }

      if (includeEnvs && collectionsFile.environments) {
        const environments = collectionsFile.environments

        if (overwrite) {
          const length = environmentsStore.value.environments.length

          for (let i = 0; i < length; i++) {
            deleteEnvironment(i)
          }
        }

        appendEnvironments(environments)
      }

      if (includeGlobals && collectionsFile.globals) {
        const globals = collectionsFile.globals

        if (overwrite) {
          clearGlobalEnvVariables()
        }

        setGlobalEnvVariables(globals)
      }

    }

    reader.readAsText(inputChooseFileToOpenAll.value[0].files[0])
    //inputChooseFileToOpenAll.value.value = ""
    
  }*/
</script>
