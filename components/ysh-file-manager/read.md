import yshFileManager from "@/components/ysh-file-manager/ysh-file-manager.vue"

<ysh-file-manager ref="filemanager" @result="resultPath"></ysh-file-manager>

components: {
	yshFileManager
}


this.$refs.filemanager._openFile()

