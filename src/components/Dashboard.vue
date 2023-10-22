<script setup lang="ts">
import { onMounted, reactive } from "vue"
import { useMQTT } from "mqtt-vue-hook"
const mqttHook = useMQTT()
const protocol = "ws"
const host = "broker.hivemq.com"
const port = 8000

interface IWiFi {
  ssid: string
  level: number
}
interface IWiFiInfo {
  data: {
    wifi: IWiFi
  }
}

const farm = reactive({
  isConnected: false,
  humidity: 0,
  temperature: 0,
  wifiInfo: {
    ssid: "",
    level: 0,
  },
  isPumpOn: false,
})

const topic = (topicName: string): string => {
  const prefix: string = "IAM_YORK/"
  return `${prefix}${topicName}`
}

const onBoardConnected = (_topic: string, _message: string) => {}
const onTemperature = () => {
  mqttHook.subscribe([topic("sensor/humidity/get")], 1)
  mqttHook.registerEvent(topic("sensor/humidity/get"), (_topic: string, _message: string) => {
    farm.humidity = +_message
  })
}
const onHumidity = () => {
  mqttHook.subscribe([topic("sensor/temperature/get")], 1)
  mqttHook.registerEvent(topic("sensor/temperature/get"), (_topic: string, _message: string) => {
    farm.temperature = +_message
  })
}
const onWiFiInfo = () => {
  mqttHook.subscribe([topic("wifi/info")], 1)
  mqttHook.registerEvent(topic("wifi/info"), (_topic: string, _message: string) => {
    const uint8array = new TextEncoder().encode(_message)
    const payload = new TextDecoder().decode(uint8array)
    const WiFiInfo: IWiFiInfo = JSON.parse(payload)
    farm.wifiInfo = { ...WiFiInfo?.data?.wifi }
  })
}
const onRelayOpen = () => {
  mqttHook.subscribe([topic("relay/status/on")], 1)
  mqttHook.registerEvent(topic("relay/status/on"), (_topic: string, _message: string) => {
    const uint8array = new TextEncoder().encode(_message)
    const payload = new TextDecoder().decode(uint8array)
    switch (payload) {
      case "PUMP":
        farm.isPumpOn = true
        break

      default:
        break
    }
  })
}
const onRelayOff = () => {
  mqttHook.subscribe([topic("relay/status/off")], 1)
  mqttHook.registerEvent(topic("relay/status/off"), (_topic: string, _message: string) => {
    const uint8array = new TextEncoder().encode(_message)
    const payload = new TextDecoder().decode(uint8array)
    switch (payload) {
      case "PUMP":
        farm.isPumpOn = false
        break

      default:
        break
    }
  })
}

const handleTogglePump = () => {
  mqttHook.publish(topic(`relay/${farm.isPumpOn ? "off" : "on"}`), "PUMP")
}

const onSubscribe = () => {
  mqttHook.subscribe([topic("status/connected")], 1)
  mqttHook.registerEvent(topic("status/connected"), onBoardConnected)
  onHumidity()
  onTemperature()
  onWiFiInfo()
  onRelayOpen()
  onRelayOff()
}
const reconnect = () => {
  if (!mqttHook.isConnected()) {
    mqttHook.connect(`${protocol}://${host}:${port}/mqtt`, {
      clean: false,
      keepalive: 60,
      clientId: `mqtt_client_${Math.random().toString(16).substring(2, 10)}`,
      connectTimeout: 4000,
    })
  }
}
onMounted(async () => {
  reconnect()
  onSubscribe()
  // setInterval(() => {
  //   mqttHook.publish(topic("status/is_connected"), "is_connected")
  // }, 1000)
  // onSubscribe()
  // console.log("555")
})
</script>

<template>
  <section class="container mx-auto">
    <div
      class="flex flex-col justify-between px-5 text-bold title text-white bg-green-900 rounded-b-3xl"
    >
      <div class="pt-12">
        <div class="py-2 text-2xl">
          <i class="fas fa-temperature-low"></i>
          <span class="">{{ farm.temperature.toFixed(1) }}°C</span>
        </div>
        <div class="py-2 text-2xl">
          <i class="fas fa-tint"></i>
          <span class="">{{ farm.humidity.toFixed(1) }}%</span>
        </div>
        <div class="py-2 text-xl flex items-center">
          <div
            class="signal-icon"
            :class="{
              weak: farm.wifiInfo.level > 0,
              medium: farm.wifiInfo.level > 33,
              strong: farm.wifiInfo.level > 66,
            }"
          >
            <div class="signal-bar"></div>
            <div class="signal-bar"></div>
            <div class="signal-bar"></div>
          </div>
          <span class="ml-4 text-xl">{{ farm.wifiInfo.ssid || "กำลังเชื่อมต่อ..." }}</span>
        </div>
        <div class="py-2 text-xl">
          <i class="fas fa-shower"></i>
          <span class="text-xl">{{ farm.isPumpOn ? "กำลังรดน้ำ" : "รอรดน้ำ" }}</span>
        </div>
      </div>
      <p class="uppercase text-2xl pb-10">โรงเพาะเห็ด</p>
    </div>
  </section>
  <section class="container mx-auto my-5">
    <div class="grid grid-cols-2 gap-4 px-5">
      <div
        :class="{ 'text-green-700': !farm.isPumpOn, 'text-red-700': farm.isPumpOn }"
        class="flex flex-col items-center border rounded bg-white control py-8"
        @click="handleTogglePump"
      >
        <i class="fas fa-power-off text-4xl"></i>
        <p class="mt-3 text-lg">{{ !farm.isPumpOn ? "เปิด" : "ปิด" }}ปั้มน้ำ</p>
      </div>
    </div>
  </section>
</template>

<style lang="scss" scoped>
.title {
  height: 50vh;
  box-shadow: rgba(0, 0, 0, 0.19) 0px 10px 20px, rgba(0, 0, 0, 0.23) 0px 6px 6px;

  i {
    @apply mr-3 font-bold w-5;
  }
}
.control {
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
}

.signal-icon {
  height: 18px;
  width: 18px;
  /* To show you the power of flexbox! */
  display: flex;
  /* Bars should be placed left to right */
  flex-direction: row;
  /* Evenly space the bars with space in between */
  justify-content: space-between;
  /* Sink the bars to the bottom, so they go up in steps */
  align-items: baseline;
}
.signal-icon .signal-bar {
  /* 4 + 3 + 4 + 3 + 4 = 18px (as set above)
     4px per bar and 3px margins between */
  width: 4px;
  /* All bars faded by default */
  opacity: 30%;
  /* Choose a color */
  @apply bg-white;
}

/* 3 different heights for 3 different bars */
.signal-icon .signal-bar:nth-child(1) {
  height: 40%;
}
.signal-icon .signal-bar:nth-child(2) {
  height: 70%;
}
.signal-icon .signal-bar:nth-child(3) {
  height: 100%;
}

/* Emphasize different bars depending on
   weak/medium/strong classes */
.signal-icon.weak .signal-bar:nth-child(1),
.signal-icon.medium .signal-bar:nth-child(1),
.signal-icon.medium .signal-bar:nth-child(2),
.signal-icon.strong .signal-bar:nth-child(1),
.signal-icon.strong .signal-bar:nth-child(2),
.signal-icon.strong .signal-bar:nth-child(3) {
  opacity: 100%;
}
</style>
