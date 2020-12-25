<template>
  <div class="app-title-bar" :class="isFocus ? 'is-focus' : 'is-blur'">
    <div class="drag-region">
      <div class="window-appicon" />
      <div class="window-title">
        <span>{{ `${appName} v${appVersion}` }}</span>
      </div>
      <div class="window-actions">
        <div class="min-button" @click="handleWindowAction('window-min')">
          <iconfont name="win-minimize" />
        </div>
        <div v-if="!isMaximized" class="max-button" @click="handleWindowAction('window-max')">
          <iconfont name="win-maximize" />
        </div>
        <div v-if="isMaximized" class="restore-button" @click="handleWindowAction('window-max')">
          <iconfont name="win-restore" />
        </div>
        <div class="close-button" @click="handleWindowAction('app-exit')">
          <iconfont name="win-close" />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, reactive, ref, toRefs } from 'vue'
import { useIpc, useService } from '/@/hooks'
import { Modal } from 'ant-design-vue'

// TODO: 失去焦点时的背景色和字体颜色变更

export default defineComponent({
  name: 'TitleBar',
  props: {
    appName: String,
  },
  setup(props, { emit }) {
    const { getBasicInformation } = useService('BaseService')
    const appName = ref(props.appName)
    const appVersion = ref('')
    const exitConfirmVisiable = ref(false)
    const isFocus = ref(false)
    const isMaximized = ref(false)

    // 获取版本信息
    const fetchAppVersion = async () => {
      const { electron } = await getBasicInformation()
      appVersion.value = electron
    }
    fetchAppVersion()

    // 窗口控制
    const handleWindowAction = (action: string) => {
      if (action === 'app-exit') {
        Modal.confirm({
          title: '确认退出吗？',
          cancelText: '取消',
          okText: '退出',
          onOk() {
            useIpc().send('renderer2main', 'app-exit')
          },
        })
      } else useIpc().send('renderer2main', action)
    }

    useIpc().on('updateWindowStatus', (event, result) => {
      if (result.isFocus !== undefined) isFocus.value = result.isFocus
      if (result.isMaximized !== undefined) isMaximized.value =result.isMaximized
    })

    const data = reactive({
      appName,
      appVersion,
      exitConfirmVisiable,
      isFocus,
      isMaximized,
    })

    return {
      ...toRefs(data),
      handleWindowAction,
    }
  },
})
</script>

<style lang="less">
@import url('../../themes/variables');

.app-title-bar {
  display: block;
  position: fixed;
  width: 100%;

  &.is-blur {
    background-color: @app-background;
  }
  &.is-focus {
    background-color: @app-component-background;
  }

  .drag-region {
    width: 100%;
    height: 100%;
    -webkit-app-region: drag;
    display: grid;
    grid-template-columns: 35px auto 138px;
  }

  .window-appicon {
    grid-column: 1;
    width: 35px;
    height: 100%;
    position: relative;
    z-index: 3000;
    background-image: url('../../assets/logo.png');
    background-repeat: no-repeat;
    background-position: 50%;
    background-size: 16px;
    flex-shrink: 0;
  }

  .window-title {
    grid-column: 2;
    display: flex;
    justify-content: center;
    overflow: hidden;
    font-size: @font-size-sm;
    color: @text-color-secondary;

    span {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      line-height: 30px;
    }
  }

  .window-actions {
    user-select: none;
    -webkit-app-region: no-drag;
    display: grid;
    grid-template-columns: repeat(3, 46px);
    position: absolute;
    top: 0;
    right: 0;
    height: 100%;

    .min-button,
    .max-button,
    .restore-button,
    .close-button {
      grid-row: 1 / span 1;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 100%;
    }

    .min-button {
      grid-column: 1;
    }
    .max-button,
    .restore-button {
      grid-column: 2;
    }
    .close-button {
      grid-column: 3;
    }

    .min-button:hover,
    .max-button:hover {
      background: lighten(@component-background, 4%);
    }
    .min-button:active,
    .max-button:active {
      background: lighten(@component-background, 8%);
    }

    .close-button:hover {
      background: #e81123;
    }
    .close-button:active {
      background: #f1707a;
      .icon {
        filter: invert(1);
      }
    }
    .icon {
      font-size: 16px;
    }
  }

  // @media (-webkit-device-pixel-ratio: 1.5),
  //   (device-pixel-ratio: 1.5),
  //   (-webkit-device-pixel-ratio: 2),
  //   (device-pixel-ratio: 2),
  //   (-webkit-device-pixel-ratio: 3),
  //   (device-pixel-ratio: 3) {
  //   .window-actions .icon {
  //     width: 10px;
  //     height: 10px;
  //   }
  // }
}

// .restore-button {
//   display: none !important;
// }
// .maximized {
//   .restore-button {
//     display: flex !important;
//   }
//   .max-button {
//     display: none;
//   }
// }
</style>