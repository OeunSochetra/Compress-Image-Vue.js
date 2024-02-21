<template>
    <div>
        <div class="upload">
        <van-uploader
            preview-size="100px"
            v-model="attachments"
            max-count="5"
            name="attachments"
            accept=".png,.jpg,.jpeg"
            :upload-text="$t('account.upload_image')"
            :multiple="true"
            :preview="true"
            :before-read="beforeRead"
            @after-read="afterRead"
                >
    </div>
</template>

<script setup lang="ts">
const compressedImages = ref<string[]>([]);
  const isCompressing = ref<boolean>(false);

  const beforeRead = async (file: File | File[]) => {
    isCompressing.value = true;
    if (Array.isArray(file)) {
      // Handle multiple files
      const compressedFiles: File[] = [];
      for (const f of file) {
        try {
          const compressedFile = await compressImage(f);
          compressedFiles.push(compressedFile);
        } catch (error) {
          console.error('Error compressing image:', error);
          throw error;
        }
      }
      isCompressing.value = false;
      return compressedFiles;
    } else {
      // Handle single file
      try {
        return await compressImage(file);
      } catch (error) {
        console.error('Error compressing image:', error);
        isCompressing.value = false;
        throw error;
      }
    }
  };

  const compressImage = async (file: File) => {
    console.log('Original file:', file);
    const options = {
      maxSizeMB: 0.5,
      maxWidthOrHeight: 500,
      useWebWorker: true
    };

    const compressedFile = await imageCompression(file, options);
    console.log('Compressed file:', compressedFile);
    isCompressing.value = false;
    return compressedFile;
  };

  const afterRead = (files: File[]) => {
    console.log('originalFIle', files);

    // Process the compressed image files as needed
    for (const file of files) {
      const imageUrl = URL.createObjectURL(file);
      compressedImages.value.push(imageUrl);
      isCompressing.value = false;
    }
  };
</script>

<style scoped>

</style>