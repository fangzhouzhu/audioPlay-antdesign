<template>
  <div>
    <audio
      ref="audio"
      style="display:none"
      @pause="onPause"
      @play="onPlay"
      @timeupdate="onTimeupdate"
      @loadedmetadata="onLoadedmetadata"
      :src="audiourl"
      controls="controls"
    ></audio>
    <!-- 音频播放控件 -->
    <div class="out-box">
      <span class="box">
        <a-icon
          :type="icon_type"
          @click="startPlayOrPause"
          style="font-size:30px;color:#40a9ff"
          theme="filled"
        />
      </span>
      <span class="box slider">
        <a-slider
          :tip-formatter="formatProcessToolTip"
          @change="changeCurrentTime"
          v-model="sliderTime"
          class="slider"
          :step="1"
          :max="audio.maxTime"
          :min="0"
          id="test"
          :default-value="0"
          style="width:100%"
        ></a-slider>
      </span>
      <span
        class="box"
        style="margin-left:10px"
      >{{audio.currentTime|formatSecond}} / {{audio.maxTime|formatSecond}}</span>
      <div class="speed box" @click="changeSpeed">
        <span>{{audio.playbackRate}}X</span>
      </div>
    </div>
  </div>
</template>

<script>
// 将整数转换成 时：分：秒的格式
function realFormatSecond(second) {
  var secondType = typeof second;

  if (secondType === 'number' || secondType === 'string') {
    second = parseInt(second);

    var hours = Math.floor(second / 3600);
    second = second - hours * 3600;
    var mimute = Math.floor(second / 60);
    second = second - mimute * 60;
    hours = hours < 10 ? '0' + hours : hours;
    return (
      hours + ':' + ('0' + mimute).slice(-2) + ':' + ('0' + second).slice(-2)
    );
  } else {
    return '00:00:00';
  }
}

export default {
  props: ['audiourl'],
  data() {
    return {
      icon_type: 'play-circle',
      sliderTime: 200,
      audio: {
        // 该字段是音频是否处于播放状态的属性
        playing: false,
        // 音频当前播放时长
        currentTime: 0,
        // 音频最大播放时长
        maxTime: 0,
        // 倍速
        playbackRate: 1,
      },
    };
  },
  methods: {
    // 控制音频的播放与暂停
    startPlayOrPause() {
      if (this.icon_type == 'pause-circle') {
        this.icon_type = 'play-circle';
      } else {
        this.icon_type = 'pause-circle';
      }
      this.audio.playing = !this.audio.playing;
      return this.audio.playing ? this.play() : this.pause();
    },
    // 播放音频
    play() {
      this.$refs.audio.play();
    },
    // 暂停音频
    pause() {
      this.$refs.audio.pause();
    },
    // 当音频播放
    onPlay() {
      this.audio.playing = true;
    },
    // 当音频暂停
    onPause() {
      this.audio.playing = false;
    },

    // 当加载语音流元数据完成后，会触发该事件的回调函数
    // 语音元数据主要是语音的长度之类的数据
    onLoadedmetadata(res) {
      this.audio.maxTime = parseInt(res.target.duration);
    },
    changeCurrentTime(index) {
      this.$refs.audio.currentTime = parseInt(index);
    },
    // 当音频当前时间改变后，进度条也要改变
    onTimeupdate(res) {
      this.audio.currentTime = parseInt(res.target.currentTime);
      this.sliderTime = this.audio.currentTime;
      if (this.audio.currentTime == this.audio.maxTime) {
        // this.onPause();
        this.startPlayOrPause();
      }
    },

    // 进度条格式化toolTip
    formatProcessToolTip(index = 0) {
      index = parseInt(index);
      return '进度条: ' + realFormatSecond(index);
    },
    changeSpeed() {
      if (this.audio.playbackRate == 2) {
        this.audio.playbackRate = 1;
      } else {
        this.audio.playbackRate += 0.5;
      }
      this.$refs.audio.playbackRate = this.audio.playbackRate;
    },
  },
  filters: {
    // 将整数转化成时分秒
    formatSecond(second = 0) {
      return realFormatSecond(second);
    },
  },
};
</script>

<style lang='less' scoped>
.speed {
  width: 50px;
  height: 26px;
  background: #40a9ff;
  border-radius: 6px;
  text-align: center;
  color: #fff;
  line-height: 26px;
  cursor: pointer;
}
.out-box {
  display: flex;
  height: 40px;
  align-items: center;
}
.box {
  margin-right: 15px;
}
.slider {
  width: calc(100% - 280px);
}
</style>
<style lang='less'>
.box {
  .ant-slider {
    margin: 10px 6px 10px;
  }
}
</style>