<template>
  <div class="container">
    <p>{{ result }}</p>
    <!-- <p>{{ mock }}</p> -->
    <!-- <p>{{ request }}</p> -->
    <!-- <p>{{ entities }}</p> -->
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
    requiredRequestParams() {
      return this.request.filter(o => o.parameter.required)
    },
    result() {
      if (!this.request.length) {
        return `    /** ${this.mock.description} **/
    ${this.mock.method}${this.mock.url.replace('/', '_')}({}) {
      return this.$http.${this.mock.method}( '${this.mock.url}')
    },`
      } else {
        let res = `    /** ${this.mock.description} **/
    ${this.mock.method}${this.mock.url.replace(
          '/',
          '_'
        )}({}, payload) {\n      if (!payload) {\n        return Promise.reject({ message: "参数错误" })\n`
        let res_end = ''
        if (this.mock.method === 'get' || this.mock.method === 'delete') {
          res_end = `      } else {\n        return this.$http.${this.mock.method}( '${this.mock.url}', { params: { ...payload } } )
      }
    },`
        } else {
          res_end = `      } else {\n        return this.$http.${this.mock.method}( '${this.mock.url}', { ...payload } )
      }
    },`
        }
        if (this.requiredRequestParams.length) {
          this.requiredRequestParams.forEach(o => {
            res += `      } else if ( !payload.${o.name} ) {\n        return Promise.reject({message:" ${o.description}不能为空" })\n`
          })
        }
        return (res += res_end)
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
