<script setup lang="ts">
  import { ref } from 'vue';
  import { useDropzone } from 'vue3-dropzone';
  import { CancelTokenSource } from 'axios';
  import {
    AudioWaveform,
    File,
    FileImage,
    FolderArchive,
    UploadCloud,
    Video,
    X,
  } from 'lucide-vue-next';
  import { Progress } from '@/components/ui/progress'
  import { ScrollArea } from '@/components/ui/scroll-area';
  
  interface FileUploadProgress {
    progress: number;
    File: File;
    source: CancelTokenSource | null;
  }
  
  enum FileTypes {
    Image = 'image',
    Pdf = 'pdf',
    Audio = 'audio',
    Video = 'video',
    Other = 'other',
  }
  
  const ImageColor = {
    bgColor: 'bg-purple-600',
    fillColor: 'fill-purple-600',
  };
  
  const PdfColor = {
    bgColor: 'bg-blue-400',
    fillColor: 'fill-blue-400',
  };
  
  const AudioColor = {
    bgColor: 'bg-yellow-400',
    fillColor: 'fill-yellow-400',
  };
  
  const VideoColor = {
    bgColor: 'bg-green-400',
    fillColor: 'fill-green-400',
  };
  
  const OtherColor = {
    bgColor: 'bg-gray-400',
    fillColor: 'fill-gray-400',
  };
  
  const props = defineProps({
    accept: {
      type: String,
      default: '*',
    },
  });
  const uploadedFiles = ref<File[]>([]);
  const filesToUpload = ref<FileUploadProgress[]>([]);
  
  const getFileIconAndColor = (file: File) => {
    if (file.type.includes(FileTypes.Image)) {
      return {
        icon: FileImage,
        size: 40,
        class: ImageColor.fillColor,
        color: ImageColor.bgColor,
      };
    }

    if (file.type.includes(FileTypes.Pdf)) {
      return {
        icon: File,
        size: 40,
        class: PdfColor.fillColor,
        color: PdfColor.bgColor,
      };
    }

    if (file.type.includes(FileTypes.Audio)) {
      return {
        icon: AudioWaveform,
        size: 40,
        class: AudioColor.fillColor,
        color: AudioColor.bgColor,
      };
    }

    if (file.type.includes(FileTypes.Video)) {
      return {
        icon: Video,
        size: 40,
        class: VideoColor.fillColor,
        color: VideoColor.bgColor,
      };
    }

    return {
      icon: FolderArchive,
      size: 40,
      class: OtherColor.fillColor,
      color: OtherColor.bgColor,
    };
  };
  
  // Unused function for now but can be used to track upload progress with axios
  /* const onUploadProgress = (
    progressEvent: AxiosProgressEvent,
    file: File,
    cancelSource: CancelTokenSource
  ) => {
    const progress = Math.round(
      (progressEvent.loaded / (progressEvent.total ?? 0)) * 100
    );

    if (progress === 100) {
      uploadedFiles.value = [...uploadedFiles.value, file];

      filesToUpload.value = filesToUpload.value.filter(
        (item) => item.File !== file
      );

      return;
    }

    filesToUpload.value = filesToUpload.value.map((item) => {
      if (item.File.name === file.name) {
        return {
          ...item,
          progress,
          source: cancelSource,
        };
      } else {
        return item;
      }
    });
  }; */
  
  const removeFile = (file: File) => {
    filesToUpload.value = filesToUpload.value.filter(
      (item) => item.File !== file
    );

    uploadedFiles.value = uploadedFiles.value.filter((item) => item !== file);
  };
  
  const onDrop = async (acceptedFiles: File[]) => {
    filesToUpload.value = [
      ...filesToUpload.value,
      ...acceptedFiles.map((file) => {
        return {
          progress: 0,
          File: file,
          source: null,
        };
      }),
    ];
  };

  const { getRootProps, getInputProps } = useDropzone({ onDrop });
</script>

<template>
    <div>
      <div>
        <label
          v-bind="getRootProps()"
          class="relative flex flex-col items-center justify-center w-full py-6 border-2 border-gray-300 border-dashed rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100"
        >
          <div class="text-center">
            <div class="border p-2 rounded-md max-w-min mx-auto">
              <UploadCloud :size="20" />
            </div>
  
            <p class="mt-2 text-sm text-gray-600">
              <span class="font-semibold">Drag files</span>
            </p>
            <p class="text-xs text-gray-500">
              Click to upload files (files should be under 10 MB)
            </p>
          </div>
        </label>
  
        <input
          v-bind="getInputProps()"
          id="dropzone-file"
          :accept="props.accept"
          type="file"
          class="hidden"
        />
      </div>
  
      <div v-if="filesToUpload.length > 0">
        <ScrollArea class="h-40">
          <p class="font-medium my-2 mt-6 text-muted-foreground text-sm">
            Files to upload
          </p>
          <div class="space-y-2 pr-3">
            <div
              v-for="fileUploadProgress in filesToUpload"
              :key="fileUploadProgress.File.lastModified"
              class="flex justify-between gap-2 rounded-lg overflow-hidden border border-slate-100 group hover:pr-0 pr-2"
            >
              <div class="flex items-center flex-1 p-2">
                <div class="text-white">
                  <component 
                  :is="getFileIconAndColor(fileUploadProgress.File).icon"
                  :size="getFileIconAndColor(fileUploadProgress.File).size"
                  :class="getFileIconAndColor(fileUploadProgress.File).class"
                  />
                </div>
  
                <div class="w-full ml-2 space-y-1">
                  <div class="text-sm flex justify-between">
                    <p class="text-muted-foreground">
                      {{ fileUploadProgress.File.name.slice(0, 25) }}
                    </p>
                    <span class="text-xs">
                      {{ fileUploadProgress.progress }}%
                    </span>
                  </div>
                  <Progress
                    :model-value="fileUploadProgress.progress"
                    :color="getFileIconAndColor(fileUploadProgress.File).color"
                  />
                </div>
              </div>
              <button
                @click="
                  () => {
                    if (fileUploadProgress.source)
                      fileUploadProgress.source.cancel('Upload cancelled');
                    removeFile(fileUploadProgress.File);
                  }
                "
                class="bg-red-500 text-white transition-all items-center justify-center cursor-pointer px-2 hidden group-hover:flex"
              >
                <X :size="20" />
              </button>
            </div>
          </div>
        </ScrollArea>
      </div>
  
      <div v-if="uploadedFiles.length > 0">
        <p class="font-medium my-2 mt-6 text-muted-foreground text-sm">
          Uploaded Files
        </p>
        <div class="space-y-2 pr-3">
          <div
            v-for="file in uploadedFiles"
            :key="file.lastModified"
            class="flex justify-between gap-2 rounded-lg overflow-hidden border border-slate-100 group hover:pr-0 pr-2 hover:border-slate-300 transition-all"
          >
            <div class="flex items-center flex-1 p-2">
              <div class="text-white">
                {{ getFileIconAndColor(file).icon }}
              </div>
              <div class="w-full ml-2 space-y-1">
                <div class="text-sm flex justify-between">
                  <p class="text-muted-foreground">
                    {{ file.name.slice(0, 25) }}
                  </p>
                </div>
              </div>
            </div>
            <button
              @click="() => removeFile(file)"
              class="bg-red-500 text-white transition-all items-center justify-center px-2 hidden group-hover:flex"
            >
              <X :size="20" />
            </button>
          </div>
        </div>
      </div>
    </div>
  </template>