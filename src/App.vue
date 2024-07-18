<script setup lang="ts">
import imageUpload from '@/components/custom/file-upload.vue';
import { MoveRight } from 'lucide-vue-next';
import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle, DialogDescription } from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
</script>

<template>
  <main class="flex justify-center items-center min-h-screen flex-col space-y-4">
      <h1 class="text-5xl font-semibold">File Vault</h1>
      <p class="text-muted-foreground">File upload component for Vue</p>
      <div class="flex gap-4">
        <Dialog>
          <DialogTrigger asChild>
            <Button class="rounded-full shadow" variant="outline">
              File upload
            </Button>
          </DialogTrigger>
          <DialogContent class="sm:max-w-[425px]">
            <DialogHeader>
              <DialogTitle class="text-center">
                Upload your files
              </DialogTitle>
              <DialogDescription class="text-center">
                The only file upload you will ever need
              </DialogDescription>
            </DialogHeader>
            <div class="grid gap-4 py-4">
              <imageUpload />
            </div>
          </DialogContent>
        </Dialog>
        <a class="flex gap-1 items-center" href="https://github.com/ThomasTartrau/file-vault-vue">
          Github
          <MoveRight :size=15 />
        </a>
      </div>
    </main>
</template>
