<template>
	<div id="PcIndex">
		<div class="glass">
			<div class="ttsPage">
				<!--左侧发音人列表-->
				<div class="ttsPage-left">
					<div class="vocalList">
						<div
							:class="list.status == 'true' ? (chickId == index ? 'chickId' : 'canChoose') : 'cantChoose'"
							:key="index"
							@click="chickVoice(index)"
							class="vocal"
							v-for="(list, index) in vocalList"
						>
							<img :alt="list.name" :src="list.headimg" id="vocalHeadImg" />
							<div class="vocalMsg">
								<div id="vocalName">{{ list.name }}</div>
								<div id="vocalPresentation">{{ list.presentation }}</div>
							</div>
						</div>
					</div>
				</div>
				<!--右侧-->
				<div class="ttsPage-right">
					<!--合成内容-->
					<div class="textInput"><textarea @input="updateTtsText" id="ttsText" maxlength="60" placeholder="请输入要合成的文本"></textarea></div>
					<!--合成参数-->
					<div class="ttsConfig">
						<div class="ttsConfig-left">
							<div class="setting cantChoose">
								<span class="parameterName">语调:</span>
								<input @input="clickVocaPit" class="parameterInput" max="100" min="-100" name="pit" step="10" type="range" value="0" />
							</div>
							<div class="setting">
								<span class="parameterName">语速:</span>
								<input @input="clickVocaVel" class="parameterInput" max="100" min="-100" name="vel" step="10" type="range" value="0" />
							</div>
							<div class="setting">
								<span class="parameterName">音量:</span>
								<input @input="clickVocaVol" class="parameterInput" max="100" min="-100" name="vol" step="10" type="range" value="0" />
							</div>
							<div class="setting">
								<span class="parameterName">仅支持中文。敬请期待...</span>
								<!-- <input class="parameterInput" type="range" name="more" step="0" min="0" max="100"   /> -->
							</div>
							<div style="clear: both;"></div>
						</div>
						<div class="ttsConfig-right">
							<div class="ttsPlay">
								<div @click="playTts()" id="ttsPlay">{{ audioStatus == 0 ? '合成中' : audioStatus == 1 ? '播放' : '停止' }}</div>
								<div @click="downLoadTTSFile()" id="ttsWaveDownload">
									<svg class="icon" height="25" p-id="1976" t="1575004811319" version="1.1" viewBox="0 0 1024 1024" width="50" xmlns="http://www.w3.org/2000/svg">
										<path
											d="M832 768v64H192v-64H128v128h768v-128zM822.624 438.624l-45.248-45.248L544 626.752V128h-64v498.752l-233.376-233.376-45.248 45.248L512 749.248z"
											fill="#ffffff"
											p-id="1977"
										/>
									</svg>
								</div>
							</div>
							<div class="ttsDownloadPage"><a href="http://www.lastdream.net/forum.php?mod=viewthread&amp;tid=1081" target="_Blank">点击下载PC版</a></div>
						</div>
					</div>
					<!--播放器-->
					<div class="audio">
						<audio @pause="onPause" @play="onPlay" ref="audio">
							<source controls preload="none" src="#" type="audio/wav" />
							您的浏览器不支持音频播放。
						</audio>
					</div>
				</div>
			</div>
		</div>
		<!--页面底部信息-->
		<div class="pageEnd">
			<p>声明：本站内容为个人兴趣自制且仅供试听，严禁用于商业用途</p>
			<p>
				<span>
					制作与维护：
					<a href="https://5ixf.cc" target="_Blank">疯子</a>
				</span>
				<span class="white-space"></span>
				<span>
					B站：
					<a href="https://space.bilibili.com/13938564" target="_Blank">0疯_子0</a>
				</span>
				<span class="white-space"></span>
				<span>
					前端开源地址：
					<a href="https://github.com/0fengzi0/vocaltts-html" target="_Blank">Github</a>
					<span class="white-space"></span>
					<a href="https://gitee.com/feng__zi/vocaltts-html" target="_Blank">Gitee</a>
				</span>
			</p>
			<p>
				友情合作：
				<a href="https://space.bilibili.com/42973598" target="_Blank">VirtualVocal粉丝大会应援团</a>
				<span class="white-space"></span>
				<a href="http://www.lastdream.net/" target="_Blank">语音时代</a>
				<span class="white-space"></span>
				<a href="https://space.bilibili.com/134558" target="_Blank">世界协奏之歌</a>
				<span class="white-space"></span>
				<a href="https://space.bilibili.com/24921468" target="_Blank">呆呆酱机器人</a>
				<span class="white-space"></span>
			</p>
			<p>本站建立时间{{ startDay }}，已提供服务{{ runDays }}天，当前版本V{{ version }}</p>
			<p v-html="record"><!--<img src="../assets/beiantubiao.png" alt="" width="13px" height="13px" />--></p>
		</div>
		<PcNotice></PcNotice>
	</div>
</template>

<script>
import config from '../../../package.json';
import Bus from '../Utils/Bus';
import HttpClient from '../Utils/HttpClient';

const PcNotice = () => import('./PcNotice');

export default {
	name: 'PcIndex',
	// 引用组件
	components: {
		PcNotice
	},
	data() {
		return {
			// 建站日期
			startDay: '2018-11-21',
			// 当前版本
			version: config.version,
			// 运行天数
			runDays: 0,
			// 发音人列表
			vocalList: [
				{
					id: '1',
					code: 'lty',
					name: '洛天依',
					version: 't2',
					headimg: 'https://s2.ax1x.com/2019/11/16/M0mbDS.png',
					presentation: '该音源为T2音源',
					status: 'true'
				}
			],
			// tts状态 0合成1播放2停止
			audioStatus: 1,
			// 合成后的音频文件名
			wavePath: '',
			// 输入的TTS文本
			inputTtsText: '',
			// 历史TTS文本
			oldTtsText: '',
			// 选中的发音人
			chickId: -1,
			// 合成参数
			ttsParameter: {
				pit: 0,
				vel: 0,
				vol: 0
			},
			// 备案信息
			record: process.env.VUE_APP_RECORD
		};
	},

	// 初始化完成
	mounted() {
		let that = this;
		// 计算运行天数
		that.countRunDays();
		// 获取声库列表
		that.getVocaList();
		// 监听合成事件
		Bus.$on('returnToken', token => {
			that.doSynth(token);
		});
	},

	// 其他函数
	methods: {
		// 计算运行天数
		countRunDays() {
			// 两个时间戳相差的毫秒数
			var usedTime = Date.parse(new Date()) - Date.parse(new Date(this.startDay));
			// 计算相差的天数
			this.runDays = Math.floor(usedTime / (24 * 3600 * 1000));
		},

		// 播放
		startWave() {
			this.$refs.audio.src = this.wavePath;
			this.$refs.audio.play();
		},

		// 停止播放
		PauseWave() {
			this.$refs.audio.pause();
		},

		// 当播放等待合成
		onWait() {
			this.audioStatus = 0;
		},

		// 当播放停止
		onPause() {
			this.audioStatus = 1;
		},

		// 当播放开始
		onPlay() {
			this.audioStatus = 2;
		},

		// 设置pit参数
		clickVocaPit(e) {
			this.ttsParameter.pit = e.srcElement.value;
			this.oldTtsText = '';
		},

		// 设置vel参数
		clickVocaVel(e) {
			this.ttsParameter.vel = e.srcElement.value;
			this.oldTtsText = '';
		},

		// 设置vol参数
		clickVocaVol(e) {
			this.ttsParameter.vol = e.srcElement.value;
			this.oldTtsText = '';
		},

		// 切换发音人
		chickVoice(index) {
			if (this.vocalList[index].status == 'true') {
				this.chickId = index;
				// 清除历史信息
				this.oldTtsText = '';
			}
		},

		// 下载合成的音频
		downLoadTTSFile() {
			if (this.wavePath != '' && this.wavePath != null) {
				window.open(this.wavePath);
			} else {
				Bus.$emit('showSnackBar', {
					msg: '还没有合成音频哦',
					color: 'warning'
				});
			}
		},

		// 点击播放按钮
		playTts: function() {
			let that = this;
			if (that.inputTtsText != '') {
				if (that.audioStatus == 1) {
					if (that.oldTtsText != that.inputTtsText) {
						// 显示验证码
						Bus.$emit('doVaptcha', {});
					} else {
						// 播放tts
						that.startWave();
					}
				} else if (that.audioStatus == 2) {
					// 停止合成
					that.PauseWave();
				}
			} else {
				// 提示合成文本为空
				Bus.$emit('showSnackBar', {
					msg: '合成文本为空,请重新输入',
					color: 'warning'
				});
			}
		},

		// 更新输入的合成文本
		updateTtsText(e) {
			this.inputTtsText = e.srcElement.value;
			this.oldTtsText = '';
		},

		// 提交合成信息
		doSynth(token) {
			let that = this;
			// 显示合成状态
			that.onWait();
			HttpClient.doHttp('vocaltts/synth/dosynth', 'post', {
				version: that.vocalList[that.chickId]['version'],
				voice: that.vocalList[that.chickId]['code'],
				text: that.inputTtsText,
				vel: that.ttsParameter.vel,
				vol: that.ttsParameter.vol,
				pit: that.ttsParameter.pit,
				token: token
			})
				.then(function(res) {
					if (res.code === 200) {
						that.wavePath = HttpClient.serviceApi + 'vocaltts/synth/getwave/wave/' + res.fileName;
						that.oldTtsText = that.inputTtsText;
						// 播放声音
						that.startWave();
					}
				})
				.catch(function(res) {
					that.onPause();
				});
		},

		// 获取发音人列表
		getVocaList() {
			let that = this;
			HttpClient.doHttp('vocaltts/voca/getvocalist', 'post').then(res => {
				if (res.code == 200) {
					that.vocalList = res.data;
					// 检测并选中第一个可用音源
					that.chooseCanUseVoice();
				}
			});
		},

		// 首次加载自动选择发音人
		chooseCanUseVoice() {
			for (let i = 0; i < this.vocalList.length; i++) {
				if (this.vocalList[i].status == 'true') {
					this.chickId = i;
					break;
				}
			}
		}
	}
};
</script>

<style lang="scss" scoped>
a {
	text-decoration: none;
	color: black !important;
}

a:hover {
	color: #ee0000 !important;
}

p {
	margin: 10px 0 !important;
}

#PcIndex {
	width: 100%;
	height: 100%;
	background: url('https://s1.ax1x.com/2020/07/12/U851Ig.jpg') no-repeat center;
	background-size: 100% 100%;
	position: fixed;
	background-size: cover;
	-webkit-background-size: cover;
	-o-background-size: cover;
}

.glass {
	width: 800px;
	height: 400px;
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	background: inherit;
	overflow: hidden;
	margin: 1% auto auto auto;
	border-radius: 15px;
	padding: 8px 10px;
	box-shadow: 15px 15px 7.5px rgba(0, 0, 0, 0.6);
}

.glass::before {
	content: '';
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	background: inherit;
	background-attachment: fixed;
	filter: blur(10px);
}

.glass::after {
	content: '';
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	background: rgba(0, 0, 0, 0.25);
}

.ttsPage {
	color: white;
	width: 100%;
	height: 100%;
	display: flex;
}

.pageEnd {
	width: 800px;
	position: absolute;
	top: 420px;
	right: 0;
	bottom: 0;
	left: 0;
	margin: auto;
	text-align: center;
}

.ttsPage-left {
	flex: 0 0 200px;
	height: 100%;
	overflow: hidden;
	z-index: 1;
}

.vocalList {
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
}

.vocal {
	border: 1px #bfbfbf none;
	border-radius: 5px;
	margin: 5px;
	display: flex;
}

.canChoose {
	cursor: Pointer;
	border: 1px #bfbfbf solid;
}

.canChoose:hover {
	transform: scale(1.05);
	box-shadow: 4px 4px 2px rgba(0, 0, 0, 0.4);
}

.chickId {
	transform: scale(1.05);
	cursor: Pointer;
	border: 1px #bfbfbf solid;
	box-shadow: 4px 4px 2px rgba(0, 0, 0, 0.4);
}

#vocalHeadImg {
	width: 50px;
	height: 50px;
	margin: 0 5px 0 5px;
	padding: 0;
}

.vocalMsg {
	flex: 1;
	margin-right: 5px;
}

#vocalName {
	font-size: 20px;
	font-weight: 900;
}

#vocalPresentation {
	font-size: 15px;
	color: #abb2bf;
}

.ttsPage-right {
	flex: 1;
	display: flex;
	flex-direction: column;
	height: 100%;
	z-index: 1;
	margin-left: 8px;
	border-left: solid 1px #bfbfbf;
}

.textInput {
	padding: 5px 5px 10px 5px;
	flex: 0 0 200px;
	width: 100%;
	border-bottom: solid 1px #bfbfbf;
}

#ttsText {
	width: 100%;
	height: 100%;
	margin: 0 0 0 5px;
	padding: 10px;
	resize: none;
	font-size: 23px;
	border: 0;
	outline: none;
	background-color: rgba(255, 255, 255, 0.5);
	border-radius: 5px;
	color: white;
}

.cantChoose {
	filter: blur(5px);
}

.cantChoose:hover {
	cursor: no-drop;
}

.ttsConfig {
	flex: 1;
	display: flex;
}

.ttsConfig-left {
	flex: 0 0 250px;
}

.setting {
	float: left;
	width: 230px;
	height: 45px;
	line-height: 50px;
	padding-left: 10px;
}

.parameterName {
	margin-right: 15px;
}

.parameterInput {
	margin: 16px 0 0 0;
	float: right;
	width: 160px;
}

.ttsConfig-right {
	flex: 1;
	display: flex;
	flex-direction: column;
	margin: 50px 10px;
	text-align: center;
}

.ttsPlay {
	flex: 0 0 50px;
	margin: 0 auto;
	font-size: 30px;
	line-height: 50px;
}

.ttsPlay:hover {
	cursor: Pointer;
}

#ttsPlay {
	border: 1px solid #bfbfbf;
	border-radius: 10px 0 0 10px;
	width: 150px;
	height: 50px;
	float: left;
}

#ttsPlay:hover {
	background-color: rgba(255, 255, 255, 0.5);
}

#ttsWaveDownload {
	border: 1px solid #bfbfbf;
	border-radius: 0 10px 10px 0;
	width: 50px;
	height: 50px;
	float: left;
}

#ttsWaveDownload:hover {
	background-color: rgba(255, 255, 255, 0.5);
}

.ttsDownloadPage {
	margin: 20px auto 0;
}

.white-space {
	margin: 0 5px;
}
</style>
