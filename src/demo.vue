<template lang="html">
<div style="padding-top: 50px">
  <h1>DEMO vue-code-highlight-edit Vue.js Component</h1>

  <p>
    <vue-code-highlight-edit v-model="value" :maxlength="40" @left-chars="leftChars = $event"></vue-code-highlight-edit>
    <strong>{{ leftChars }}</strong>
  </p>

  <p>
    {{ value }}
  </p>
</div>
</template>

<script>

import Component from '../src/Component.vue'

export default {
  components: {'vue-code-highlight-edit': Component},
  data () {
    return {
      value: 'My Input Value',
      leftChars: 0
    }
  }
}
</script>
