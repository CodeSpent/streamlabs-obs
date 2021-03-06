<template>
<div
  class="live-dock"
  :class="{ collapsed, 'can-animate': canAnimate, 'live-dock--left': onLeft }"
  :style="{ width: (liveDockSize) + 'px' }">
  <div
    class="live-dock-chevron icon-button"
    v-if="collapsed"
    @click="expand">
    <i :class="{
      'icon-down icon-left': !onLeft,
      'icon-down icon-right': onLeft
    }" />
  </div>

  <transition name="slide-fade">
    <div
      v-if="!collapsed"
      class="live-dock-expanded-contents">
      <div
        class="live-dock-chevron icon-button"
        @click="collapse">
        <i
          :class="{
          'icon-down icon-left': onLeft,
          'icon-down icon-right': !onLeft
          }" />
      </div>
      <div class="live-dock-header">
        <div class="flex flex--center">
          <div :class="{ 'live-dock-pulse': true, 'live-dock-offline': !isStreaming  }" />
          <span class="live-dock-text">
            {{ liveText }}
          </span>
          <span class="live-dock-timer">
            {{ elapsedStreamTime }}
          </span>
        </div>
        <div class="live-dock-viewer-count">
          <i
            :class="{
              'icon-view': !hideViewerCount,
              'icon-hide': hideViewerCount
            }"
            @click="toggleViewerCount"/>
          <span class="live-dock-viewer-count__count">{{ viewerCount }}</span><span v-if="viewerCount >= 0">{{ $t('viewers')}}</span>
        </div>
      </div>

      <div class="live-dock-info">
        <div class="live-dock-platform-tools">
          <a
            @click="showEditStreamInfo"
            v-if="isTwitch || isMixer || (isYoutube && isStreaming) || isFacebook"
            v-tooltip="editStreamInfoTooltip">
            <i class="icon-edit" />
          </a>
          <a
            @click="openYoutubeStreamUrl"
            v-if="isYoutube && isStreaming"
            v-tooltip="viewStreamTooltip">
            <i class="icon-studio" />
          </a>
          <a
            @click="openYoutubeControlRoom"
            v-if="isYoutube && isStreaming"
            v-tooltip="controlRoomTooltip">
            <i class="icon-settings" />
          </a>
        </div>
        <div class="flex">
          <a @click="refreshChat" v-if="isTwitch || isMixer || (isYoutube && isStreaming) || isFacebook">
            {{ $t('Refresh Chat') }}
          </a>
        </div>
      </div>

      <div class="live-dock-chat" v-if="!hideStyleBlockingElements && (isTwitch || isMixer || (isYoutube && isStreaming) || isFacebook)">
          <div v-if="hasChatApps" class="flex">
            <tabs :tabs="chatTabs" v-model="selectedChat" :hideContent="true" />
            <i
              class="live-dock-chat-apps__popout icon-pop-out-1"
              v-tooltip.left="$t('Pop out to new window')"
              v-if="isPopOutAllowed"
              @click="popOut"
            />
          </div>
        <!-- v-if is required because left-side chat will not properly load on application startup -->
        <chat v-if="!applicationLoading && selectedChat === 'default'" />
        <PlatformAppPageView
          v-if="selectedChat !== 'default'"
          class="live-dock-platform-app-webview"
          :appId="selectedChat"
          :pageSlot="slot"
          :key="selectedChat"
        />
      </div>
      <div class="flex flex--center flex--column live-dock-chat--offline" v-else >
        <img class="live-dock-chat__img--offline" :src="offlineImageSrc">
        <span v-if="!hideStyleBlockingElements">{{ $t('Your chat is currently offline') }}</span>
      </div>
    </div>
  </transition>
</div>
</template>

<script lang="ts" src="./LiveDock.vue.ts"></script>

<style lang="less" scoped>
@import '../styles/index';

.live-dock {
  position: relative;
  z-index: 1000;
  width: 28%;
  box-sizing: border-box;
  border-left: 1px solid var(--border);
  .padding(2);

  &.can-animate {
    .transition();
  }

  &.live-dock--left {
    border-right: 1px solid var(--border);

    .live-dock-chevron {
      right: 5px;
      left: inherit;
    }
  }

  &.collapsed {
    width: 20px !important;
    padding: 0;
  }

  @media (max-width: 1100px) {
    display: none;
  }
}

.live-dock-chevron {
  cursor: pointer;
  .absolute(@top: 0, @bottom: 0, @left: 0);
  display: flex;
  align-items: center;
  height: 100%;
  padding-left: 4px;

  &:hover {
    opacity: 1;
  }

  i {
    font-size: 10px;
  }
}

.live-dock-end-stream {
  margin-left: 10px;
}

.live-dock-header {
  display: flex;
  flex-direction: row;
  .margin-bottom();
  align-items: center;
  justify-content: space-between;
}

.live-dock-text {
  margin: 0 2px 0 4px;
  .weight(@medium);
}

.live-dock-expanded-contents {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.live-dock-info {
  display: flex;
  justify-content: space-between;
  .margin-bottom();

  .live-dock-platform-tools {
    a {
      padding: 0 8px;
    }
  }
}

.live-dock-viewer-count {
  .flex();
  .flex--center();

  i {
    .margin-right();
  }

  .live-dock-viewer-count-toggle {
    opacity: 0;
    cursor: pointer;
  }

  &:hover {
    .live-dock-viewer-count-toggle {
      opacity: 1;
    }
  }
}

.live-dock-viewer-count__count {
  padding-right: 3px;
}

.live-dock-chat {
  .flex();
  .flex--column();
  .flex--grow();
}

.live-dock-chat--offline {
  height: 100%;
}

.live-dock-chat__img--offline {
  width: 60%;
  .flex();
  .flex--center();
  .flex--column();
  margin-bottom: 16px;
}

.live-dock-pulse {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--warning);
  margin: 0 8px;
  box-shadow: 0 0 0 rgba(252, 62, 63, 0.4);

  &.live-dock-offline {
    background: var(--icon);
    animation: none;
  }
}

.live-dock-platform-tools {
  .flex();
}

.live-dock-chat-apps__popout {
  .padding();
  .cursor--pointer();
}

.live-dock-platform-app-webview {
  .flex--grow();
}
</style>
