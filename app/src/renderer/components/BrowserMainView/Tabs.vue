<template lang="pug">
  #browser-tabs(v-sortable="", @dblclick="$electron.remote.getCurrentWindow().maximize()")
    .window-buttons
    .browser-tab(v-for="(page, i) in pages", :class="i == currentPageIndex ? 'active' : ''")
      <transition name="fade">
        div#tab-icons(@click="$parent.onToggleAudio($event, i, !page.isAudioMuted)")
          i.el-icon-loading(v-if="page.isLoading")
          img(v-show="page.favicon", :src="page.favicon", height='16', width='16', v-else)
          icon(name="volume-off", v-if="page.hasMedia && page.isAudioMuted")
          icon(name="volume-up", v-else-if="page.hasMedia && !page.isAudioMuted")
      </transition>
      span(:id="`${i}`", @click="$parent.onTabClick($event, parseInt($event.target.id))", @contextmenu.prevent="$parent.onTabContextMenu($event, i)")
        | {{ page.title || 'loading' }}
      a.close(@click="onClose")
        icon(name="times")
    a.newtab(@click="$parent.onNewTab()")
</template>

<script>
  import Icon from 'vue-awesome/components/Icon';
  import 'vue-awesome/icons/times';
  import 'vue-awesome/icons/volume-up';
  import 'vue-awesome/icons/volume-off';

  import Sortable from 'sortablejs';

  export default {
    directives: {
      sortable: {
        update(el) {
          Sortable.create(el, {
            draggable: '.browser-tab',
            animation: 150,
            ghostClass: 'ghost',
          });
        },
      },
    },
    components: {
      Icon,
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
      onClose(event) {
        let id = null;
        try {
          id = event.target.previousSibling.id;
        } catch (e) {
          try {
            id = event.target.parentNode.previousSibling.id;
          } catch (e) {
            id = event.target.parentNode.parentNode.previousSibling.id;
          }
        }
        this.$parent.onTabClose(event, parseInt(id, 10));
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

<style scoped>
  #browser-tabs {
    display: flex;
    background: linear-gradient(to bottom, #ddd 90%, #bbb);
    font-size: 15px;
    height: 38px;
    -webkit-touch-callout: none;
    -webkit-user-select: none;
  }
  #browser-tabs > div.window-buttons {
    -webkit-app-region: drag;
    padding-right: 80px;
  }
  #browser-tabs > div.browser-tab {
    flex: 1;
    display: flex;
    background: linear-gradient(to bottom, #ddd 90%, #bbb);
    color: #777;
    border-top: 1px solid #ccc;
    border-right: 1px solid #ccc;
    cursor: default;
    white-space: nowrap;
    overflow:hidden;
  }
  #browser-tabs > div.active {
    background: linear-gradient(to bottom, #e5e5e5 90%, #ddd);
    color: #555;
  }
  #browser-tabs > div.ghost {
    opacity: .5;
    background: #C8EBFB;
  }
  #browser-tabs > div > span {
    flex: 1;
    padding: 6px 0 4px 12px;
    line-height: 16px;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .browser-tab {
    align-items: center;
  }
  #tab-icons {
    margin-top: 3px;
  }
  #tab-icons > div, #tab-icons > img {
    margin-left: 10px;
  }
  #tab-icons > i, #tab-icons > svg {
    width: 20px;
    margin-left: 10px;
  }
  #browser-tabs > div > a {
    -webkit-app-region: no-drag;
    display: inline-block;
    padding: 5px 10px;
    margin-right: 5px;
    cursor: pointer;
    float: right;
  }
  #browser-tabs > div > a:hover {
    background: rgb(245, 119, 119);
    color: #fff;
  }
  #browser-tabs > a {
    -webkit-app-region: no-drag;
    display: inline-block;
    color: #888;
    font-size: 11px;
    cursor: pointer;
    margin: 6px 4px;
  }
  #browser-tabs > a.newtab {
    padding: 8px 12px 5px 10px;
    margin: 0;
  }
  #browser-tabs > a.newtab:hover {
    color: #fff;
    background: rgb(99, 190, 229);
  }
  #browser-tabs > a.maximize {
    color: rgb(32, 205, 20);
    text-shadow: 0 0 1px rgba(8, 77, 3, 1);
  }
  #browser-tabs > a.maximize svg {
    font-size: 12.5px
  }
  #browser-tabs > a.minimize {
    color: rgb(255, 213, 0);
    text-shadow: 0 0 1px rgba(65, 42, 2, 1);
  }
  #browser-tabs > a.minimize svg {
    font-size: 12.5px
  }
  #browser-tabs > a.close {
    color: red;
    text-shadow: 0 0 1px rgba(50, 1, 1, 1);
    margin-left: 8px;
  }
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s
  }
  .fade-enter, .fade-leave-active {
    opacity: 0
  }
</style>