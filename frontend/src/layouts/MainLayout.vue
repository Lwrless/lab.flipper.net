<template>
  <q-layout view="hhh LpR fff">
    <q-header>
      <q-toolbar>
        <q-btn
          v-if="$q.screen.width <= 900"
          @click="leftDrawer = !leftDrawer"
          icon="menu"
          dense
          flat
          round
          class="q-mr-xs"
        ></q-btn>

        <img
          v-show="!$q.screen.xs"
          src="../assets/flipper_lab_logo_monochrome.svg"
          class="q-ml-xs"
          style="height: 36px;"
        />

        <q-space />

        <q-btn
          v-if="$q.screen.width <= 750"
          @click="linksMenu = !linksMenu"
          icon="open_in_new"
          dense
          flat
          round
          class="q-ml-sm"
        >
          <q-menu fit>
            <q-list class="nav-links">
              <ExternalLink
                v-for="link in extLinks"
                :key="link.title"
                v-bind="link"
              />
            </q-list>
          </q-menu>
        </q-btn>
        <template v-else>
          <div class="nav-links">
            <a v-for="link in extLinks"
              :key="link.title"
              v-bind="link"
              :href="link.link"
            >{{ link.title }}</a>
          </div>
        </template>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawer"
      class="bg-grey-2"
      style="overflow-x: hidden"
      :width="175"
      :breakpoint="900"
    >
      <div
        class="full-height relative-position"
        style="width: 200%; display: grid; grid-template-rows: 1fr; grid-template-columns: 175px 175px; transition-duration: 300ms;"
        :style="`left: ${flags.settingsView ? '-175px' : '0'}`"
      >
        <q-list style="width: 175px">
          <RouterLink
            v-for="link in routes"
            :key="link.title"
            v-bind="link"
            class="q-px-lg q-py-md"
          />

          <div :class="$q.screen.height > 545 ? 'absolute-bottom' : ''" style="width: 175px">
            <q-item
              clickable
              class="q-px-lg q-py-md"
              @click="flags.settingsView = true"
            >
              <q-item-section avatar style="min-width: initial;">
                <q-icon name="svguse:common-icons.svg#settings"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>Settings</q-item-label>
              </q-item-section>
            </q-item>
            <q-separator style="width: 85%; margin: auto;"/>
            <q-item
              v-if="flags.portSelectRequired || !flags.connected && !flags.portSelectRequired"
              clickable
              class="q-px-md q-py-sm"
              @click="flags.portSelectRequired ? selectPort() : start(true)"
            >
              <q-item-section avatar style="min-width: initial; position: relative; right: -3px;">
                <q-icon name="svguse:common-icons.svg#connect" size="32px"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>Connect</q-item-label>
              </q-item-section>
            </q-item>
            <q-item
              v-else-if="info"
              clickable
              class="q-px-md q-py-sm"
              @click="disconnect"
            >
              <q-item-section avatar style="min-width: initial; position: relative; right: -3px;">
                <q-icon name="svguse:common-icons.svg#connected" size="32px"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>Connected</q-item-label>
              </q-item-section>
            </q-item>
            <q-item
              v-else
              clickable
              class="q-px-md q-py-sm"
              @click="flags.portSelectRequired ? selectPort() : start(true)"
            >
              <q-item-section avatar style="min-width: initial; position: relative; right: -3px;">
                <q-icon name="svguse:common-icons.svg#connect" size="32px"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>Connecting...</q-item-label>
              </q-item-section>
            </q-item>
          </div>
        </q-list>
        <div class="relative-position flex justify-end" style="width: 175px">
          <div class="column justify-end no-wrap">
            <div class="column items-center">
              <div v-if="info && info.hardware && info.power" class="flex justify-center q-px-md">
                <img v-if="info.hardware.color === '1'" src="../assets/flipper_black.svg" style="width: 100%"/>
                <img v-else-if="info.hardware.color === '3'" src="../assets/flipper_transparent.svg" style="width: 100%"/>
                <img v-else src="../assets/flipper_white.svg" style="width: 100%"/>
                <div class="flex full-width justify-between items-center q-mt-md q-mb-sm">
                  <div style="font-size: 1rem; font-weight: 600;">{{ info.hardware.name }}</div>
                  <div class="flex flex-center">
                    <q-icon
                      :name="batteryIcon"
                      size="22px"
                      class="rotate-90"
                      :color="batteryColor"
                    ></q-icon>
                    <div class="q-ml-xs text-caption">{{ info.power.charge.level }}%</div>
                  </div>
                </div>

                <div class="flex full-width justify-between items-center q-mt-sm q-mb-xs">
                  <div>Internal used:</div>
                  <div class="text-bold">{{ internalUsed }}%</div>
                </div>
                <q-linear-progress style="height: 8px; border-radius: 8px;" :value="internalUsed / 100" class="q-mb-sm"/>

                <template v-if="info.storage.sdcard.status.isInstalled">
                  <div class="flex full-width justify-between items-center q-mt-xs">
                    <div>SD card used:</div>
                    <div class="text-bold">{{ sdCardUsed }}%</div>
                  </div>
                  <q-linear-progress style="height: 8px; border-radius: 8px;" :value="Math.ceil(sdCardUsed/2)*2 / 100" class="q-mb-sm"/>
                </template>
              </div>

              <div class="q-my-md q-px-md">
                <q-toggle
                  size="2.25rem"
                  dense
                  class="q-my-sm"
                  v-model="flags.connectOnStart"
                  @click="toggleConnectOnStart"
                  label="Connect on load"
                ></q-toggle>
                <q-toggle
                  size="2.25rem"
                  dense
                  class="q-my-sm"
                  v-model="flags.autoReconnect"
                  @click="toggleAutoReconnect"
                  label="Auto reconnect"
                ></q-toggle>
                <q-toggle
                  size="2.25rem"
                  dense
                  class="q-my-sm"
                  v-model="flags.installFromFile"
                  @click="toggleInstallFromFile"
                  label="Third-party FW"
                ></q-toggle>

                <q-toggle
                  v-if="flags.catalogCanBeEnabled"
                  size="2.25rem"
                  dense
                  class="q-mt-lg q-mb-sm"
                  v-model="flags.catalogEnabled"
                  @click="toggleCatalogEnabled"
                  label="Enable Apps"
                ></q-toggle>
                <q-toggle
                  v-if="flags.catalogCanSwitchChannel && flags.catalogEnabled"
                  size="2.25rem"
                  dense
                  class="q-my-sm"
                  v-model="flags.catalogChannelProduction"
                  @click="toggleCatalogChannel"
                  label="Production Apps"
                ></q-toggle>
              </div>
            </div>
            <q-item
              clickable
              class="q-px-lg q-py-sm"
              @click="flags.logsPopup = true"
            >
              <q-item-section avatar style="min-width: initial;">
                <q-icon name="svguse:common-icons.svg#logs"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>View logs</q-item-label>
              </q-item-section>
            </q-item>
            <q-item
              clickable
              class="q-px-lg q-py-sm"
              @click="flags.settingsView = false"
            >
              <q-item-section avatar style="min-width: initial;">
                <q-icon name="mdi-chevron-left" size="2rem" style="margin-left: -4px; margin-right: -4px;"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>Back</q-item-label>
              </q-item-section>
            </q-item>
          </div>
        </div>
      </div>
    </q-drawer>

    <q-page-container class="flex justify-center">
      <router-view
        v-if="!flags.connectionRequired || flags.updateInProgress || (flags.serialSupported && info !== null && info.doneReading)"
      />
      <q-page v-else class="flex-center column">
        <div
          v-if="flags.serialSupported && (!flags.connected || info == null || !flags.rpcActive || flags.rpcToggling)"
          class="flex-center column q-my-xl"
        >
          <q-btn
            v-if="flags.portSelectRequired || !flags.connected && !flags.portSelectRequired"
            @click="flags.portSelectRequired ? selectPort() : start(true)"
            outline
            class="q-mt-md"
          >
            Connect
          </q-btn>
          <template v-else>
            <q-spinner
              color="primary"
              size="3em"
              class="q-mb-md"
            ></q-spinner>
            <p>Waiting for Flipper...</p>
          </template>
        </div>
        <div v-if="!flags.serialSupported" class="column text-center q-px-lg q-py-lg">
          <h5>Unsupported browser</h5>
          <p>
            Your browser doesn't support WebSerial API.
            For better experience we recommend using Chrome for desktop.<br />
            <a href="https://caniuse.com/web-serial" target="_blank">Full list of supported browsers</a>
          </p>
        </div>
      </q-page>
      <q-dialog v-model="flags.logsPopup">
        <q-card>
          <q-card-section class="row items-center q-pb-none">
            <div class="text-h6">Logs</div>
            <q-space />
            <q-btn icon="close" flat round dense v-close-popup />
          </q-card-section>

          <q-card-section>
            <p>You can report bugs <a href="https://forum.flipperzero.one/c/web-app/22" target="blank_">here</a>. Attached logs may be helpful.</p>
            <q-scroll-area
              style="height: 300px; min-width: 280px; width: calc(min(80vw, 500px));"
              class="bg-grey-12 q-px-sm q-py-xs rounded-borders"
            >
              <code>
                <span v-if="!history.length">Logs will appear here...</span>
                <span v-for="line in history" :key="line.timestamp">
                  {{ `${line.time.padEnd(8)} [${line.level.toUpperCase()}] ${line.message}` }}
                  <br />
                </span>
              </code>
            </q-scroll-area>
          </q-card-section>

          <q-card-section align="right" class="q-pt-none">
            <q-btn
              flat
              label="Download"
              @click="downloadLogs"
            ></q-btn>
          </q-card-section>
        </q-card>
      </q-dialog>

      <q-dialog v-model="flags.flipperOccupiedDialog">
        <q-card class="dialog">
          <q-btn icon="close" flat round dense v-close-popup class="dialog-close-btn"/>

          <q-card-section class="q-pa-none q-ma-md" align="center">
            <q-icon name="mdi-alert-circle" color="primary" size="64px" />
            <div class="text-h6 q-my-sm">Can't connect to Flipper</div>
            <p>It seems that the serial port is occupied. Close any other app (e.g. qFlipper) connected to Flipper and try again.</p>
          </q-card-section>

          <q-card-section class="q-pt-none" align="center">
            <q-btn
              outline
              color="primary"
              label="Try again"
              @click="start"
            ></q-btn>
          </q-card-section>
        </q-card>
      </q-dialog>

      <q-dialog v-model="flags.microSDcardMissingDialog">
        <q-card class="dialog">
          <q-btn icon="close" flat round dense v-close-popup class="dialog-close-btn"/>

          <q-card-section class="q-pa-none q-ma-md" align="center">
            <q-icon name="mdi-alert-circle" color="primary" size="64px" />
            <div class="text-h6 q-my-sm">MicroSD card not detected</div>
            <p>It seems that the MicroSD card is not mounted or damaged. Insert the microSD card into the slot and try again.</p>
          </q-card-section>

          <q-card-section class="q-pt-none" align="center">
            <q-btn
              outline
              color="primary"
              label="Instruction manual"
              href="https://docs.flipper.net/basics/sd-card#Hjdbt"
              target="_blank"
            />
          </q-card-section>
        </q-card>
      </q-dialog>
      <q-dialog v-model="flags.serialUnsupportedDialog">
        <q-card class="dialog">
          <q-btn icon="close" flat round dense v-close-popup class="dialog-close-btn"/>

          <q-card-section class="q-pa-none q-ma-md" align="center">
            <q-icon name="mdi-alert-circle" color="primary" size="64px" />
            <div class="text-h6 q-my-sm">Unsupported browser</div>
            <p>
              Your browser doesn't support WebSerial API.
              For better experience we recommend using Chrome for desktop.<br />
            </p>
          </q-card-section>
          <q-card-section class="q-pt-none" align="center">
            <q-btn
              outline
              color="primary"
              label="Full list of supported browsers"
              href="https://caniuse.com/web-serial"
              target="_blank"
            ></q-btn>
          </q-card-section>
        </q-card>
      </q-dialog>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useQuasar } from 'quasar'
import { useRoute, useRouter } from 'vue-router'
import ExternalLink from 'components/ExternalLink.vue'
import RouterLink from 'components/RouterLink.vue'
import { logger, history, log } from 'composables/useLog'
import showNotif from 'composables/useShowNotif'

import { useMainStore } from 'src/stores/main'
const mainStore = useMainStore()

const flags = computed(() => mainStore.flags)
const flipper = computed(() => mainStore.flipper)
const info = computed(() => mainStore.info)

const $q = useQuasar()
$q.screen.setSizes({ md: 900 })

const router = useRouter()
const route = useRoute()

const componentName = 'Main'

const routes = [
  {
    title: 'My Flipper',
    icon: 'svguse:common-icons.svg#device',
    link: '/'
  },
  {
    title: 'Apps',
    icon: 'svguse:common-icons.svg#apps',
    link: '/apps'
  },
  {
    title: 'Files',
    icon: 'svguse:common-icons.svg#files',
    link: '/archive'
  },
  {
    title: 'CLI',
    icon: 'svguse:common-icons.svg#cli',
    link: '/cli'
  },
  {
    title: 'NFC tools',
    icon: 'svguse:common-icons.svg#nfctools',
    link: '/nfc-tools'
  },
  {
    title: 'Paint',
    icon: 'svguse:common-icons.svg#paint',
    link: '/paint'
  },
  {
    title: 'Pulse Plotter',
    icon: 'svguse:common-icons.svg#subtools',
    link: '/pulse-plotter'
  }
]
const extLinks = [
  {
    title: 'Home',
    icon: 'mdi-home-outline',
    link: 'https://flipperzero.one/'
  },
  {
    title: 'Shop',
    icon: 'mdi-cart-outline',
    link: 'https://shop.flipperzero.one/'
  },
  {
    title: 'Docs',
    icon: 'mdi-book-open-variant',
    link: 'https://docs.flipperzero.one/'
  },
  {
    title: 'Blog',
    icon: 'mdi-newspaper-variant-outline',
    link: 'https://blog.flipperzero.one/'
  },
  {
    title: 'Forum',
    icon: 'mdi-forum-outline',
    link: 'https://forum.flipperzero.one/'
  }
]
const canLoadWithoutFlipper = [
  'remote-cli',
  'pulse-plotter',
  'apps'
]

const leftDrawer = ref(true)
const linksMenu = ref(false)
const reconnectLoop = ref(null)
const connectionStatus = ref('Ready to connect')

const batteryIcon = computed(() => {
  const roundedCharge = Math.round(Number(info.value?.power.charge.level) / 10) * 10
  if (roundedCharge === 0) {
    return 'mdi-battery-outline'
  } else if (roundedCharge === 100) {
    return 'mdi-battery'
  }
  return 'mdi-battery-' + roundedCharge
})
const batteryColor = computed(() => {
  const charge = Number(info.value?.power.charge.level)
  if (charge >= 75) {
    return 'positive'
  } else if (charge >= 30) {
    return 'warning'
  }
  return 'negative'
})
const sdCardUsed = computed(() => {
  if (info.value?.storage.sdcard.freeSpace) {
    return 100 - Math.floor(info.value?.storage.sdcard.freeSpace / (info.value?.storage.sdcard.totalSpace / 100))
  } else {
    return 1
  }
})
const internalUsed = computed(() => {
  if (info.value?.storage.internal.freeSpace) {
    return 100 - Math.floor(info.value?.storage.internal.freeSpace / (info.value?.storage.internal.totalSpace / 100))
  } else {
    return 1
  }
})

watch(route, (to) => {
  checkConnectionRequirement(to.path)
})

const selectPort = async () => {
  const filters = [
    { usbVendorId: 0x0483, usbProductId: 0x5740 }
  ]
  await navigator.serial.requestPort({ filters })
  return start(true)
}
const disconnect = () => {
  flipper.value.disconnect()
    .then(() => {
      connectionStatus.value = 'Disconnected'
      mainStore.toggleFlag('connected', false)
      mainStore.setInfo(null)
      log({
        level: 'info',
        message: `${componentName}: Flipper disconnected`
      })
    })
    .catch(error => {
      log({
        level: 'error',
        message: `${componentName}: Error while disconnecting ${error}`
      })
      connectionStatus.value = error.toString()
    })
}

const findKnownDevices = () => {
  const filters = [
    { usbVendorId: 0x0483, usbProductId: 0x5740 }
  ]
  return navigator.serial.getPorts({ filters })
}

const autoReconnect = () => {
  if (reconnectLoop.value) {
    clearInterval(reconnectLoop.value)
    reconnectLoop.value = null
  }
  if (flags.value.autoReconnect) {
    reconnectLoop.value = setInterval(async () => {
      if (flags.value.autoReconnect) {
        const ports = await findKnownDevices()
        if (ports && ports.length > 0) {
          clearInterval(reconnectLoop.value)
          reconnectLoop.value = null
          return await start()
        }
      } else {
        clearInterval(reconnectLoop.value)
        reconnectLoop.value = null
      }
    }, 3000)
  }
}

const toggleConnectOnStart = () => {
  localStorage.setItem('connectOnStart', flags.value.connectOnStart)
}
const toggleAutoReconnect = () => {
  localStorage.setItem('autoReconnect', flags.value.autoReconnect)
}
const toggleInstallFromFile = () => {
  localStorage.setItem('installFromFile', flags.value.installFromFile)
}
const toggleCatalogEnabled = () => {
  localStorage.setItem('catalogEnabled', flags.value.catalogEnabled)
  location.reload()
}
const toggleCatalogChannel = () => {
  localStorage.setItem('catalogChannel', flags.value.catalogChannelProduction ? 'production' : 'dev')
  location.reload()
}

const checkConnectionRequirement = (path) => {
  mainStore.toggleFlag('connectionRequired', true)
  for (const link of canLoadWithoutFlipper) {
    if ((path && path.includes(link)) || route.path.includes(link)) {
      mainStore.toggleFlag('connectionRequired', false)
      break
    }
  }

  if (flags.value.catalogEnabled !== true) {
    routes.value = routes.filter(e => e.link !== '/apps')
    if (location.pathname === '/apps') {
      router.push({ name: 'Device' })
    }
  }
}

const downloadLogs = () => {
  let text = ''
  for (const line of history) {
    text += `${line.time} [${line.level}] ${line.message}\n`
  }
  const dl = document.createElement('a')
  dl.setAttribute('download', 'logs.txt')
  dl.setAttribute('href', 'data:text/plain,' + text)
  dl.style.visibility = 'hidden'
  document.body.append(dl)
  dl.click()
  dl.remove()
}

const start = async (manual) => {
  await mainStore.start(manual)
}

onMounted(async () => {
  if ($q.screen.lt.md) {
    leftDrawer.value = false
  }
  if ('serial' in navigator) {
    if (localStorage.getItem('connectOnStart') !== 'false') {
      mainStore.toggleFlag('connectOnStart', true)
      if (flags.value.connectionRequired) {
        await start()
      }
    } else {
      mainStore.toggleFlag('connectOnStart', false)
    }
    if (localStorage.getItem('autoReconnect') !== 'false') {
      mainStore.toggleFlag('autoReconnect', true)
    }
    if (localStorage.getItem('installFromFile') === 'true') {
      mainStore.toggleFlag('installFromFile', true)
    }

    const isProd = process.env.PRODUCTION
    const savedChannel = localStorage.getItem('catalogChannel')
    if (isProd) {
      localStorage.setItem('catalogChannel', 'production')
    } else {
      if (savedChannel === 'production') {
        mainStore.toggleFlag('catalogChannelProduction', true)
      } else {
        mainStore.toggleFlag('catalogChannelProduction', false)
      }

      if (!savedChannel) {
        localStorage.setItem('catalogChannel', 'dev')
        mainStore.toggleFlag('catalogChannelProduction', false)
      }

      mainStore.toggleFlag('catalogCanSwitchChannel', true)
    }

    navigator.serial.addEventListener('disconnect', e => {
      autoReconnect()
    })
  } else {
    mainStore.toggleFlag('serialSupported', false)
  }
  checkConnectionRequirement()

  navigator.serial.addEventListener('disconnect', (e) => {
    if (!flags.value.updateInProgress) {
      showNotif({
        message: 'Flipper has been disconnected'
      })
      mainStore.toggleFlag('connected', false)
      mainStore.toggleFlag('portSelectRequired', true)
    }
    log({
      level: 'info',
      message: `${componentName}: Flipper has been disconnected`
    })
  })

  logger.setLevel('info', true)
  const originalFactory = logger.methodFactory
  logger.methodFactory = function (methodName, logLevel, loggerName) {
    const rawMethod = originalFactory(methodName, logLevel, loggerName)

    return function (message) {
      if (methodName !== 'debug') {
        rawMethod(message)
      }
    }
  }
  logger.setLevel(logger.getLevel())
})
</script>
