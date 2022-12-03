<template>
	<view class="index">
		<swiper class="swiper" autoplay="false" vertical="true" interval="990000" @change="changeVideo">
			<swiper-item v-for="(item,index) in video_list">
				<video :src="item.video_url" preload show-play-btn="true" controls="false" v loop="true"
					:id="`video_${item.video_id}`" objectFit="fill" :enable-progress-gesture="false" @click="clickVideo"
					ref="video_url" play-btn-position="center" class="video" @timeupdate="timeupdate">
				</video>
				<cover-image class="play" v-show="show_play" @tap="videoPlay" src="/static/images/播放2.png">
				</cover-image>
				<cover-view class="cover-view-left">
					<text class="view-left-text">@{{ item.nickname }}</text>
					<view class="view-left-text-content">
						<text class="text-content-text">{{ item.video_describe }}</text>
					</view>
				</cover-view>
			</swiper-item>
		</swiper>
	</view>
</template>
<script>
	let play = false;
	export default {
		data() {
			return {
				video_list: [],
				time: 0,
				barWidth: 0,
				animationData: {},
				times: null,
				play: false,
				show_play: false,
				current_index: 0
			};
		},
		created() {
			this.getVideo()
		},
		methods: {
			getVideo() {
				uni.request({
					url: 'https://bdb24c6d-8c19-4f80-8e7e-c9c9f037f131.bspapp.com/video',
					method: 'POST',
					data: {
						info: 'get_video'
					},
					success: res => {
						console.log(res);
						for (let item of res.data.data) {
							this.video_list.push({
								video_id: item._id,
								nickname: item.username,
								video_describe: item.title,
								video_url: item.src,
								flag: false
							})
						}
						console.log(this.video_list);
					}
				})
			},
			timeupdate(event) {
				let t_w = parseInt(this.width);
				this.duration = event.detail.duration;
				this.time = event.detail.currentTime;
				let width = (this.time / this.duration) * t_w;
				let w = 0;
			},
			clickVideo() {
				// console.log('单视频点击事件');
				this.videoPlay();
			},
			videoPlay() {
				let video_id = this.video_list[this.current_index].video_id;
				if (play) {
					console.log('播放视频', `video_${video_id}`);
					this.videoCtx = uni.createVideoContext(`video_${video_id}`, this);
					this.videoCtx.play();
					this.show_play = false;
					play = false;
				} else {
					console.log('暂停视频', `video_${video_id}`);
					this.videoCtx = uni.createVideoContext(`video_${video_id}`, this);
					this.videoCtx.pause();
					this.show_play = true;
					play = true;
				}
			},
			videoPause() {
				let video_id = this.video_list[this.current_index].video_id;
				this.videoCtx = uni.createVideoContext(`video_${video_id}`, this);
				this.videoCtx.pause();
				this.show_play = true;
				play = true;
			},
			changeVideo(e) {
				// 暂停之前的视频
				this.videoPause();
				this.current_index = e.detail.current;
				console.log(e.detail.current);
				// 播放现在的视频
				this.videoPlay();
				// 判断是否第一条
				if (e.detail.current == 0) {
					console.log('到顶了');
					return false;
				}
				// 判断是否最后一条
				if (e.detail.current == this.video_list.length - 1) {
					console.log('到底了');
					return false;
				}
			}
		},
		watch: {
			play(newVal, oldVal) {
				this.videoPlay();
			}
		}
	};
</script>

<style scoped>
	@import url("../static/css/home.css");
</style>
