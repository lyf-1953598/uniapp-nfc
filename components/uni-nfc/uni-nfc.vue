<template>
	<view>
		<uni-popup ref="popup" @change="change">
			<view class="popup">
				<view class="">请刷卡读取数据</view>
				<image src="@/static/status.png" mode=""></image>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	var NfcAdapter;
	import uniPopup from "../uni-popup/uni-popup.vue"
	export default {
		components: {
			uniPopup
		},
		data() {
			return {
				formData: {
					bind_code: '',
					tagid: '',
					time: null,
					show: false,

				}
			}
		},
		methods: {
			open() {
				//开启阴影遮罩
				this.$refs.popup.open("top");
				//初始化ncf 并开启监听
				this.NFCInit();
			},
			change(e, s) {
				if (!e.show) {
					//销毁监听事件
					plus.globalEvent.removeEventListener("newintent", false);
				}
			},
			// nfc入口
			NFCInit() {
				try {
					var main = plus.android.runtimeMainActivity();
					//console.log(main);
					var Intent = plus.android.importClass('android.content.Intent');
					// console.log(Intent);
					var Activity = plus.android.importClass('android.app.Activity');
					//console.log(Activity);
					var PendingIntent = plus.android.importClass('android.app.PendingIntent');
					// console.log(PendingIntent);
					var IntentFilter = plus.android.importClass('android.content.IntentFilter');

					NfcAdapter = plus.android.importClass('android.nfc.NfcAdapter');

					var _nfcAdapter = NfcAdapter.getDefaultAdapter(main);


					var ndef = new IntentFilter('android.nfc.action.NDEF_DISCOVERED'); //NfcAdapter.ACTION_NDEF_DISCOVERED
					// console.log(ndef);
					var tag = new IntentFilter('android.nfc.action.TAG_DISCOVERED'); //NfcAdapter.ACTION_TECH_DISCOVERED
					// console.log(tag);
					var tech = new IntentFilter('android.nfc.action.TECH_DISCOVERED');
					// console.log(tech);
					var intentFiltersArray = [ndef, tag, tech];

					var techListsArray = [
						['android.nfc.tech.Ndef'],
						['android.nfc.tech.IsoDep'],
						['android.nfc.tech.NfcA'],
						['android.nfc.tech.NfcB'],
						['android.nfc.tech.NfcF'],
						['android.nfc.tech.Nfcf'],
						['android.nfc.tech.NfcV'],
						['android.nfc.tech.NdefFormatable'],
						['android.nfc.tech.MifareClassi'],
						['android.nfc.tech.MifareUltralight']
					];
					var _intent = new Intent(main, main.getClass());
					// console.log(_intent);
					_intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);

					var pendingIntent = PendingIntent.getActivity(main, 0, _intent, 0);
					var that = this;
					//监听贴卡到手机事件 
					plus.globalEvent.addEventListener('newintent', e => {
						//获取ic卡数据
						that.NFCReadUID()
						//销毁监听事件
						plus.globalEvent.removeEventListener("newintent", false);
					});
					if (_nfcAdapter == null) {} else if (_nfcAdapter.isEnabled() == false) {} else {
						_nfcAdapter.enableForegroundDispatch(main, pendingIntent, IntentFilter, techListsArray);
					}
				} catch (e) {
					//TODO handle the exception
				}
			},
			//获取ic卡数据
			NFCReadUID() {
				var main = plus.android.runtimeMainActivity();
				var _intent = main.getIntent();
				var _action = _intent.getAction();
				// console.log("action type:" + _action);

				if (NfcAdapter.ACTION_NDEF_DISCOVERED == _action || NfcAdapter.ACTION_TAG_DISCOVERED == _action ||
					NfcAdapter.ACTION_TECH_DISCOVERED == _action) {
					var Tag = plus.android.importClass('android.nfc.Tag');
					var bind_codes;
					var tagFromIntent = _intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
					var bind_code = _intent.getByteArrayExtra(NfcAdapter.EXTRA_ID);
					// ic卡号 进制转换
					bind_codes = this.Bytes2HexString(bind_code);
					this.formData.bind_code = bind_codes
					//反转两位
					var tagid = this.reverseTwo(this.bytesToHexString(tagFromIntent.getId()));
					// console.log(this.bytesToHexString(tagFromIntent.getId()));
					// console.log(tagid);
					let icId = parseInt(tagid, 16) + '';
					
					//当ic卡号不到10位的时候 前面自动添加0  补齐10位
					icId = icId.length < 10 ? (Array(10).join(0) + icId).slice(-10) : icId;
					if (icId) {
						this.$emit("changeNfc", {
							tagid: tagid
						})
					} else {
						uni.showToast({
							title: "读取数据失败,请重新读取",
							icon: 'none'
						})
					}
					//关闭遮罩
					this.$refs.popup.close();
				}
			},
			reverseTwo(str) {

				var str1 = "";
				for (var i = 1; i <= str.length; i++) {
					str1 += str[i - 1];
					if (i % 2 == 0) {
						if (i == str.length) {
							break;
						}
						str1 += ":";
					}
				}
				var str2 = "";
				for (var i = str1.split(":").length - 1; i >= 0; i--) {
					str2 += str1.split(":")[i];
				}
				return str2;
			},
			bytesToHexString(inarray) {
				var i, j, x;
				var hex = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A",
					"B", "C", "D", "E", "F"
				];
				var out = "";
				for (j = 0; j < inarray.length; ++j) {
					x = parseInt(inarray[j]) & 0xff;
					i = (x >> 4) & 0x0f;
					out += hex[i];
					i = x & 0x0f;
					out += hex[i];
				}
				return out;
			},

			//将byte[] 转为Hex，
			Bytes2HexString(arrBytes) {
				var str = '';
				for (var i = 0; i < arrBytes.length; i++) {
					var tmp;
					var num = arrBytes[i];
					if (num < 0) {
						//Java中数值是以补码的形式存在的，应用程序展示的十进制是补码对应真值。补码的存在主要为了简化计算机底层的运算，将减法运算直接当加法来做
						tmp = (255 + num + 1).toString(16);
					} else {
						tmp = num.toString(16);
					}
					if (tmp.length == 1) {
						tmp = '0' + tmp;
					}
					str += tmp;
				}
				return str;
			}
		}
	}
</script>

<style scoped lang="scss">
	.popup {
		width: 100%;

		view {
			font-size: 45upx;
			text-align: center;
			color: #fff;
			margin-bottom: 60upx;
		}

		image {
			display: block;
			margin: 0 auto;
		}
	}
</style>
