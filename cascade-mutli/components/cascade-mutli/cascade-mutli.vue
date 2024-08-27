<template>
  <view>
    <view class="input-block" @click="inputClick">
      <input
        :value="_label"
        :style="`text-align:${inputAlign};${disabled ? 'color:#c0c4cc;' : ''}`"
        :placeholder="placeholder"
        placeholder-style="color: #c0c4cc;font-size: 32rpx;font-weight: none"
        disabled
      />
      <image class="w-[38rpx] h-[38rpx] block mt-[2rpx]" src="/static/imgs/mine/arrow.png" />
    </view>
    <!-- 弹出层 -->
    <view :class="['next-scroll-popup', isShow ? 'next-scroll-open' : '', animation ? 'next-scroll-animation' : '']">
      <view class="next-scroll-box">
        <view class="next-scroll-top flex justify-between items-center px-[38rpx]">
          <view />
          <!-- <view class="text-36 text-[#333] font-bold">请选择</view> -->
          <image class="w-[32rpx] h-[32rpx] block" src="/static/imgs/close.png" @click="getResult('cancel')" />
        </view>
        <scroll-view class="next-scroll-title ml-[24rpx]" scroll-x="true" :scroll-left="scrollViewWidth" scroll-with-animation>
          <view class="next-scroll-title-item-box mr-[12rpx]" v-for="(i, e) in tabList" @click="checkTab(e)" :key="e">
            <template v-if="tabId == e && isMulti && checkList[e]">
              <view class="flex items-center" :id="'next-' + e" :class="['next-scroll-title-item', ' next-scroll-title-item-true']">
                {{ tabMultiLabel }}
              </view>
            </template>
            <template v-else>
              <view class="flex items-center" v-if="tabId >= e" :id="'next-' + e" :class="['next-scroll-title-item', tabId == e ? ' next-scroll-title-item-true' : '']">
                {{ checkList[e] ? checkList[e][labelKey] : i.title }}
                <image v-if="tabId > e" class="w-[28rpx] h-[28rpx] block" src="/static/imgs/mine/arrow.png" />
              </view>
            </template>
          </view>
        </scroll-view>
        <scroll-view class="next-scroll-view_H" scroll-y="true">
          <view class="next-scroll-view-grid-box text-center" v-if="checkBox && checkBox.length && checkBox[tabId] && checkBox[tabId].length">
            <template v-for="(item, index) in checkBox[tabId]" :key="index">
              <template v-if="isMultiLast">
                <view @click="check(index)" :class="isSelected(item) ? 'next-scroll-view-item-true' : 'next-scroll-view-item'">
                  {{ item[labelKey] || '' }}
                </view>
              </template>
              <template v-else>
                <view @click="check(index)" :class="checkList && checkList[tabId] && checkList[tabId][labelKey] == item[labelKey] ? 'next-scroll-view-item-true' : 'next-scroll-view-item'">
                  {{ item[labelKey] || '' }}
                </view>
              </template>
            </template>
          </view>
          <view class="next-scroll-view-noBox" v-else>
            <view class="text">暂无数据</view>
          </view>
        </scroll-view>
        <view
          class="w-[580rpx] h-[88rpx] m-[27rpx_auto_30rpx] bg-[#0068FF] rounded-full text-[#fff] text-36 font-bold flex justify-center items-center"
          v-if="checkList.length == tabList.length"
          @click="getResult('confirm')"
        >
          确定
        </view>
        <view v-show="safeArea" class="next-scroll-temp" />
      </view>
    </view>
    <!-- 遮罩层 -->
    <view v-show="isShow" class="next-scroll-mask" @click.stop="isMask ? close() : ''" />
  </view>
</template>

<script lang="ts" setup>
import { ref, watch, computed, nextTick, getCurrentInstance } from 'vue';

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [],
  },
  /** 最后是否多选 */
  isMulti: {
    type: Boolean,
    default: false,
  },
  isMask: {
    type: Boolean,
    default: true,
  },
  animation: {
    type: Boolean,
    default: true,
  },
  safeArea: {
    type: Boolean,
    default: true,
  },
  options: {
    type: Array,
    default: () => [],
  },
  valueKey: {
    type: String,
    default: 'value',
  },
  labelKey: {
    type: String,
    default: 'label',
  },
  childrenKey: {
    type: String,
    default: 'children',
  },
  showArrow: {
    type: Boolean,
    default: true,
  },
  placeholder: {
    type: String,
    default: '请选择',
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  inputAlign: {
    type: String,
    default: 'right',
  },
  customRenderLabelFunc: {
    type: Function,
  },
});

const emit = defineEmits(['close', 'update:modelValue', 'confirm']);

const instance = getCurrentInstance();

const isShow = ref(false);
const checkBox = ref<Array<any>>([]);
const tabId = ref(0);
const checkList = ref<Array<any>>([]);
const checkListModel = ref<Array<any>>([]);
const id = ref(0);
const tabList = ref([{ title: '请选择', id: 0 }]);
const scrollViewWidth = ref(0);
const elWidth = ref(0);
const currentMutliAnswer = ref<any[]>([]);

const _label = ref();
const _value = ref();

const isMultiLast = computed(() => props?.isMulti && id.value == tabList.value.length - 1 && tabId.value !== 0);

const tabMultiLabel = computed(() => {
  const tempArr = _label.value?.split('-');
  return tempArr?.[tempArr?.length - 1];
});

const setCheckData = () => {
  const arr: any[] = [];
  const boxArr: any[] = [];
  (props.modelValue || []).forEach((val: any, index) => {
    if (index === 0) {
      arr[0] = props.options.find((item: any) => item[props.valueKey] === val) || {};
      boxArr[0] = props.options;
    } else {
      const children = arr[index - 1][props.childrenKey] || [];
      if (props?.isMulti && index == props?.modelValue?.length - 1 && index !== 0) {
        const tempArr = val?.map((citem: any) => {
          const finItem = children.find((item: any) => item[props.valueKey] === citem) || {};
          return finItem;
        });
        arr[index] = tempArr;
        currentMutliAnswer.value = tempArr;
      } else {
        arr[index] = children.find((item: any) => item[props.valueKey] === val) || {};
      }

      boxArr[index] = children;
    }
  });
  const lastLen = arr.length - 1;
  checkListModel.value = arr;

  tabList.value = arr?.map(item => ({
    id: item?.[props?.valueKey],
    title: '请选择',
  }));
  id.value = lastLen;
  tabId.value = lastLen;
  checkBox.value = boxArr;
  checkList.value = arr;

  uni
    .createSelectorQuery()
    .in(instance)
    .select('.next-scroll-title')
    .boundingClientRect((rect: any) => {
      scrollViewWidth.value = Math.round(rect.width);
    })
    .exec();
};

const getData = async () => {
  if (checkList.value.length === tabList.value.length) return;
  uni.showLoading({ title: '加载中...' });
  const list: any[] = [];
  if (checkList.value.length) {
    const tempid = checkList.value[id.value - 1][props.valueKey];
    const item = checkBox.value[id.value - 1].find((item: any) => item[props.valueKey] === tempid);
    (item[props.childrenKey] || []).forEach((e: any) => list.push(e));
  } else {
    props.options.forEach(e => list.push(e));
  }
  checkBox.value[id.value] = list;

  uni.hideLoading();
};

const init = () => {
  id.value = 0;
  tabId.value = 0;
  checkBox.value = [];
  checkList.value = [];
  uni
    .createSelectorQuery()
    .in(instance)
    .select('.next-scroll-title')
    .boundingClientRect((rect: any) => {
      scrollViewWidth.value = Math.round(rect.width);
    })
    .exec();
  getData();
};

watch(
  () => [props.modelValue, props?.options],
  val => {
    if (props.modelValue?.length > 0) {
      setCheckData();
    } else {
      init();
    }
  },
  {
    immediate: true,
  },
);

watch(
  () => checkList.value,
  () => {
    const labelStr = (checkList.value || [])
      .map((item, index) => {
        if (props?.isMulti && index == checkList.value.length - 1 && index !== 0) {
          return (Array.isArray(item) ? item : [])?.map((citem: any) => citem?.[props.labelKey])?.join(',');
        } else {
          return item[props.labelKey];
        }
      })
      .join('-');
    _label.value = props.customRenderLabelFunc && typeof props.customRenderLabelFunc === 'function' ? props.customRenderLabelFunc(labelStr) : labelStr;

    _value.value = (checkList.value || []).map((item, index) => {
      if (props?.isMulti && index == checkList.value.length - 1 && index !== 0) {
        return (Array.isArray(item) ? item : [])?.map((citem: any) => citem?.[props.valueKey]);
      } else {
        return item[props.valueKey];
      }
    });
  },
  {
    immediate: true,
    deep: true,
  },
);

const inputClick = () => {
  if (!props.disabled) {
    open();
  }
};

const open = () => {
  isShow.value = true;
};

const close = () => {
  if (props.modelValue?.length > 0) {
    setCheckData();
  } else {
    init();
  }
  isShow.value = false;
  emit('close');
};

const isSelected = (option: any) => {
  return !!currentMutliAnswer.value.find((item: any) => item?.[props.valueKey] == option?.[props.valueKey]);
};
const toggleSelection = (item: any) => {
  let tempAnswer = currentMutliAnswer.value;
  if (!tempAnswer?.[0]?.[props.valueKey]) {
    tempAnswer = [];
  }
  const index = tempAnswer.findIndex(citem => citem?.[props.valueKey] == item?.[props.valueKey]);
  if (index === -1) {
    tempAnswer.push(item);
    currentMutliAnswer.value = tempAnswer;
  } else {
    tempAnswer.splice(index, 1);
    currentMutliAnswer.value = tempAnswer;
  }
};

const check = async (index: number) => {
  const children = checkBox.value[id.value][index][props.childrenKey];
  let n = 0;
  if (children && children.length) {
    n = id.value + 1;
  } else {
    n = id.value;
  }
  tabList.value = Array.from({ length: n + 1 }, (_, i) => ({
    title: '请选择',
    id: i,
  }));
  if (isMultiLast.value) {
    toggleSelection(checkBox.value[id.value][index]);

    checkList.value[id.value] = currentMutliAnswer.value;
  } else {
    checkList.value[id.value] = checkBox.value[id.value][index];
  }
  if (id.value < tabList.value.length - 1) id.value++;
  await getData();
  nextTick(() => {
    uni
      .createSelectorQuery()
      .in(instance)
      .select(`#next-${tabId.value}`)
      .boundingClientRect((rect: any) => {
        elWidth.value = Math.round(rect.width);
      })
      .exec();
    setTimeout(() => {
      scrollViewWidth.value += elWidth.value;
    });
    if (tabId.value < tabList.value.length - 1) tabId.value++;
  });
};

const checkTab = (e: any) => {
  if (e === id.value) return;
  if (props?.isMulti) {
    currentMutliAnswer.value = [];
  }
  id.value = e;
  tabId.value = e;
  checkList.value = checkList.value.splice(0, e);
};

const getResult = (event: 'confirm' | 'cancel') => {
  if (event === 'confirm') {
    if (checkList.value.length !== tabList.value.length) return;

    const result = checkList.value;
    checkListModel.value = result;
    emit('update:modelValue', _value.value);
    emit('confirm', { value: result });
  }
  close();
};
</script>

<style>
::v-deep ::-webkit-scrollbar {
  width: 0;
  height: 0;
  color: transparent;
  display: none;
}
</style>
<style lang="scss" scoped>
.input-block {
  display: flex;
  align-items: center;
  justify-content: center;
  input {
    flex: 1;
    color: #333;
    pointer-events: none;
    text-overflow: ellipsis;
  }
}
.next-scroll-popup {
  width: 100%;
  position: fixed;
  left: 0;
  bottom: -100%;
  z-index: 999;
}

.next-scroll-open {
  bottom: 0px !important;
}

.next-scroll-animation {
  transition: all 0.25s linear;
}

.next-scroll-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  z-index: 998;
}

.next-scroll-box {
  position: absolute;
  bottom: 0;
  width: 100%;
  background: #ffff;
  border-radius: 24rpx 24rpx 0 0;
}

.next-scroll-temp {
  padding-bottom: env(safe-area-inset-bottom);
}

.next-scroll-top {
  height: 88rpx;
  line-height: 88rpx;
  border-bottom: 1px solid rgba(151, 151, 151, 0.3);
  margin-bottom: 20rpx;
}

.next-scroll-top-left {
  float: left;
  padding-left: 24rpx;
  font-size: 30rpx;
}

.next-scroll-top-right {
  float: right;
  padding-right: 24rpx;
  color: #3c9cff;
  font-size: 30rpx;
}

.next-scroll-title {
  white-space: nowrap;
  width: 100%;
  height: 60rpx;
  line-height: 60rpx;
  background-color: #fff;
}

.next-scroll-view_H {
  white-space: nowrap;
  width: 100%;
  height: 400rpx;
  line-height: 100rpx;
  background-color: #ffffff;
}

.next-scroll-title-item {
  position: relative;
}

.next-scroll-title-item-box {
  display: inline-block;
  font-size: 28rpx;
  color: #333333;
}

.next-scroll-title-item-true {
  font-size: 28rpx;
  font-weight: 700;
  color: #3c9cff;
}

.next-scroll-view-box {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin: 16rpx;
}

.next-scroll-view-grid-box {
  width: calc(100% - 20rpx);
  margin: 10rpx;
  padding-bottom: 10rpx;
}

.next-scroll-view-noBox {
  width: 100%;
  height: 400rpx;
  padding-top: 20px;
  text-align: center;

  image {
    width: 200rpx;
    height: 200rpx;
    margin-top: 32rpx;
  }

  .text {
    width: 100%;
    text-align: center;
    color: #888;
    font-size: 28rpx;
    margin-top: -40rpx;
  }
}

.next-scroll-view-item {
  padding: 0rpx 24rpx;
  text-align: center;
  border-radius: 6rpx;
  background: #fff;
  color: #666666;
  font-size: 28rpx;
  margin: 12rpx 4rpx;
  height: 66rpx;
  line-height: 66rpx;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.next-scroll-view-item-true {
  padding: 0rpx 24rpx;
  text-align: center;
  border-radius: 6rpx;
  background-color: #f0f4fc;
  color: #1481ff;
  font-size: 28rpx;
  margin: 12rpx 4rpx;
  height: 66rpx;
  line-height: 66rpx;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>
