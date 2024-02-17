<template>
  <view class="content">
    <view class="view-input">
      
      <view>均数每份：{{ textCount }}￥<text @tap="copy(textCount)" class="copy">❏</text></view>
      <view>剩余总价：{{ textCountOther }}￥<text @tap="copy(textCountOther)" class="copy" v-if="index">❏</text></view>
      <text class="view-input-power">
        <view>
          ⚡️ n-1每电费：{{ textPowerOtherCount }}<text @tap="copy(textPowerOtherCount)" class="copy" v-if="index">❏</text>
        </view>

        <view>
          ⚡️ 3/1分出： {{ textPowerCount }}<text @tap="copy(textPowerCount)" class="copy">❏</text>
        </view>
        <view>
          ⚡️ 剩余总价： {{ textPowerCountOther }}<text @tap="copy(textPowerCountOther)" class="copy">❏</text>
        </view>

      </text>
    </view>
    <view class="view-content">
      <view class="select">
        <view class="select-title">
          选择你要分配的总人数
        </view>
        <view>
          <picker @change="changeIndex" :value="index" :range="indexArray">
            <view class="uni-input">{{ indexArray[index as number] }}☚ </view>
          </picker>
        </view>
        <view class="str">
          {{ textStr }}
          <view class="text">{{ text }}</view>
        </view>
        <view class="help">
          <text @click="open" class="help-win">帮助</text>
          <uni-popup ref="helpPopup" type="bottom">
            <view class="help-message">
              <strong>帮助</strong>
              <view>
                <text style="color: #a4a4a4;font-size: 30rpx;"> 本计算器用于计算平均分出的￥，同时计算分出一份3/1的情况</text>
              </view>
              <view>
                1、点击复制图标将结果复制
              </view>
              <view>
                2、长按退格键可以将所有字符删除
              </view>
              <view>
                3、电费计算是根据一人份的结果3/1后再将剩余的n-1份分出，所以至少<strong>两</strong>份以上，如果只有一份，没有计算意义，单纯将数字分开
              </view>
              <view style="color: #fe8181;">
                4、为了方便使用，所有计算的值都经历小数点后两位的四舍五入，可能存在些许误差，0.01左右，不要在意嘿嘿
              </view>
            </view>
          </uni-popup>

        </view>
      </view>
    </view>
    <view class="view-keyboard">
      <view class="view-keyboard-other">
        <view @tap="addcount('.')">.</view>
        <view class="view-keyboard-other-zero" @tap="addcount('0')">0</view>
        <view @tap="del()" @longpress="del(true)">退格</view>
      </view>
      <view v-for="(item, index) in keyboard" class="view-keyboard-item" @tap="addcount(item + '')">
        <view> {{ item }}</view>
      </view>

    </view>
  </view>
</template>

<script setup lang="ts">
import type { InputOnInput, PickerValue, PickerViewOnChange } from '@uni-helper/uni-app-types';
import type { UniPopup, UniPopupType } from '@uni-helper/uni-ui-types';
import { computed } from 'vue';
import { watch, type Ref } from 'vue';
import { ref } from 'vue'

// 记录数据
const textStr = ref("")
const text = ref(0)

// 动态将结果算出
// 每份的钱
const textCount = computed(() => {
  return (text.value / indexArray.value[index.value as number]).toFixed(2)
})
// 剩余的钱
const textCountOther = computed(() => {
  return (text.value - parseFloat(textCount.value)).toFixed(2)
})
// 电费每份的钱，需要一个人3/1后再分配给所有人
const textPowerCount = computed(() => {
  return (parseFloat(textCount.value) / 3).toFixed(2)
})
// 除以完后的结果，需要去除一份
const textPowerOtherCount = computed(() => {
  // 如果除数等于0，表示不需要分离了
  if (!((indexArray.value[index.value as number]) - 1)) {
    return "单份"
  } else {
    return ((text.value - parseFloat(textPowerCount.value)) /
      ((indexArray.value[index.value as number]) - 1)).toFixed(2)
  }
})
// 将除去3/1的数删掉的总价

const textPowerCountOther = computed(() => {
  return (text.value - parseFloat(textPowerCount.value)).toFixed(2)
})


// 判断字符串,符合条件的匹配成功，不删除
watch(textStr, (val, oldval) => {
  /^([^.]*(\.[^.]*){0,1})$/.test(val) &&
    /^.{1,12}$/.test(val) &&
    /^[^\.]+(\.[^\.]{0,2})?$/.test(val) ?
    null
    :
    del()

  changeNum(textStr.value)
})
// // 计算方法
// const calculator:InputOnInput = (event)=>{
// console.log(event);

// }
// picker选择器的方法
const indexArray = ref([1, 2, 3, 4, 5, 6])
// 默认值是最后一位
const index: Ref<PickerValue> = ref(indexArray.value.length - 1)
const changeIndex: PickerViewOnChange = (e) => {
  index.value = e.detail.value

}


// 键盘数组
const keyboard = ref([1, 2, 3, 4, 5, 6, 7, 8, 9])
// 键盘输入数字事件
const addcount = (text: string) => {
  textStr.value += text
}


// 删除事件
const del = (clear?: boolean) => {
  clear ? textStr.value = "" : textStr.value = textStr.value.slice(0, -1)

}


// 计算输入的字符串与人数的分数
const changeNum = (str: string) => {
  if (str !== "") {
    text.value = parseFloat(str)
  } else {
    text.value = 0
  }

}
// 复制内容到粘贴板
const copy = (str: string) => {
  uni.setClipboardData({
    data: str,
  })
}
// 开启帮助弹窗
const helpPopup: Ref<UniPopup | null> = ref(null)
const open = () => {
  helpPopup.value!.open("center" as UniPopupType)
}

</script>

<style>
.view-input {
  display: flex;
  justify-content: center;
  flex-direction: column;
  box-sizing: border-box;
  width: 100vw;
  padding: 1vh 2vw;
  background-color: #f5f5f5;
  font-size: 40rpx;
}

.view-input-power {
  font-size: 35rpx;
  color: rgb(135, 135, 135);
}

.view-content {
  background-color: #ffffff;
}

.select {
  display: flex;
  flex-direction: column;
  align-content: center;
  height: 10vh;
  padding: 0 5vw;
  font-size: 60rpx;
}

.select-title {
  font-size: 40rpx;
}

.str {
  margin: 1vh 0 vw;
}

.text {
  font-size: 45rpx;
  color: #8a8a8a;
}

.view-keyboard {
  position: absolute;
  bottom: 0;
  display: flex;
  flex-wrap: wrap-reverse;
  width: 100%;
  font-size: 55rpx;
  font-weight: 900;
}

.view-keyboard-item {
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 33.33%;
  height: 20vw;
  max-height: 10vh;
  background-color: #f5f5f5;
  border: .5px solid #ffffff;

}

.view-keyboard-other {
  display: flex;
  margin-bottom: 2vh;
  width: 100vw;
  height: 20vw;
  background-color: #f5f5f5;
}

.view-keyboard-other view {
  display: flex;
  align-content: center;
  justify-content: center;
  flex-wrap: wrap;
  flex: 1;
}

.view-keyboard-other .view-keyboard-other-zero {
  box-sizing: border-box;
  flex: 2;
  border: .5px solid #ffffff
}

.help {
  display: flex;
  justify-content: left;
}

.help-win {
  max-width: 30vw;
  font-size: 25rpx;
  font-weight: 300;
  color: #8b8b8b;
  background-color: #fff;

}

.help-message {
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 2vh 2vw;
  font-size: 35rpx;
  color: #4e4e4e;
  background-color: #fcfcfc;
}

.help-message view {
  margin: 1vh 0;
}

.copy {
  padding: 0 .5vw;
  color: #000;
}
</style>
