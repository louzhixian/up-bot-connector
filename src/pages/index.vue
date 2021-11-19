<script setup lang="ts">
import UP, { UPAuthMessage } from 'up-core-test'
import axios from 'axios'
import { useUserStore } from '~/stores/user'

const Guild = '881192262439166023'
const ChannelGeneral = '881192262439166025'

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

const openModal = ref(false)
// const targetUrl = `discord:///channels/${Guild}/${ChannelGeneral}`

const connect = async() => {
  UP.config(
    'beta.app.unipass.id',
    'https://beta.app.unipass.id/connect',
    'https://beta.app.unipass.id/authorize')
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

  axios.post('https://dev--up-bot.yamen.autocode.gg/', {
    payload: {
      upid: name.value,
      dcid: dcid.value,
      dcuid: dcuid.value,
      sig,
    },
  })

  axios.post('https://l3testapi.unipass.vip/account/discord', {
    uniPassId: name.value,
    discordId: dcid.value,
  })

  openModal.value = true
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

    <button class="btn btn-active lowercase">
      {{ dcid }}
    </button>

    <div class="flex-col">
      <button
        class="m-8 text-lg btn btn-primary normal-case"
        @click="name.length ? disconnect() : connect()"
      >
        {{ name.length ? `UPID: ${name}` : t('button.connect') }}
      </button>
      <button
        class="m-8 text-lg btn btn-primary normal-case"
        :disabled="!name.length"
        @click="login"
      >
        {{ t('button.authorize') }}
      </button>
    </div>

    <!-- <label for="my-modal-2" class="btn btn-secondary modal-button">open modal</label> -->
    <input id="my-modal-2" v-model="openModal" type="checkbox" class="modal-toggle">
    <div class="modal">
      <div class="modal-box">
        <p class="text-2xl text-success">
          🎉 Authorization Success 🎉
        </p>
        <p class="text-lg mt-6">
          Please close this page and check your discord for welcome message
        </p>
        <div class="modal-action mt-12">
          <label for="my-modal-2" class="btn btn-primary normal-case">Got it</label>
        </div>
      </div>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
