<template>
  <view class="relative" :style="[boxBaseSyle()]">
    <image v-for="(avatar, index) in list" :key="avatar.id" :src="avatar.url" class="absolute br-50_" :style="[itemBaseSyle(), itemSyle(index, avatar)]" />
  </view>
</template>

<script setup lang="ts">
import { computed, watch, nextTick } from 'vue';
import { ref } from 'vue';

type UserSwiperProps = {
  /** 轮播间隔时间 */
  interval?: number;
  urls?: any[];
  width?: number;
  height?: number;
  /** 重叠的部分 */
  overlap?: number;
  prop?: {
    /** urls数组中，图片url对应属性的key */
    url?: string;
  };
  /** 头像组展示数量 */
  showNumber?: number;
};
const props = withDefaults(defineProps<UserSwiperProps>(), {
  interval: 3000,
  urls: () => [],
  width: 50,
  height: 50,
  overlap: 20,
  prop: () => ({
    url: 'url',
  }),
  showNumber: 3,
});

const list = ref<any[]>([]);
const intervalTimer = ref<NodeJS.Timeout>();

const curLastIndex = ref(props?.showNumber);

const marginLeft = computed(() => props?.width - props?.overlap);

const stop = () => {
  clearInterval(intervalTimer.value);
};

const start = () => {
  intervalTimer.value = setInterval(() => {
    forwardMove();
  }, props?.interval); // 可以根据需要调整间隔时间
};

watch(
  () => props?.urls,
  newVal => {
    // TODO 新数据到来，一个一个的替换，
    list.value = newVal.slice(0, props?.showNumber).map((item, index) => {
      let url = item && item[props?.prop.url || ''];
      if (typeof item === 'string') {
        url = item;
      }

      return { url, order: index + 1, id: `avatar-${index}` };
    });
    if (newVal.length > props?.showNumber) {
      stop();
      start();
    } else {
      stop();
    }
  },
  {
    immediate: true,
  },
);

const itemSyle = (index: number, item: any) => {
  // 正常状态的展示
  const style = {
    'z-index': item.order,
    transition: '0.25s',
    transform: ` translateX(${(item.order - 1) * marginLeft.value}rpx) scale(1)`,
  };
  const total = list.value.length;

  // 缩小中
  if (item.order == 0) {
    style.transform = ` translateX(${item.order * marginLeft.value}rpx) scale(0)`;
  }

  // 位移中 移动到末尾
  if (item.order == -1) {
    style.transform = ` translateX(${(total - 1) * marginLeft.value}rpx) scale(0)`;
    style['z-index'] = total;
  }

  // 进行缩放和位移状态的改变
  // nextState(item);

  if (item.order == total) {
    style['z-index'] = total;
  }
  return style;
};

const itemBaseSyle = () => {
  return { width: props?.width + 'rpx', height: props?.height + 'rpx' };
};

const boxBaseSyle = () => {
  const len = list.value.length;
  return {
    width: len * props?.width - (len - 1) * props?.overlap + 'rpx',
    height: props?.height + 'rpx',
  };
};

// const nextState = (item: any) => {
//   if (item.order == 0) {
//     const timer = setTimeout(() => {
//       // 300后缩放结束 开始移动到末尾
//       item.order = -1;
//       clearTimeout(timer);
//     }, 300);
//   } else if (item.order == -1) {
//     const timer1 = setTimeout(() => {
//       // 300后移动到末尾结束 开始正常展示
//       item.order = list.value.length;
//       clearTimeout(timer1);
//     }, 300);
//   }
// };
const forwardMove = async () => {
  list.value.forEach((item, index) => {
    if (item.order == 1) {
      const timer = setTimeout(() => {
        item.order = 0;
        list.value.shift();
        clearTimeout(timer);
      }, 250);
    }
    item.order--;
  });

  await nextTick();
  const netItem = props?.urls?.[curLastIndex.value];
  let url = netItem && netItem[props?.prop.url || ''];
  if (typeof netItem === 'string') {
    url = netItem;
  }

  const tempItme = { url, order: -1, id: `avatar-${curLastIndex.value}` };
  list.value.push(tempItme);
  const timer1 = setTimeout(() => {
    // 300后移动到末尾结束 开始正常展示
    list.value[list.value.length - 1].order = list.value.length;
    clearTimeout(timer1);
  }, 300);
  if (curLastIndex.value === props?.urls?.length - 1) {
    curLastIndex.value = 0;
  } else {
    curLastIndex.value++;
  }
};
</script>

<style scoped>
.avatar-slider {
  overflow: hidden;
  width: 100%;
}

.relative {
  position: relative;
}
.absolute {
  position: absolute;
}
.br-50_ {
  border-radius: 50%;
}
</style>
