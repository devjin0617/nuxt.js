<template>
  <div id="__nuxt">
    <component v-if="layout" :is="layout"></component>
  </div>
</template>

<script>
let layouts = {
<%
var layoutsKeys = Object.keys(layouts);
layoutsKeys.forEach(function (key, i) { %>
  "_<%= key %>": process.BROWSER_BUILD ? () => System.import('<%= layouts[key] %>') : require('<%= layouts[key] %>')<%= (i + 1) < layoutsKeys.length ? ',' : '' %>
<% }) %>
}

export default {
  head: <%= JSON.stringify(head) %>,
  data: () => ({
    layout: null,
    layoutName: ''
  }),
  methods: {
    setLayout (layout) {
      if (!layout || !layouts['_' + layout]) layout = 'default'
      this.layoutName = layout
      let _layout = '_' + layout
      if (typeof layouts[_layout] === 'function') {
        return this.loadLayout(_layout)
      }
      this.layout = layouts[_layout]
      return Promise.resolve(this.layout)
    },
    loadLayout (_layout) {
      return layouts[_layout]()
      .then((Component) => {
        layouts[_layout] = Component
        this.layout = layouts[_layout]
        return this.layout
      })
      .catch((e) => {
        if (this.$nuxt) {
          return this.$nuxt.error({ statusCode: 500, message: e.message })
        }
        console.error(e)
      })
    }
  }
}
</script>

<% css.forEach(function (c) { %>
<style src="<%= (typeof c === 'string' ? c : c.src) %>" lang="<%= (c.lang ? c.lang : 'css') %>"></style>
<% }) %>
