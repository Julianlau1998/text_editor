<template>
  <div class="is-editor-wrapper">
        <div class="buttonsWrapper">
            <transition name="fade" mode="out-in">
                <span :key="1">
                  <label class="label is-pointer noselect button is-primary">
                    <input @change="fileInput" accept=".txt, .md" type="file" required/>
                    <span>
                      <i class="fas fa-upload pl-1 pr-2" /> Import
                    </span>
                  </label>
                  <button
                      v-if="(!iOS || shareAvailable)"
                      :key="1"
                      class="button is-primary ml-4 pl-5 pr-4 mt-4 is-save-button"
                      @click="saveFileModal"
                  >
                    Save
                    <i class="fas fa-download pl-1 pr-2" />
                  </button>
                </span>
            </transition>
        </div>
        <br>
        <br>
        <transition name="fade" mode="out-in">
            <div :key="1">
                <textarea
                    @click="addClick"
                    v-model="inputText"
                    class="is-editor pt-5 pb-5 is-primary mt-1"
                    placeholder="Type your text here"
                />
            </div>
        </transition>

        <button
            v-if="iosLiteApp"
            @click="openAppStore"
            class="button is-ads-button is-border-secondary mt-6"
        >
          Get Rid Of Ads
        </button>

        <SaveModal
            v-if="saveFileModalOpen"
            @save="downloadFile"
            @close="saveFileModalOpen=false"
        />
  </div>
</template>

<script>
import SaveModal from '@/components/modals/SaveModal'

export default {
    name: 'Editor',
    components: {
        SaveModal
    },
    props: {
        share: {
            type: Boolean,
            required: true
        }
    },
    data () {
        return  {
            inputText: '',
            saveFileModalOpen: false,
            shareAvailable: false
        }
    },
    watch: {
        inputFile (val) {
            if (val != '') {
                this.inputText = val
                this.$store.state.inputFile = ''
            }
        },
        share (val) {
            if (val) this.shareFile()
        }
    },
    computed: {
        iOS () {
            return this.$store.state.iOS
        },
        inputFile () {
            return this.$store.state.inputFile
        },
        iosLiteApp () {
          return window.webkit && window.webkit.messageHandlers.openAppStore
        }
    },
    created () {
        this.shareAvailable = window.navigator.share
    },
    methods: {
        saveFileModal () {
            if (!this.iOS) {
                this.saveFileModalOpen = true
                return
            }
            this.shareFile()
        },
        createFileLink (fileName) {
            const file = new Blob([this.inputText], { type: "data:text/csv;charset=utf-8" }, `${fileName}.txt`)
            return window.URL.createObjectURL(file)
        },
        downloadFile (fileName) {
            const link = this.createFileLink (fileName)

            let hiddenElement = document.createElement('a')
            hiddenElement.href = link
            hiddenElement.download = `${fileName}.txt`
            hiddenElement.click();
            this.saveFileModalOpen = false
        },
        shareFile () {
            if (this.inputText.length) {
                navigator.share({
                    "title": 'txt File',
                    "text": this.inputText
                })
                .then(() => {
                    this.addClick()
                })
            }
        },
        fileInput (event) {
          const file = event.target.files[0];
          const reader = new FileReader();
          reader.onload = e => this.inputText =  e.target.result
          reader.readAsText(file);
          this.addClick()
          this.settings = false
        },
        openAppStore () {
          window.webkit.messageHandlers.openAppStore.postMessage({
            "message": 'openAppStore'
          });
        },
        addClick () {
            this.$emit('addClick')
        }
    }
}
</script>
