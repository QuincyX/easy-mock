<template>
  <div class="container">
    <p>{{ result }}</p>
    <p>{{ mock }}</p>
    <p>{{ request }}</p>
    <p>{{ entities }}</p>
    <p>{{ response }}</p>
  </div>
</template>

<script>
export default {
  props: {
    request: '',
    entities: '',
    response: '',
    mock: ''
  },
  computed: {
    result() {
      if (!this.request.length) {
        return `    /** ${this.mock.description} **/
    ['${this.mock.method}${this.mock.url.replace('/', '_')}']({}) {
      return this.$http.${this.mock.method}( '${this.mock.url}')
    },`
      } else {
        return `    /** ${this.mock.description} **/
    ${this.mock.method}${this.mock.url.replace('/', '_')}({}, payload) {
      if (!payload) {
        return Promise.reject({ message: "参数错误" })
      } else {
        return this.$http.post( '${this.mock.url}',{ ...payload })
      }
    },`
      }
    }
  }
}
</script>

<style>
.container {
  white-space: pre-wrap;
}
</style>
