<template>
  <b-card-body :title="$t(`view.confirm-email.title`)">
    <div
      v-if="error"
      class="error"
    >
      <p class="text-danger mb-1">
        {{ $t('general.error-tpl', { error: parseError(error) }) }}
      </p>
      <p>{{ $t(`view.confirm-email.check-or-repeat`) }}</p>
    </div>
    <div
      v-else-if="processing"
      class="processing"
    >
      {{ $t(`view.confirm-email.sending`) }}
    </div>
    <div
      v-else
      class="confirmed"
    >
      {{ $t(`view.confirm-email.confirmed`) }}
    </div>
    <div v-if="internalSignUpEnabled">
      <router-link :to="{ name: 'auth:login'}">
        {{ $t('link.login') }}
      </router-link>
    </div>
  </b-card-body>
</template>

<script>
import { tokenRegex } from 'corteza-webapp-auth/src/lib/common'

export default {
  name: 'ConfirmEmail',

  props: {
    internalSignUpEnabled: {
      type: Boolean,
      default: false,
    },
    afterConfirmEmail: {
      type: Function,
      default: null,
    },
  },

  data () {
    return {
      processing: true,
      error: null,
    }
  },

  created () {
    const token = this.$route.query.token
    if (!token || !tokenRegex.test(token)) {
      this.error = 'Invalid token'
    } else {
      this.confirmToken(token)
    }
  },

  methods: {
    afterConfirm () {
      window.setTimeout(() => {
        this.$auth.goto('/')
      }, 3000)
    },

    confirmToken (token) {
      this.error = null
      this.processing = true

      this.$SystemAPI.authInternalConfirmEmail({ token }).then(({ jwt, user } = {}) => {
        this.$auth.JWT = jwt
        this.$auth.user = user
        if (this.afterConfirmEmail) {
          this.afterConfirmEmail()
        } else {
          this.afterConfirm()
        }
      }).catch(({ message } = {}) => {
        this.error = message
      }).finally(() => {
        this.processing = false
      })
    },
  },
}
</script>
