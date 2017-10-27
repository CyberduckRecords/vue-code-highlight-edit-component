<template>
  <div>
    <button id="add-html" @click="addWin('html')"><i class="material-icons">add</i>HTML</button>
    <button id="add-css" @click="addWin('css')"><i class="material-icons">add</i>CSS</button>
    <button id="add-js" @click="addWin('javascript')"><i class="material-icons">add</i>JS</button>
    <CdCode v-for="(win, i) in wins" :key="i" class="code" v-model="win.code" :language="win.lang" title="Code Editor" draggable writeable closeable></CdCode>
  </div>
</template>

<script>
import CdCode from '@/components/Cdcode'

export default {
  name: 'app',
  components: {
    CdCode
  },
  data () {
    return {
      wins: []
    }
  },
  methods: {
    addWin (lang) {
      this.wins.push({
        lang: lang,
        code: ''
      })
    }
  }
}
</script>

<style lang="css">
  html, body {
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    background: LightCoral;
  }
  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }
  *:focus {
    outline: none;
  }
  a {
    text-decoration: none;
    color: inherit;
  }
  .code {
    width: 60% !important;
  }
</style>
