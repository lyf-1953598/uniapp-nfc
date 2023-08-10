<template xlang="wxml" minapp="mpvue">
	<view>
	</view>
</template>
<script>
export default {
	name: 'ysh-file-manager',
	props: {},
	data() {
		return {
		}
	},
	methods: {
		_openFile() {
			this.qxcz()
		},
		qxcz(){
			var _this = this
			plus.android.requestPermissions(['android.permission.READ_EXTERNAL_STORAGE'], e => {
				if (e.deniedAlways.length > 0) { //权限被永久拒绝
					// 弹出提示框解释为何需要读写手机储存权限，引导用户打开设置页面开启
					uni.showToast({
						title: '您拒绝了存储权限，请去设置-应用开启存储权限！',
						icon: 'none',
						duration: 2000
					});
					
				}
				if (e.deniedPresent.length > 0) { //权限被临时拒绝
					// 弹出提示框解释为何需要读写手机储存权限，可再次调用plus.android.requestPermissions申请权限
					plus.android.requestPermissions(['android.permission.READ_EXTERNAL_STORAGE'])
					//console.log('666666666 ' + e.deniedPresent.toString());
				}
				if (e.granted.length > 0) { //权限被允许
					//调用依赖获取读写手机储存权限的代码
					// _this.upload() // 获取权限成功之后调用的函数
					this._openFileTemp()
					//console.log('2222222222 ' + e.granted.toString());
				}
			}, function(e) {
				console.log('R12133313221' + JSON.stringify(e));
			});
		},
		_openFileTemp() {
			// #ifdef APP-PLUS
			let CODE_REQUEST = 1000;
			if (plus.os.name.toLowerCase() != "android") {
				uni.showModal({
					title: '提示',
					content: '仅支持Android平台！',
					success: function (res) {}
				});
				return false
			}
			let that = this
			// java 代码来自 http://www.cnblogs.com/panhouye/archive/2017/04/23/6751710.html
			let main = plus.android.runtimeMainActivity();
			let Intent = plus.android.importClass("android.content.Intent");

			// 
			let fileIntent = new Intent(Intent.ACTION_GET_CONTENT)
			//fileIntent.setType(“image/*”);//选择图片
			//fileIntent.setType(“audio/*”); //选择音频
			//fileIntent.setType(“video/*”); //选择视频 （mp4 3gp 是android支持的视频格式）
			//fileIntent.setType(“video/*;image/*”);//同时选择视频和图片
			fileIntent.setType("*/*"); //无类型限制
			 
			fileIntent.addCategory(Intent.CATEGORY_OPENABLE);
			
			// 获取回调
			main.onActivityResult = function (requestCode, resultCode, data) {
				let Activity = plus.android.importClass("android.app.Activity");
				let ContentUris = plus.android.importClass("android.content.ContentUris");
				let Cursor = plus.android.importClass("android.database.Cursor");
				let Uri = plus.android.importClass("android.net.Uri");
				let Build = plus.android.importClass("android.os.Build");
				let Environment = plus.android.importClass("android.os.Environment");
				let DocumentsContract = plus.android.importClass("android.provider.DocumentsContract");
				let MediaStore = plus.android.importClass("android.provider.MediaStore");
				// 给系统导入 contentResolver
				let contentResolver = main.getContentResolver()
				plus.android.importClass(contentResolver);
				// 返回路径
				let path = '';
				let size = '';
				if (resultCode == Activity.RESULT_OK) {
					let uri = data.getData()
					
					if ("file" == uri.getScheme().toLowerCase()) { //使用第三方应用打开
						path = uri.getPath();
					} else {
						if (Build.VERSION.SDK_INT > Build.VERSION_CODES.KITKAT) { //4.4以后
							path = getPath(this, uri);
							
							//读文件大小
						  let FileInputStream = plus.android.importClass("java.io.FileInputStream");  
						  let fileSize = new FileInputStream(path);  
						   size = fileSize.available();  
						  // 单位转换  
						  let fileSizeString;  
						  if(size == 0){  
							  fileSizeString = "0B";  
						  }else if(size < 1024){  
							  fileSizeString = size + "B";  
						  }else if(size < 1048576){  
							  fileSizeString = (size/1024).toFixed(2) + "KB";  
						  }else if (size < 1073741824){  
							  fileSizeString = (size/1048576).toFixed(2) + "MB";  
						  }else{  
							  fileSizeString = (size/1073741824).toFixed(2) + "GB";  
						  }
						} else { //4.4以下下系统调用方法
							path = getRealPathFromURI(uri)
						}
					}
					
					// 回调
					that.$emit('result', {
						path: path,
						size: size
					})
				}
				// 4.4 以上 从Uri 获取文件绝对路径
				function getPath(context, uri) {
					let isKitKat = Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT;
					let scheme = uri.getScheme().toLowerCase()
					if (isKitKat && DocumentsContract.isDocumentUri(context, uri)) {
						// ExternalStorageProvider
						if (isExternalStorageDocument(uri)) {
							let docId = DocumentsContract.getDocumentId(uri);
							let split = docId.split(":");
							let type = split[0];
							// 如果是手机内部存储
							if ("primary" == type.toLowerCase()) {
								return Environment.getExternalStorageDirectory() + "/" + split[1];
							} else {
								return '/storage/' + type + "/" + split[1];
							}
						}
						// DownloadsProvider
						else if (isDownloadsDocument(uri)) {
							//console.log(2)
							let id = DocumentsContract.getDocumentId(uri);
							
							if (id.indexOf(':') != -1) {
								let split = id.split(":");
								return split[1]
							} else {
								let contentUri = ContentUris.withAppendedId(Uri.parse("content://downloads/public_downloads"), id);
								return getDataColumn(context, contentUri, null, null);
							}
						}
						// MediaProvider
						else if (isMediaDocument(uri)) {
							let docId = DocumentsContract.getDocumentId(uri);
							let split = docId.split(":");
							let type = split[0];
							
							let contentUri = null;
							if ("image" == type.toLowerCase()) {
								contentUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
							} else if ("video" == type.toLowerCase()) {
								contentUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
							} else if ("audio" == type.toLowerCase()) {
								contentUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
							} else if ("document" == type.toLowerCase()) {
								contentUri = MediaStore.Files.getContentUri("external");
							}
							let selection = "_id=?";
							let selectionArgs = [split[1]];
							return getDataColumn(context, contentUri, selection, selectionArgs);
						}
					}
					// MediaStore (and general)
					else if ("content" == scheme) {
						return getDataColumn(context, uri, null, null);
					}
					// File
					else if ("file" == scheme) {
						return uri.getPath();
					}
				}
				// 4.4 以下 获取 绝对路径
				function getRealPathFromURI(uri) {
					let res = null
					let proj = [MediaStore.Images.Media.DATA]
					let cursor = contentResolver.query(uri, proj, null, null, null);
					if (null != cursor && cursor.moveToFirst()) {						;
						let column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
						res = cursor.getString(column_index);
						cursor.close();
					}
					return res;
				}
				// 通过uri 查找出绝对路径
				function getDataColumn(context, uri, selection, selectionArgs) {
					//console.log(context)
					let cursor = null;
					let column = "_data";
					let projection = [column];
					cursor = contentResolver.query(uri, projection, selection, selectionArgs, null);
					if (cursor != null && cursor.moveToFirst()) {
						let column_index = cursor.getColumnIndexOrThrow(column);
						return cursor.getString(column_index);
					} else {
						return '';
					}
				}
				function isExternalStorageDocument(uri) {
					return "com.android.externalstorage.documents" == uri.getAuthority() ? true : false
				}
				function isDownloadsDocument(uri) {
					return "com.android.providers.downloads.documents" == uri.getAuthority() ? true : false
				}
				function isMediaDocument(uri) {
					return "com.android.providers.media.documents" == uri.getAuthority() ? true : false
				}
			}
			main.startActivityForResult(fileIntent, CODE_REQUEST);
			// #endif
			// #ifndef APP-PLUS
			uni.showModal({
				title: '提示',
				content: '仅支持Android平台',
				success: function (res) {

				}
			});
			// #endif
		},
	}
}
</script>