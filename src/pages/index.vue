<script setup lang="ts">
import UP, { UPAuthMessage } from 'up-core-test'
import axios from 'axios'
import { useUserStore } from '~/stores/user'

const user = useUserStore()
const name = computed(() => user.savedName)

const router = useRouter()

const dcid = computed(() => {
  const dcid = router.currentRoute.value.query.dcid
  if (typeof dcid === 'string')
    return dcid.replace('~', '#')
  return null
})

const dcuid = computed(() => router.currentRoute.value.query.dcuid)

const connect = async() => {
  UP.config(
    't.app.unipass.id',
    'https://t.app.unipass.id/connect',
    'https://t.app.unipass.id/sign')
  const upAccount = await UP.connect()
  user.setNewName(upAccount.username)
}

const disconnect = () => {
  UP.disconnect()
  user.savedName = ''
}

const login = async() => {
  const message = `UniPass Bot Connector is connecting the Discord account ${dcid.value} with your UniPass ID ${name.value}. Please confirm.`
  const upMessage = new UPAuthMessage('PLAIN_MSG', name.value, message)
  const sig = await UP.authorize(upMessage)

  // eslint-disable-next-line no-console
  console.log('Authorized: ', sig)

  axios.post('https://dev--up-bot.yamen.autocode.gg/', {
    payload: {
      upid: name.value,
      dcid: dcid.value,
      dcuid: dcuid.value,
      sig,
    },
  })
}

const { t } = useI18n()
</script>

<template>
  <div>
    <p class="text-4xl">
      <!-- <carbon-campsite class="inline-block" /> -->
    </p>
    <p>
      <a rel="noreferrer" href="https://github.com/lay2dev/unipass-bot-connector" target="_blank">
        UniPass Bot Connector
      </a>
    </p>
    <p>
      <em class="text-sm opacity-75">{{ t('intro.desc') }}</em>
    </p>

    <div class="py-4" />

    <input
      id="input"
      v-model="dcid"
      :placeholder="t('intro.your-discord-id')"
      :aria-label="t('intro.your-discord-id')"
      type="text"
      autocomplete="false"
      p="x-4 y-2"
      w="250px"
      text="center"
      disabled
      bg="transparent"
      border="~ rounded gray-200 dark:gray-700"
      outline="none active:none"
      @keydown.enter="connect"
    >

    <div class="flex-col">
      <button
        class="m-8 text-lg btn"
        @click="name.length ? disconnect() : connect()"
      >
        {{ name.length ? `UPID: ${name}` : t('button.connect') }}
      </button>
      <button
        class="m-8 text-lg btn"
        :disabled="!name.length"
        @click="login"
      >
        {{ t('button.authorize') }}
      </button>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
