<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';
import { useRouterPush } from '@/hooks/common/router';
import { $t } from '@/locales';

interface ImageFile {
  url: string;
  file: File;
}

const { routerPushByKey } = useRouterPush();

const title = ref<string>('');
const subtitle = ref<string>('');
const text = ref<string>('');
const images = ref<ImageFile[]>([]);
const draggingIndex = ref<number | null>(null);


const handleFiles = (event: Event) => {
  const input = event.target as HTMLInputElement;
  if (!input.files) return;
  
  const filesArray: File[] = Array.from(input.files); // Convert FileList to an array
  
  for (let file of filesArray) {
    const url = URL.createObjectURL(file);
    images.value.push({ url, file });
  }
};

const generateVideo = async () => {
  const formData = new FormData();
  formData.append('resource', JSON.stringify({
    title: title.value,
    subtitle: subtitle.value,
    text: text.value
  }));

  images.value.forEach((image, index) => {
    formData.append(`images[${index}]`, image.file);
  });

  try {
    const response = await axios.post('http://52.80.87.28:8000/', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', 'video.mp4');
    document.body.appendChild(link);
    link.click();
  } catch (error) {
    console.error('生成视频时出错:', error);
  }
};

const removeImage = (index: number) => {
  images.value.splice(index, 1);
};

const dragStart = (index: number) => {
  draggingIndex.value = index;
};

const drop = (index: number) => {
  if (draggingIndex.value !== null) {
    const draggedImage = images.value[draggingIndex.value];
    images.value.splice(draggingIndex.value, 1);
    images.value.splice(index, 0, draggedImage);
    draggingIndex.value = null;
  }
};
</script>


<template>
  <div>
    <div>
      <NButton @click="routerPushByKey('function_tab')">{{ $t('返回') }}</NButton>
    </div>
  </div>
  <div id="app" class="container">

    <div class="form-section">
      <!-- Title Input -->
      <div class="input-group">
        <label for="title">标题</label>
        <input type="text" id="title" v-model="title" placeholder="标题" maxlength="20" class="input" />
      </div>

      <!-- Subtitle Input -->
      <div class="input-group">
        <label for="subtitle">副标题</label>
        <input type="text" id="subtitle" v-model="subtitle" placeholder="副标题" maxlength="40" class="input" />
      </div>

      <!-- Text Input -->
      <div class="input-group">
        <label for="text">文案</label>
        <textarea id="text" v-model="text" placeholder="文案" maxlength="160" class="textarea"></textarea>
      </div>

      <!-- Buttons -->
      <div class="button-group">
        <button @click="generateVideo" class="button">生成视频</button>
        <input type="file" id="image-upload" @change="handleFiles" multiple class="input" style="display: none;" />
        <label for="image-upload" class="button">上传图片</label>
      </div>
    </div>

    <!-- Image Preview Section -->
    <div class="image-preview">
      <div
        v-for="(image, index) in images"
        :key="index"
        draggable="true"
        @dragstart="dragStart(index)"
        @dragover.prevent
        @drop="drop(index)"
        class="image-container"
      >
        <img :src="image.url" class="preview-image" />
        <button @click="removeImage(index)" class="delete-button">✖</button>
      </div>
    </div>
  </div>
</template>

<style>
  .container {
    display: grid;
    grid-template-columns: 60% 40%;
    gap: 20px;
    padding: 20px;
    min-height: 100%;
    width: 250%;
    max-width: 100%;
    box-sizing: border-box;
  }

  .input-group {
    display: flex;
    flex-direction: column;
    margin-bottom: 50px;
  }

  label {
    font-weight: bold;
    margin-bottom: 5px;
    font-size: large;
  }

  .input, .textarea {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: large;
  }

  .input {
    width: calc(100% - 20px);
  }

  .textarea {
    min-height: 150px;
    width: calc(100% - 20px);
    font: 1em sans-serif;
    font-size: medium;
  }

  .button {
    background-color: #4CAF50;
    color: white;
    cursor: pointer;
    border: none;
    border-radius: 4px;
    padding: 10px 15px;
    font-size: large;
  }

  .image-preview {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    align-content: flex-start;
    padding: 10px;
    border: 1px solid #ddd;
  }

  .image-container {
    position: relative;
    padding: 5px;
  }

  .preview-image {
    width: 100px;
    height: 100px;
    object-fit: cover;
    border-radius: 5px;
  }

  .button-group {
    display: flex;
    justify-content: space-between;
    width: 100%;
  }

  .delete-button {
    position: absolute;
    top: 5px;
    right: 5px;
    background-color: red;
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 50%;
    padding: 5px;
    width: 20px;
    height: 20px;
    display: none;
    align-items: center;
    justify-content: center;
    text-align: center;
    line-height: 20px;
  }

  .image-container:hover .delete-button {
    display: flex;
  }

</style>