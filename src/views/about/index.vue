<template>
  <div class="video-editor">
    <input type="file" @change="handleVideoUpload" accept="video/*" />
    <video ref="videoPlayer" @timeupdate="updateTime" @loadedmetadata="setVideoDuration" v-if="videoSrc">
      <source :src="videoSrc" type="video/mp4">
      Your browser does not support the video tag.
    </video>

    <div class="slider-container">
      <div class="slider">
        <!-- 移除数字输入框的代码 -->
        <div class="slider-range-wrapper" :style="styles" :class="{ 'o-range': isRange }">
          <div class="slider-value left-value" :style="{ left: leftValuePosition }">
            {{ formatValue(rangeValue[0]) }}
          </div>

          <div class="slider-value right-value" :style="{ left: rightValuePosition }">
            {{ formatValue(rangeValue[1]) }}
          </div>
          <input
          type="range"
          v-model.number="rangeValue[0]"
          :min="min"
          :max="max"
          :step="step"
          @input="emitRange"
          class="slider-range o-lower"
        />
        <input
          type="range"
          v-model.number="rangeValue[1]"
          :min="min"
          :max="max"
          :step="step"
          v-if="isRange"
          @input="emitRange"
          class="slider-range o-upper"
        />

        </div>
      </div>
    </div>
  </div>
</template>


<script lang="ts">
import { defineComponent, ref, computed, watch } from 'vue';
import { nextTick } from 'vue';
export default defineComponent({
  name: 'VideoEditor',
  setup() {
    const videoPlayer = ref<HTMLVideoElement | null>(null);
    const videoSrc = ref<string | null>(null);
    const currentTime = ref(0);
    const videoDuration = ref(0);
    const rangeValue = ref([0, 0]);
    const inputValue = ref([0, 0]);
    const min = ref(0);
    const max = ref(150);
    const step = ref(0.1);

    const handleVideoUpload = (event: Event) => {
      const files = (event.target as HTMLInputElement).files;
      if (files && files[0]) {
        const file = files[0];
        videoSrc.value = URL.createObjectURL(file);
      }
    };

    watch(() => videoPlayer.value?.duration, (newDuration) => {
      if (newDuration) {
        videoDuration.value = newDuration;
        rangeValue.value = [0, newDuration];
      }
    });

    const setVideoDuration = () => {
      if (videoPlayer.value) {
        console.log("Video duration set, player:", videoPlayer.value);
        videoDuration.value = videoPlayer.value.duration;
        max.value = videoPlayer.value.duration;
        rangeValue.value = [0, videoPlayer.value.duration];
      }
    };



    const updateTime = () => {
      if (videoPlayer.value) {
        currentTime.value = videoPlayer.value.currentTime;
      }
    };

    const isRange = computed(() => Array.isArray(rangeValue.value));

    const rangeMaxLower = computed(() => rangeValue.value[1]);
    const rangeMinUpper = computed(() => rangeValue.value[0]);

    const validLower = computed(() => 
      inputValue.value[0] >= 0 && inputValue.value[0] <= rangeMaxLower.value
    );

    const validUpper = computed(() => 
      inputValue.value[1] >= rangeMinUpper.value && inputValue.value[1] <= videoDuration.value
    );

    const styles = computed(() => {
      // 定义 styles 对象的类型，允许任意字符串键和字符串值
      const styles: Record<string, string> = {};
      const lowerPercent = ((rangeValue.value[0] - 0) / (videoDuration.value - 0)) * 100;
      const upperPercent = ((rangeValue.value[1] - 0) / (videoDuration.value - 0)) * 100;
      styles["--left"] = `${Math.min(Math.max(lowerPercent, 0), 100)}%`;
      styles["--width"] = `${Math.min(Math.max(upperPercent - lowerPercent, 0), 100)}%`;
      return styles;
    });

    const leftValuePosition = computed(() => {
      const percent = ((rangeValue.value[0] - min.value) / (max.value - min.value)) * 100;
      return `${percent}%`;
    });

    const rightValuePosition = computed(() => {
      const percent = ((rangeValue.value[1] - min.value) / (max.value - min.value)) * 100;
      return `${percent}%`;
    });

    const formatValue = (value: number) => {
      return value.toFixed(2);
    };




    const emitRange = async () => {
      if (videoPlayer.value) {
        videoPlayer.value.currentTime = rangeValue.value[0];
        await nextTick(); // 等待 DOM 更新
        console.log("Video player currentTime set to", videoPlayer.value.currentTime);
      }
    };


    const inputDone = () => {
      if (validLower.value && validUpper.value) {
        rangeValue.value = [...inputValue.value];
        // 更新视频剪辑时间
      }
    };

    const inputCancel = () => {
      inputValue.value = [...rangeValue.value];
    };

    return { 
      videoPlayer, videoSrc, currentTime, videoDuration, rangeValue, inputValue,
      handleVideoUpload, setVideoDuration, updateTime,
      isRange, rangeMaxLower, rangeMinUpper, validLower, validUpper, styles,
      emitRange, inputDone, inputCancel, min, max, step, leftValuePosition, rightValuePosition, formatValue
    };
  }
});
</script>

<style lang="scss">
video {
  max-width: 20%;
  height: 60%;
}

$slider-bg: #e5e5e5;
$black-border: rgba(0, 0, 0, 0.15);
$black-divider: rgba(0, 0, 0, 0.1);
$primary: #1890ff;
$danger: #f5222d;
$white: #fff;

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.slider {
  position: relative;
  margin: 0 16px;
  padding: 12px 0;
  width: 320px;

  &-head {
    display: flex;
    justify-content: space-between;
  }

  &-name {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 14px;
    color: rgba(0, 0, 0, 0.65);

    .svg-icon {
      width: 16px;
      height: 16px;
      margin-right: 8px;
    }
  }

  &-input {
    width: 72px;
    height: 24px;
    border-radius: 4px;
    padding: 0 8px;
    border: 1px solid $black-border;
    font-size: 12px;

    &:hover,
    &:focus {
      border: 1px solid $primary;
      outline: none;
    }

    &.o-error {
      border: 1px solid $danger;
    }
  }

  &-info {
    display: flex;

    &-separator {
      margin: 0 4px;
    }
  }

  &-range {
    position: relative;
    -webkit-appearance: none;
    margin: 0;
    outline: none;
    border: none;
    width: 100%;
    top: 2px;
    height: 4px;
    background-color: transparent;
    z-index: 2;

    &::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 16px;
      height: 16px;
      box-sizing: border-box;
      border-radius: 50%;
      border: solid 2px $primary;
      background-color: $white;
      pointer-events: auto;
      cursor: pointer;
    }

    &.o-upper {
      position: absolute;
      top: 12px;
      left: 0;
      width: 100%;
    }

    &-wrapper {
      position: relative;
      height: 24px;

      &.o-range {
        input {
          pointer-events: none;
        }

        &::before {
          margin-left: var(--left);
        }
      }

      &::before {
        content: "";
        position: absolute;
        top: 12px;
        height: 4px;
        left: 0;
        width: var(--width);
        border-radius: 2px 0 0 2px;
        background-color: $primary;
        pointer-events: none;
        z-index: 1;
      }

      &::after {
        content: "";
        position: absolute;
        top: 12px;
        height: 4px;
        left: 0;
        right: 0;
        border-radius: 2px;
        background-color: $slider-bg;
        pointer-events: none;
      }
    }
  }
}
.slider-value {
  position: absolute;
  top: -20px; // 调整以适应设计
  transform: translateX(-50%); // 使文本居中
  background-color: $white;
  padding: 2px 5px;
  border-radius: 4px;
  font-size: 12px;

  &.left-value {
    // 针对左滑块数值的特定样式
  }

  &.right-value {
    // 针对右滑块数值的特定样式
  }
}
</style>
