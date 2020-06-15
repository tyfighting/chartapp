<template>
  <div class="waveform-wrapper">
    <div class="waveform-container">
      <div id="waveform" class="waveform" ref="waveform"></div>
      <div id="timeline" class="timeline"></div>
      <div class="regions-control" v-if="percents === -1">
        <vue-slider
          v-model="sliderValue"
          :max="5000"
          :tooltip="'none'"
          :enable-cross="false"
          :duration="0.25"
          :clickable="false"
          :adsorb="false"
          @change="handleRegionsChange"
          @drag-end="handleRegionsDragging"
        >
          <template v-slot:dot>
            <div class="custom-dot"></div>
          </template>
        </vue-slider>
      </div>
    </div>
    <div
      class="progress"
      :style="{
        width: `${percents}%`,
        visibility: percents < 0 ? 'hidden' : 'visible'
      }"
    ></div>
    <div class="tools-container">
      <time class="play-time tools-item">{{ currentTime }}</time>
      <label class="previous tools-item"
        ><button class="icon-previous btn" @click="changeUrl(-1)"></button
        >上一条</label
      >
      <label class="backward tools-item"
        ><button
          class="icon-backward btn"
          @click="handleSeekProgress(-1)"
        ></button
        >快退</label
      >
      <label class="play tools-item"
        ><button :class="[isPlayingClass, 'btn']" @click="playMusic"></button
        >{{ isPlaying | isPlayingText }}</label
      >
      <label class="forward tools-item"
        ><button
          class="icon-forward btn"
          @click="handleSeekProgress(1)"
        ></button
        >快进</label
      >
      <label class="next tools-item"
        ><button class="icon-next btn" @click="changeUrl(1)"></button
        >下一条</label
      >
      <span class="split-line tools-item"></span>
      <label class="forward tools-item"
        ><button class="icon-volume btn"></button>音量</label
      >
      <label class="next tools-item"
        ><button class="icon-loop btn" @click="handleLoopStatus"></button
        >{{ isRegionsLoop ? "循环播放" : "正常播放" }}</label
      >
      <label class="refresh tools-item"
        ><button class="icon-refresh btn"></button>刷新</label
      >
      <time class="play-time tools-item">{{ regionStartTime }}s</time>
      <time class="play-time tools-item">{{ regionEndTime }}s</time>
      <div class="tools-volume">
        <i class="el-icon-remove decrease" @click="handleClickVolume(-1)"></i>
        <div class="controls-bar" @click="handleClickVolume(0)">
          <div class="value-bar" :style="{ width: `${volumeValue}%` }"></div>
        </div>
        <i class="el-icon-circle-plus plus" @click="handleClickVolume(1)"></i>
      </div>
    </div>
  </div>
</template>

<script>
import WaveSurfer from "@/../public/js/wavesurfer";
import TimelinePlugin from "@/../public/js/wavesurfer.timeline";
import RegionsPlugin from "@/../public/js/wavesurfer.regions";
import { prefixZero } from "@/utils/utils";
import { timeLineOptions, waveSurferOptions } from "@/config/wavesurfer";
import VueSlider from "vue-slider-component";
import "vue-slider-component/theme/default.css";

export default {
  name: "WaveSurfer",
  data() {
    return {
      isPlaying: false,
      currentTime: "--:--",
      regionStartTime: "-",
      regionEndTime: "-",
      percents: -2,
      volumeValue: 100,
      sliderValue: [0, 5000],
      isRegionsLoop: false,
      audioSourceUrl: ""
    };
  },
  computed: {
    isPlayingClass() {
      return this.isPlaying ? "icon-pause" : "icon-play";
    }
  },
  filters: {
    isPlayingText(boolean) {
      return boolean ? "暂停" : "播放";
    }
  },
  methods: {
    handleRegionsDragging() {
      const distance = 20;
      let [start, end] = this.sliderValue;
      if (end - start < distance) {
        end = start + distance;
        console.log(start, end);
        if (end > 5000) {
          end = end - distance;
          start = start - distance;
        }
        this.sliderValue = [start, end];
        this.updateRegionsBySlider(this.sliderValue);
      }
    },
    handleLoopStatus() {
      this.isRegionsLoop = !this.isRegionsLoop;
      this.playMusic(this.isRegionsLoop);
    },
    changeUrl(value) {
      let audioSourceUrl = this.audioSourceUrl;
      audioSourceUrl =
        Number(audioSourceUrl.substr(0, audioSourceUrl.indexOf("."))) +
        Number(value);
      this.loadAudioWaveforms(`${audioSourceUrl}.mp3`);
    },
    handleRegionsChange(value) {
      this.updateRegionsBySlider(value);
      // 设置regions区域改变标志
      this.RegionsChangeBySlider = true;
    },
    handleSeekProgress(value) {
      // 快退或快进
      value > 0
        ? this.wavesurfer.skipForward()
        : this.wavesurfer.skipBackward();
    },
    handleClickVolume(value) {
      if (value) {
        this.volumeValue += value * 5;
      } else {
        if (!this.$volumeWidth) {
          this.$volumeWidth = event.target.offsetWidth;
        }
        this.volumeValue = (event.offsetX / this.$volumeWidth).toFixed(4) * 100;
      }
      this.wavesurfer.setVolume(this.volumeValue / 100);
    },
    initWaveSurfer() {
      // 初始化实例，并注册监听事件
      if (this.wavesurfer) {
        this.wavesurfer.empty();
      }
      timeLineOptions.container = "#timeline";
      const timelinePlugin = TimelinePlugin.create(timeLineOptions);

      waveSurferOptions.container = this.$refs["waveform"];
      waveSurferOptions.height = this.$refs["waveform"].clientHeight;
      waveSurferOptions.plugins = [timelinePlugin, RegionsPlugin.create()];

      this.wavesurfer = WaveSurfer.create(waveSurferOptions);

      // 对regions添加一系列监听事件，实现各个功能
      this.wavesurfer.on("audioprocess", value => {
        const times = Math.floor(value);
        this.currentTime =
          prefixZero(Math.floor(times / 60)) + ":" + prefixZero(times % 60);
      });
      this.wavesurfer.on("loading", percents => {
        this.percents = percents;
      });
      this.wavesurfer.on("finish", () => {
        this.isPlaying = false;
      });
      this.wavesurfer.on("pause",() => {
        // 监控regions改变，导致循环播放暂停的问题
        if (this.RegionsChangeBySlider) {
          this.wavesurfer.play();
          this.RegionsChangeBySlider = false;
        }
      });
      this.wavesurfer.on("region-created", regions => {
        this.regionStartTime = regions.start;
        this.regionEndTime = regions.end;
      });
      this.wavesurfer.on("region-updated", regions => {
        const regionStart = regions.start.toFixed(2);
        const regionEnd = regions.end.toFixed(2);

        // 判断start更新而不是end更新或者dragging
        if (
          this.regionStartTime !== regionStart &&
          regionEnd === this.regionEndTime
        ) {
          this.regionStartTime = regionStart;
          this.wavesurfer.seekTo(
            this.regionStartTime / this.wavesurfer.durationinseconds
          );
        }
        this.regionEndTime = regionEnd;
      });
      this.wavesurfer.on("region-update-end", regions => {
        const regionStart = regions.start.toFixed(2);
        const regionEnd = regions.end.toFixed(2);
        if (
          regionStart === this.regionStartTime &&
          regionEnd === this.regionEndTime
        ) {
          return false;
        }
        const durationInSeconds = this.wavesurfer.durationinseconds;
        // regions在resizing或dragging时触发事件
        this.regionStartTime = regionStart;
        this.regionEndTime = regionEnd;
        this.wavesurfer.seekTo(regionStart / durationInSeconds);
        const start = (regionStart / durationInSeconds) * 5000;
        const end = (regionEnd / durationInSeconds) * 5000;
        this.sliderValue = [start, end];
      });
      this.loadWavePromise = new Promise((resolve, reject) => {
        this.wavesurfer.on("ready", () => {
          this.wavesurfer.clearRegions();
          this.regionEndTime = this.wavesurfer.durationinseconds = +this.wavesurfer
            .getDuration()
            .toFixed(2);
          this.wavesurfer.addRegion({
            id: "region",
            start: this.regionStartTime,
            end: this.regionEndTime,
            color: "rgba(185, 255, 255, 0.3)",
            resize: false
          });
          this.percents = -1;
          this.playMusic();
          resolve(ready);
        });
        this.wavesurfer.on("error", () => {
          this.$message({
            type: "error",
            message: "抱歉，资源加载失败！",
            duration: 1200
          });
          reject(error);
        });
      });
    },
    // 加载音频，在点击其它信息时由父组件触发调用
    loadAudioWaveforms(url) {
      console.log(url); // demo.mp3
      // url = 'demo.mp3'
      this.audioSourceUrl = url;
      this.wavesurfer.load(this.audioSourceUrl);
      this.sliderValue = [0, 5000];
      return this.loadWavePromise;
    },
    playMusic(isRegionsLoop) {
      if (this.wavesurfer) {
        if (isRegionsLoop === true) {
          this.wavesurfer.regions.list["region"].playLoop();
        } else if (isRegionsLoop === false) {
          this.wavesurfer.regions.list["region"].un("out", this.playLoop);
          this.wavesurfer.play(
            this.wavesurfer.getCurrentTime(),
            this.wavesurfer.getDuration()
          );
        } else {
          this.wavesurfer.playPause();
        }
        this.isPlaying = this.wavesurfer.isPlaying();
      } else {
        this.$message({
          type: "error",
          message: "请选择要播放的记录！",
          duration: 2000
        });
      }
    },
    updateRegionsBySlider(value) {
      const durationInSeconds = this.wavesurfer.durationinseconds;
      const start = ((value[0] / 5000) * durationInSeconds).toFixed(2);
      const end = ((value[1] / 5000) * durationInSeconds).toFixed(2);
      this.wavesurfer.regions.list["region"].update({
        start: start,
        end: end
      });
    }
  },
  mounted() {
    this.initWaveSurfer();
    this.loadAudioWaveforms("2.mp3");
  },
  beforeDestroy() {
    this.wavesurfer.empty();
    this.wavesurfer.destroy();
    this.wavesurfer = null;
    console.log("beforeDestroy");
  },
  components: {
    VueSlider
  }
};
</script>
<style lang="scss" scoped>
.waveform-wrapper {
  margin: 0 auto 16px;
  width: 1000px;
  height: 280px;
  box-sizing: border-box;
  background-image: url("../assets/image/wavesurfer_bg.png");
  background-size: 100% 100%;
  border-radius: 24px;
  overflow: hidden;
  position: relative;
  .waveform-container {
    margin: 20px auto 0;
    width: 1000px;
    position: relative;
    .waveform {
      height: 180px;
      box-sizing: border-box;
    }
    .timeline {
      margin: 0 auto;
      width: 1000px;
      min-height: 20px;
    }
    .regions-control {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      width: 1000px;
      height: 20px;
      z-index: 1111;
      .custom-dot {
        width: 16px;
        height: 10px;
        color: #fff;
        background-color: currentColor;
        cursor: pointer;
        position: relative;
        &::before {
          content: "";
          position: absolute;
          left: 0;
          top: -6px;
          width: 0;
          height: 0;
          border-left: 8px solid transparent;
          border-right: 8px solid transparent;
          border-bottom: 6px solid currentColor;
        }
      }
    }
  }
}
.tools-container {
  margin-left: 4px;
  height: 58px;
  padding: 0 25px;
  font-size: 0.14px;
  color: white;
  box-shadow: inset 0 0 10px 0 #41b9ea;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  .split-line {
    width: 1px;
    height: 36px;
    background: rgba(47, 213, 255, 1);
    opacity: 0.5;
  }
  .tools-item {
    margin-right: 10px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    .btn {
      padding: 0;
      font-size: 1.6em;
      color: #027ff2ff;
      border: none;
      outline: none;
      background-color: transparent;
      cursor: pointer;
      transition: all 0.25s cubic-bezier(0, 0.79, 0.64, 1.04);
      &:active {
        transform: translate3d(2px, 2px, 2px);
      }
    }
  }
  .play-time {
    height: 36px;
    line-height: 36px;
    padding: 10px;
    font-size: 20px;
    font-weight: bold;
    color: rgba(47, 213, 255, 1);
    letter-spacing: 0.1em;
    box-shadow: inset 0 0 5px 2px #120d76;
    box-sizing: border-box;
    cursor: default;
  }
  .refresh {
    margin-right: auto;
  }
  .tools-volume {
    width: 198px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    .controls-bar {
      width: 142px;
      height: 5px;
      background-color: #08097eff;
      border-radius: 2px;
      overflow: hidden;
      position: relative;
      .value-bar {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        background-color: #027ff2ff;
        border-radius: 2px;
      }
    }
    .plus,
    .decrease {
      font-size: 1rem;
      color: #027ff2ff;
      cursor: pointer;
    }
  }
}
.progress {
  position: absolute;
  left: 0;
  right: 0;
  top: 100px;
  height: 20px;
  z-index: 1;
  &::before {
    position: absolute;
    left: 0;
    top: 0;
    content: "";
    width: 100%;
    height: 100%;
    padding: 0 10px;
    box-sizing: border-box;
    background-clip: content-box;
    background-image: linear-gradient(
      45deg,
      rgba(255, 255, 255, 0.15) 25%,
      transparent 25%,
      transparent 50%,
      rgba(255, 255, 255, 0.15) 50%,
      rgba(255, 255, 255, 0.15) 75%,
      transparent 75%,
      transparent
    );
    background-color: #337ab7;
    background-repeat: repeat;
    background-size: 40px 40px;
    transition: width 0.6s ease;
    animation: progress-bar-stripes 2s linear infinite;
    border-radius: 5px;
    z-index: 1;
  }
  &::after {
    position: absolute;
    left: 0;
    top: 0;
    content: "";
    width: 1000px;
    height: 100%;
    padding: 0 10px;
    background-color: #81b5e2;
    background-clip: content-box;
    box-sizing: border-box;
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    z-index: 0;
  }
  @keyframes progress-bar-stripes {
    from {
      background-position: 0 0;
    }
    to {
      background-position: -80px 0;
    }
  }
}
</style>
