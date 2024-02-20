<template>
    <div>
        <van-uploader
        preview-size="100px"
        max-count="5"
        name="attachments"
        accept=".png,.jpg,.jpeg"
        :upload-text="$t('account.upload_image')"
        :multiple="true"
        @change="compressImage"
        :preview="true"
                >
    </div>
</template>

<script setup lang="ts">
  const compressImage = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) return;

  const reader = new FileReader();

  reader.onload = e => {
    const image = new Image();
    image.onload = () => {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');

      // Set the canvas dimensions to the resized dimensions
      const maxWidth = 800; // Max width for resized image
      const maxHeight = 600; // Max height for resized image
      let width = image.width;
      let height = image.height;

      if (width > height) {
        if (width > maxWidth) {
          height *= maxWidth / width;
          width = maxWidth;
        }
      } else {
        if (height > maxHeight) {
          width *= maxHeight / height;
          height = maxHeight;
        }
      }

      canvas.width = width;
      canvas.height = height;

      // Draw image on canvas
      context?.drawImage(image, 0, 0, width, height);

      // Convert canvas to Blob
      canvas.toBlob(
        (blob: Blob | null) => {
          if (!blob) return;

          // Compress image until its size is below 500 KB
          let quality = 0.7; // Initial quality
          let compressedBlob: any = blob;
          while (compressedBlob?.size > 500 * 1024 && quality > 0) {
            quality -= 0.1;
            compressedBlob = canvas.toBlob(blob => blob, 'image/jpeg', quality);
          }
          console.log('blog', typeof compressedBlob);
          // Create a new file from the compressed blob
          const compressedFile: File = new File([compressedBlob], file.name, {
            type: 'image/jpeg',
            lastModified: Date.now()
          });

          // Log the file size in KB
          const fileSizeKB = compressedFile.size / 1024;
          console.log('Compressed file size:', fileSizeKB.toFixed(2), 'KB');

          // Now you can upload the compressedFile
          console.log('Compressed file:', compressedFile);
        },
        'image/jpeg',
        0.7
      );
    };

    image.src = e.target?.result as string;
  };

  reader.readAsDataURL(file);
};

</script>

<style scoped>

</style>