<script>
import api from '@oj/api'

export default {
  data () {
    return {
      formEmailAuth: {
        token: ''
      }
    }
  },
  mounted () {
    this.formEmailAuth.token = this.$route.params.token
    const data = Object.assign({}, this.formEmailAuth)
    api.emailAuth(data).then(res => {
      this.$router.replace({
        path: '/'
      })
    })
  }
}
</script>
