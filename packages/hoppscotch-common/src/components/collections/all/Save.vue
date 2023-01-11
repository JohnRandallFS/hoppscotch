<template>
  <SmartModal
    v-if="show"
    dialog
    :title="t('collection.save')"
    @close="hideModal">
    <template #body>
      <div class="flex mt-2 sm:mt-0">
        <ButtonPrimary
          :label="t('collection.save')"
          filled
          class="flex-1 rounded-r-none min-w-20"
          @click="SaveToDisk()"
          />
      </div>
      <span class="my-2 text-center">
        <div class="flex mt-2 sm:mt-0">
         <SmartCheckbox
            :on="includeEnvs"
            class="px-2"
            @change="includeEnvs = !includeEnvs">
          {{ t('environment.include_envs') }}
         </SmartCheckbox>
        </div>
        <div class="flex mt-2 sm:mt-0">
         <SmartCheckbox
            :on="includeGlobals"
            class="px-2"
            @change="includeGlobals = !includeGlobals">
           {{ t('environment.include_globals') }}
         </SmartCheckbox>
        </div>
      </span>
    </template>
  </SmartModal>
</template>


<script lang="ts">
  import { useReadonlyStream } from "@composables/stream"
  import { restCollections$ } from "~/newstore/collections"
  import {
    environments$,
    getGlobalVariables,
  } from "~/newstore/environments"
  import { ref, defineComponent } from "vue"
  import { useToast } from "@composables/toast"
  import { useI18n } from "~/composables/i18n"
  import { cloneDeep } from "lodash-es"

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
        includeEnvs: ref(true),
        includeGlobals: ref(true),
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
      SaveToDisk() {
        SaveToDisk(this.includeEnvs, this.includeGlobals)
      }
    },
  })

  const SaveToDisk = async (includeEnvs: boolean, includeGlobals: boolean) => {

    const collectionJson = ref("")
    const environmentJson = ref("")

    const collectionsStream = useReadonlyStream(restCollections$, [])
    collectionJson.value = JSON.stringify(collectionsStream.value, null, 2)
    const collections = "collections: " + collectionJson.value

    const environmentsStream = useReadonlyStream(environments$, [])
    environmentJson.value = JSON.stringify(environmentsStream.value, null, 2)
    const environments = includeEnvs
      ? ", environments: " + environmentJson.value
      : ""

    const globals = includeGlobals
      ? ", globals: " + cloneDeep(getGlobalVariables())
      : ""
 
    const file = new Blob(["{" + collections + environments + globals + "}"], { type: "application/json" })
    const a = document.createElement("a")
    const url = URL.createObjectURL(file)
    a.href = url

    // TODO: get uri from meta
    a.download = `Hoppscotch_Collections.json`
    document.body.appendChild(a)
    a.click()
    //this.toast.success(this.t("state.download_started").toString())
    setTimeout(() => {
      document.body.removeChild(a)
      URL.revokeObjectURL(url)
    }, 1000)
  }
</script>
