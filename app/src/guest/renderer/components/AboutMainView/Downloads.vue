<template lang="pug">
  div
    el-row(type="flex", align="middle")
      el-col(:span="12")
        h1 Downloads
      el-col(:span="6", :offset="3")
        el-button(type="info", @click="setDownloads(-1)") Clear
    el-row
      el-col(:span="24")
        ul(class="download-list")
          li(v-for="(file, index) in files", :key="index", class="download-list__item")
            span(v-if="file.state !== 'progressing'", class="el-icon-close", @click="setDownloads(index)")
            el-progress(:status="checkStateForProgress(file.state)", :text-inside="true", :percentage="percentage(file)", :stroke-width="18", class="download-list__item-progress")
            a(class="download-list__item-name download-list__item-link", :href="`${file.url}`")
              i(class="el-icon-document") {{ file.name }}
            el-button-group
              el-button(:disabled="file.state !== 'progressing'", v-if="file.isPaused && file.canResume", :plain="true", type="warning", size="small", icon="caret-right", @click="resumeDownload(file.startTime)")
              el-button(:disabled="file.state !== 'progressing'", v-else, :plain="true", type="warning", size="small", icon="minus", @click="pauseDownload(file.startTime)")
              el-button(:disabled="file.state !== 'progressing'", :plain="true", type="danger", size="small", icon="circle-close", @click="cancelDownload(file.startTime)")
              el-button(:disabled="file.state === 'cancelled'", :plain="true", type="info", size="small", icon="document", @click="showItemInFolder(file.savePath)")
</template>

<script>
  export default {
    data() {
      return {
        files: [],
        handler: null,
      };
    },
    methods: {
      fetch() {
        this.handler = setInterval(() => {
          // eslint-disable-next-line no-undef
          ipcRenderer.send('guest-want-data', 'downloads');
        }, 100);
      },
      clear() {
        clearInterval(this.handler);
      },
      percentage(file) {
        return parseInt((file.getReceivedBytes / file.totalBytes) * 100, 10) || 0;
      },
      setDownloads(index) {
        if (index !== -1) {
          this.files.splice(index, 1);
        } else {
          this.files = this.files.filter(file => file.state === 'progressing');
        }
        // eslint-disable-next-line no-undef
        ipcRenderer.send('set-downloads', this.files);
      },
      showItemInFolder(savePath) {
        // eslint-disable-next-line no-undef
        ipcRenderer.send('show-item-in-folder', savePath);
      },
      checkStateForProgress(state) {
        switch (state) {
          case 'progressing':
            return '';
          case 'cancelled':
          case 'interrupted':
            return 'exception';
          case 'completed':
          default:
            return 'success';
        }
      },
      pauseDownload(startTime) {
        // eslint-disable-next-line no-undef
        ipcRenderer.send('pause-downloads-progress', startTime);
      },
      resumeDownload(startTime) {
        // eslint-disable-next-line no-undef
        ipcRenderer.send('resume-downloads-progress', startTime);
      },
      cancelDownload(startTime) {
        // eslint-disable-next-line no-undef
        ipcRenderer.send('cancel-downloads-progress', startTime);
      },
    },
    mounted() {
      // eslint-disable-next-line no-undef
      ipcRenderer.on('guest-here-your-data', (event, ret) => {
        this.files = ret;
      });
      this.fetch();
    },
    beforeDestroy() {
      this.clear();
      // eslint-disable-next-line no-undef
      ipcRenderer.removeAllListeners([
        'guest-here-your-data',
      ]);
    },
  };
</script>

<style>
  .download-list {
    margin-top: 10px;
    padding: 0;
    list-style: none;
  }
  .download-list__item {
    overflow: hidden;
    background-color: #fff;
    border: 1px solid #c0ccda;
    border-radius: 6px;
    box-sizing: border-box;
    margin: 10px 2px;
    padding: 5px 10px;
    width: auto;
    display: flex;
    align-items: center;
    justify-content: space-around;
  }
  a.download-list__item-link {
    text-decoration: none;
  }
  a.download-list__item-link:hover {
    color: green;
  }
  .download-list__item-name {
    color: #48576a;
    padding-left: 4px;
    transition: color .3s;
    flex: 5;
  }
  .download-list__item-name > i {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    width: 400px;
  }
  .download-list__item-progress {
    right: 0;
    flex: 1;
  }
</style>
