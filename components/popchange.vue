<template>
  <div class="popup-wrapper" v-if="showPopChange">
    <div class="popup-content">
      <div class="popup-title">您正在修改：{{ title }}</div>
      <div class="input-container">
        <input v-model="name" type="text" :placeholder="title" class="input-field" />
      </div>
      <div class="input-container">
        <input v-model="startTime" :placeholder="starthint" type="date" class="input-field" />
      </div>
      <div class="input-container">
        <input v-model="endTime" :placeholder="endhint" type="date" class="input-field" />
      </div>
      <div class="button-container">
        <button class="confirm-button" @click="confirm">确认</button>
        <button class="cancel-button" @click="cancel">取消</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    showPopChange: {
      type: Boolean,
      default: false,
    },
    title: {
      type: String,
      default: '弹窗主题',
    },
	starthint: {
	  type: String,
	  default: '请输入新开始时间',
	},
	endhint: {
	  type: String,
	  default: '请输入新结束时间',
	},
  },
  data() {
    return {
      name: '',
      startTime: '',
      endTime: '',
    };
  },
  methods: {
    confirm() {

      this.$emit('uploadData', {
        name: this.name,
        startTime: this.startTime,
        endTime: this.endTime,
      });
      this.closePopup();
    },
    cancel() {
      this.closePopup();
    },
    closePopup() {
      // 关闭弹窗
      this.$emit('close');
    },
  },
};
</script>

<style scoped>
.popup-wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
}

.popup-title {
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 15px;
}

.input-container {
  margin-bottom: 10px;
  margin-top: 10px;
  width: 95%;
}

.input-field {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button-container {
  display: flex;
  justify-content: flex-end;
  justify-items: center;
  margin-top: 20px;
 
}

.confirm-button{
  padding: 8px 16px;
  padding-top: 0;
  padding-bottom: 0;
  margin-left: 10px;
  
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  margin-bottom: 10px;
}

.cancel-button {
  padding: 8px 16px;
  padding-top: 0;
  padding-bottom: 0;
  margin-right: 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background-color: #ccc;
  margin-top: 10px;
  margin-bottom: 10px;
}

</style>
