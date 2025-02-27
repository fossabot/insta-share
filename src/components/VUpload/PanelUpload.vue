<template>
  <section id="panel-upload">
    <div class="content panel-upload--content">
      <div class="panel-upload--dropzone" :class="{ active: isDragged }" @dragenter="onDragEnter" @dragleave="onDragLeave" @drop.prevent="onDropHandler" @dragover.prevent>
        <input type="file" multiple ref="fileRef" @change="onFileChangedHandler" />

        <div class="dropzone-label" @click="openSelectFile">
          <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#FFFFFF"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M9 16h6v-6h4l-7-7-7 7h4v6zm3-10.17L14.17 8H13v6h-2V8H9.83L12 5.83zM5 18h14v2H5z"/></svg>
          <span>Drop files here or click to select files.</span>

          <div class="dropzone-is-loading" :class="{ active: (fileCount > 0) }">
            <div class="dropzone-loading--bar"></div>
          </div>
          <span v-show="(fileCount > 0)">{{ (fileCount - finished) }} of {{ fileCount }} files being transfered.</span>
        </div>

        <div class="dropzone-details">
          <div class="dropzone-detail">{{ result.count }} files</div>
          <div class="dropzone-detail">{{ fileSize(result.size) }}</div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { computed, inject, reactive, ref } from "vue";
import { useStore } from "@src/store";
import { uploadBlob } from "@src/services/nft.js"
import { fileSize } from "@src/services/helpers";

export default {
  name: "PanelUpload",
  setup() {
    const notyf = inject("notyf");
    const fileRef = ref(null);
    const isDragged = ref(false);
    const finished = ref(0);
    const isUploading = ref(false);

    const store = useStore();

    const onDropHandler = ($event) => {
      if (isUploading.value) return false;

      isDragged.value = false;

      fileRef.value.files = $event.dataTransfer.files;

      onFileChangedHandler();
    }
    const openSelectFile = () => {
      if (isUploading.value) return false;

      fileRef.value.click();
    }
    const onDragEnter = () => {
      isDragged.value = true;
    }
    const onDragLeave = () => {
      isDragged.value = false;
    }

    const uploadFileHandler = async (file) => {
      const result = await uploadBlob(file);
      
      finished.value++;
      
      const [ isError, _ ] = result;
      
      if (isError) {
        notyf.error(isError.toString());
      }

      return result;
    }
    const onFileChangedHandler = async () => {
      isUploading.value = true;

      store.addFiles(...fileRef.value.files);

      const files = store.files.map(file => uploadFileHandler(file));

      try {
        let results = await Promise.all(files);
        const successfully = results.filter(([error, file]) => !error);

        store.addResults(...successfully.map(([ error, file ]) => file));
        store.resetFiles();

        fileRef.value.value = null;

        if (successfully.length > 0) {
          notyf.success(`${successfully.length} files successfully processed.`);
        }
      } catch (error) {
        notyf.error(`Opss!, something error while processing your files.`);
      } finally {
        finished.value = 0;
        isUploading.value = false;
      }
    }

    const fileCount = computed(() => {
      return store.files.length;
    });
    const result = computed(() => {
      return {
        count: store.results.length,
        size: store.results.reduce((sum, result) => {
          return sum + result.file.size;
        }, 0)
      }
    });

    return {
      finished,
      fileRef,
      fileCount,
      result,
      isDragged,
      fileSize,
      onDragEnter,
      onDragLeave,
      onDropHandler,
      openSelectFile,
      onFileChangedHandler,
    }
  },
}
</script>

<style lang="scss">
section#panel-upload {
  background-color: rgba(255, 255, 255, .1);
  border-top-left-radius: 1rem;
  border-bottom-left-radius: 1rem;

  .panel-upload--content,
  .panel-upload--content .panel-upload--dropzone {
    width: 100%;
    height: 100%;
  }

  .panel-upload--dropzone {
    position: relative;
    cursor: pointer;
    overflow: hidden;

    display: flex;
    align-content: center;
    align-items: center;
    justify-content: center;

    &.active {
      > * {
        pointer-events: none;
      }

      .dropzone-label {
        background-color: rgba(0, 0, 0, .2);
      }
    }

    input {
      display: none;
    }
    .dropzone-label {
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: .8rem;
      border-radius: .5rem;
      text-align: center;
      width: 80%;

      svg {
        height: 48px;
        width: 48px;
        margin-bottom: 1rem;
      }

      span {
        font-size: .8rem;
      }
    }
    .dropzone-details {
      position: absolute;
      display: flex;

      bottom: 1rem;
      left: 1rem;

      .dropzone-detail {
        background-color: rgba(255, 255, 255, .1);
        border-radius: 1rem;
        padding: .4rem .8rem;
        font-size: .8rem;
        margin-right: .6rem;
      }
    }

    .dropzone-is-loading {
      opacity: 0;

      position: relative;
      height: 4px;
      display: block;
      width: 150px;
      background-color: rgba(0, 0, 0, .25);
      border-radius: 2px;
      margin: 1rem 0 1rem 0;
      overflow: hidden;

      &.active {
        opacity: 1;
      }

      .dropzone-loading--bar {
        background-color: var(--primary-color);

        &:before {
          content: '';
          position: absolute;
          background-color: inherit;
          top: 0;
          left: 0;
          bottom: 0;
          will-change: left, right;
          animation: indeterminate 2.1s cubic-bezier(0.65, 0.815, 0.735, 0.395) infinite;
        }
        &:after {
          content: '';
          position: absolute;
          background-color: inherit;
          top: 0;
          left: 0;
          bottom: 0;
          will-change: left, right;
          animation: indeterminate-short 2.1s cubic-bezier(0.165, 0.84, 0.44, 1) infinite;
          animation-delay: 1.15s;
        }
      }
    }
  }
}

@keyframes indeterminate {
  0% {
    left: -35%;
    right: 100%;
  }
  60% {
    left: 100%;
    right: -90%;
  }
  100% {
    left: 100%;
    right: -90%;
  }
}

@keyframes indeterminate-short {
  0% {
    left: -200%;
    right: 100%;
  }
  60% {
    left: 107%;
    right: -8%;
  }
  100% {
    left: 107%;
    right: -8%;
  }
}
</style>