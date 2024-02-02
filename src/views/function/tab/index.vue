<script setup lang="ts">
import { ref } from 'vue';
import { useRouterPush } from '@/hooks/common/router';
import { $t } from '@/locales';
import { useTabStore } from '@/store/modules/tab';
import axios from 'axios';

const tabStore = useTabStore();
const { routerPushByKey } = useRouterPush();

const tabLabel = ref('');

function changeTabLabel() {
  tabStore.setTabLabel(tabLabel.value);
}

function resetTabLabel() {
  tabStore.resetTabLabel();
}

const title = ref<string>('');
const subtitle = ref<string>('');
const videoFile = ref<File | null>(null);

const handleVideo = (event: Event) => {
  const input = event.target as HTMLInputElement;
  if (!input.files) return;
  videoFile.value = input.files[0];
};

const generateVideo = async () => {
  if (!videoFile.value) return;
  const formData = new FormData();
  formData.append('resource', JSON.stringify({
    title: title.value,
    subtitle: subtitle.value
  }));
  formData.append('video', videoFile.value);

  try {
    const response = await axios.post('http://52.80.87.28:8001/translate', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', 'translated_video.mp4');
    document.body.appendChild(link);
    link.click();
  } catch (error) {
    console.error('生成视频时出错:', error);
  }
};
</script>


<template>
  <div>
    <NButton @click="routerPushByKey('function_multi-tab')">
      {{ $t('跳转') }}
    </NButton>
  </div>
  <div id="app" class="container">
    <div class="form-section">
      <!-- Title Input -->
      <div class="input-group">
        <label for="title">标题</label>
        <input type="text" id="title" v-model="title" placeholder="标题" maxlength="100" class="input" />
      </div>

      <!-- Subtitle Input -->
      <div class="input-group">
        <label for="subtitle">副标题</label>
        <input type="text" id="subtitle" v-model="subtitle" placeholder="副标题" maxlength="200" class="input" />
      </div>

      <!-- Video Upload and Submit Button Group -->
      <div class="button-group">
        <input type="file" id="video-upload" @change="handleVideo" class="input-file" />
        <label for="video-upload" class="button">上传视频</label>

        <button @click="generateVideo" class="button">生成视频</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
      padding: 20px;
      width: 100%;
      max-width: 600px;
      margin: auto;
      box-sizing: border-box;
    }
  
    .input-group {
      display: flex;
      flex-direction: column;
      margin-bottom: 20px;
    }
  
    label {
      font-weight: bold;
      margin-bottom: 10px;
      font-size: medium;
    }
  
    .input, .input-file {
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      font-size: medium;
    }
  
  
  
    .button-group {
      display: flex;
      justify-content: space-between;
    }
  
    .button {
      background-color: #4CAF50;
      color: white;
      cursor: pointer;
      border: none;
      border-radius: 4px;
      padding: 10px 15px;
      font-size: medium;
      flex-grow: 1; /* Make buttons grow equally */
      margin: 5px; /* Spacing between buttons */
    }
</style>
