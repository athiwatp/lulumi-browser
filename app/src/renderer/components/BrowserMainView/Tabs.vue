<template lang="pug">
  #chrome-tabs-shell(@dblclick="onDoubleClick")
    .chrome-tabs(v-sortable="")
      div(v-for="(page, i) in pages", :class="i == currentPageIndex ? 'chrome-tab chrome-tab-draggable chrome-tab-current' : 'chrome-tab chrome-tab-draggable'", :id="`${i}`", :ref="`tab-${i}`")
        svg(width="15", height="30", class="left-edge")
          path(class="edge-bg", d="m14,29l0,-28l-2,0.1l-11.45,27.9l13.2,0z", stroke-linecap="null", stroke-linejoin="null", stroke-dasharray="null", stroke-width="0")
          path(class="edge-border", d="m1,28.5l11.1,-28l1.9,0", stroke-linejoin="round", stroke-dasharray="null", stroke-width="null", fill="none")
        .chrome-tab-bg
          div#tab-icons(class="chrome-tab-favicon")
            i.el-icon-loading(v-if="page.isLoading")
            img(v-show="page.favicon", :src="page.favicon", height='16', width='16', v-else)
            icon(@click.native="$parent.onToggleAudio($event, i, !page.isAudioMuted)", name="volume-off", v-if="page.hasMedia && page.isAudioMuted", class="volume")
            icon(@click.native="$parent.onToggleAudio($event, i, !page.isAudioMuted)", name="volume-up", v-else-if="page.hasMedia && !page.isAudioMuted", class="volume")
          el-tooltip(:content="page.title || 'loading'", placement="bottom", :openDelay="1000")
            span(class="chrome-tab-title", :id="`span-${i}`", @click="$parent.onTabClick($event, parseInt(($event.target.id).split('-')[1]), 10)", @contextmenu.prevent="$parent.onTabContextMenu($event, i)")
              | {{ page.title || 'loading' }}
        a.close(@click="onClose", class="chrome-tab-close")
        svg(width="15", height="30", class="right-edge")
          path(class="edge-bg", d="m14,29l0,-28l-2,0.1l-11.45,27.9l13.2,0z", stroke-linecap="null", stroke-linejoin="null", stroke-dasharray="null", stroke-width="0")
          path(class="edge-border", d="m1,28.5l11.1,-28l1.9,0", stroke-linejoin="round", stroke-dasharray="null", stroke-width="null", fill="none")
      div(class="chrome-tab chrome-tab-add-btn", @click="$parent.onNewTab()")
        svg(width="15", height="30", class="left-edge")
          path(class="edge-bg", d="m14,29l0,-28l-2,0.1l-11.45,27.9l13.2,0z", stroke-linecap="null", stroke-linejoin="null", stroke-dasharray="null", stroke-width="0")
        .chrome-tab-bg
        .chrome-tab-favicon
          i(class="el-icon-plus")
        svg(width="15", height="30", class="right-edge")
          path(class="edge-bg", d="m14,29l0,-28l-2,0.1l-11.45,27.9l13.2,0z", stroke-linecap="null", stroke-linejoin="null", stroke-dasharray="null", stroke-width="0")
</template>

<script>
  import Icon from 'vue-awesome/components/Icon';
  import 'vue-awesome/icons/times';
  import 'vue-awesome/icons/volume-up';
  import 'vue-awesome/icons/volume-off';

  import Sortable from 'sortablejs';

  import { Button, Tooltip } from 'element-ui';

  export default {
    directives: {
      sortable: {
        update(el) {
          Sortable.create(el, {
            draggable: '.chrome-tab-draggable',
            animation: 150,
            ghostClass: 'ghost',
            onUpdate(event) {
              const item = event.item;
              if (!item.dataset.oldIndex) {
                item.dataset.oldIndex = event.oldIndex;
              }
              item.dataset.newIndex = event.newIndex;
            },
          });
        },
      },
    },
    components: {
      Icon,
      'el-button': Button,
      'el-tooltip': Tooltip,
    },
    computed: {
      pages() {
        return this.$store.getters.pages;
      },
      currentPageIndex() {
        return this.$store.getters.currentPageIndex;
      },
    },
    methods: {
      onDoubleClick() {
        const mainWindow = this.$electron.remote.getCurrentWindow();
        if (mainWindow.isMaximized()) {
          mainWindow.unmaximize();
        } else {
          mainWindow.maximize();
        }
      },
      onClose(event) {
        this.$parent.onTabClose(event, parseInt(event.target.parentElement.id, 10));
      },
      onMouseMove(event) {
        const x = event.pageX - event.target.offsetLeft;
        const y = event.pageY - event.target.offsetTop;
        const xy = `${x} ${y}`;

        // eslint-disable-next-line max-len
        const bgWebKit = `-webkit-gradient(radial, ${xy}, 0, ${xy}, 100, from(rgba(255,255,255,0.8)), to(rgba(255,255,255,0.0))), linear-gradient(to bottom, #ddd 90%, #f5f5f5)`;

        const target = event.target.parentNode.parentNode;
        if (!target.classList.contains('chrome-tab-current')) {
          target.getElementsByClassName('left-edge')[0].style.background = bgWebKit;
          target.getElementsByClassName('chrome-tab-bg')[0].style.background = bgWebKit;
          target.getElementsByClassName('right-edge')[0].style.background = bgWebKit;
        }
      },
      onMouseLeave(event) {
        const target = event.target.parentNode.parentNode;
        if (!target.classList.contains('chrome-tab-current')) {
          target.getElementsByClassName('left-edge')[0].style.background = '';
          target.getElementsByClassName('chrome-tab-bg')[0].style.background = '';
          target.getElementsByClassName('right-edge')[0].style.background = '';
        }
      },
    },
    mounted() {
      const ipc = this.$electron.ipcRenderer;
      ipc.on('reload', () => {
        if (this.$parent.onClickRefresh) {
          this.$parent.onClickRefresh();
        }
      });
    },
  };
</script>

<style lang="less" scoped>
  #chrome-tabs-shell {
    height: 38px;
    position: relative;
    padding: 0 18px 0 0px;
    border-bottom: 1px solid #999;
  }

  .chrome-tabs {
    position: relative;
    display: flex;
    font-size: 15px;

    * {
      -webkit-user-select: none;
      cursor: default;
      font-family: "Lucida Grande", "Lucida Sans Unicode", Verdana, Arial, Helvetica, sans-serif;
      font-size: 12px;
      line-height: 13px;
    }

    .chrome-tab {
      border: 0 !important;
      position: relative;
      margin: 0 -5px;
      top: 7px;
      z-index: 1;
      -webkit-app-region: no-drag;
      transition: .1s transform;
      border-bottom: 1px solid #999;
      text-align: start;

      &:not(:last-child) {
        flex: 1;
      }

      &:last-child {
        width: 45px;
        z-index: 0;
        opacity: .7;
      }

      // tab decoration
      svg {
        position: absolute;
        top: 0;
        height: 29px;

        .edge-bg {
          fill: #ddd;
          transition: .2s fill;
        }
        .edge-border {
          stroke: #808080;
          stroke-width: 1px;
        }

        &.right-edge {
          transform: scaleX(-1);
          right: 0;
        }
      }
      .chrome-tab-bg {
        position: absolute;
        display: flex;
        align-items: center;
        top: 0;
        left: 14px;
        right: 14px;
        height: 29px;
        background: #ddd;
        border-top: 1px solid #808080;
        transition: .2s background;
      }

      .chrome-tab-favicon {
        position: relative;
        display: flex;
        align-items: center;
        left: 10px;
        z-index: 3;

        img {
          width: 16px;
          height: 16px;
          padding-right: 2px;
        }

        svg.volume {
          position: relative;
          flex: 1;
          width: 16px;
          height: 16px;
          padding: 0 3px;
        }
      }
      .spinner {
        position: relative;
        left: 1px;
        top: 1px;
      }
      &.chrome-tab-pinned {
        .chrome-tab-favicon {
          left: 20px;
        }
      }

      .chrome-tab-title {
        position: relative;
        flex: 1;
        color: #222222;
        padding: 17px 20px 0px 16px;
        height: 28px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      &.chrome-tab-nofavicon .chrome-tab-title {
        padding-left: 16px;
      }

      .chrome-tab-close {
        display: none;
        position: absolute;
        right: 14px;
        top: 8px;
        width: 15px;
        height: 15px;
        z-index: 2;
        border-radius: 8px;

        &:before {
          content: "\00D7";
          position: absolute;
          top: 1px;
          right: 3px;
          font-size: 12px;
          line-height: 12px;
          color: #777777;
        }
        &:hover:before,
        &:active:before {
          color: white;
        }
        &:hover,
        &:active {
          background: #f17469;
        }
        &:active:before {
        }
      }

      &:hover {
        .chrome-tab-bg {
          background: #f0f0f0;
        }
        svg .edge-bg {
          fill: #f0f0f0;
        }
        .chrome-tab-close {
          display: block;
        }
      }

      &.chrome-tab-dragging {
        transition: none !important;
      }

      &.chrome-tab-current {
        z-index: 2;

        svg {
          height: 29px;
          .edge-bg {
            fill: #f5f5f5;
          }
        }
        .chrome-tab-bg {
          height: 29px;
          background: #f5f5f5;
        }
      }

      &.chrome-tab-add-btn {
        svg, .chrome-tab-bg {
          visibility: hidden;
          border: 0;
        }

        .chrome-tab-favicon {
          top: 6px;
          padding: 0 6px;
        }
        .icon {
          font-size: 17px;
          line-height: 15px;
          color: #666;
        }

        &:hover {
          svg, .chrome-tab-bg {
            visibility: visible;
          }
          svg .edge-bg {
            fill: #ccc;
          }
          .chrome-tab-bg {
            background: #ccc;
          }
          .icon {
            color: #444;
          }
        }
      }
    }
  }

  .chrome-tab-draggable {
    cursor: move;
	  cursor: -webkit-grabbing;
  
    &.ghost {
      opacity: .5;
    }
  }

  // make room for the traffic lights
  .darwin:not(.fullscreen) .chrome-tabs {
    padding-left: 80px;
    display: flex;
  }
</style>
