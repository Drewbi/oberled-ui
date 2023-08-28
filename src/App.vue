<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import TouchScreen from './components/TouchScreen.vue'
import StatusIndicator from './components/StatusIndicator.vue'

const socket = ref(null)
const data = ref(null)
const connected = ref(false)
const screenOnline = ref(false)

onMounted(() => {
    connect()
})

onUnmounted(() => {
    if (socket.value) {
        socket.value.close()
        connected.value = false
    }
})

const connect = () => {
    socket.value = new WebSocket('wss://oberled-socket.drub.workers.dev')

    socket.value.onopen = () => {
        console.log('Connection opened')
        connected.value = true
    }

    socket.value.onmessage = (event) => {
        if (event.data === "ping") {
            socket.value.send("pong")
        } else {
            const parsed = JSON.parse(event.data)
            if (!parsed) {
                console.error('Failed to parse message')
            }
            else if (parsed.error) {
                console.error('WebSocket Error: ', parsed.error)
            } else {
                if (parsed.positions) data.value = parsed.positions.filter((pos) => pos && pos.x !== undefined && pos.y !== undefined)
                if (parsed.screenOnline !== undefined) screenOnline.value = parsed.screenOnline
            }
        }

    }

    socket.value.onerror = (error) => {
        console.error('WebSocket Error: ', error)
        connected.value = false
    }
}

const send = (data) => {
    if (connected.value && socket.value && socket.value.readyState === WebSocket.OPEN) {
        socket.value.send(data)
    }
}
</script>

<template>
    <div id="container">
        <TouchScreen :data="data" @send-move="send($event)" />
        <div id="indicators">
            <StatusIndicator title="Server" :isConnected="connected" />
            <StatusIndicator title="Screen" :isConnected="screenOnline" />
        </div>
    </div>
</template>

<style scoped>
#container {
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    overflow: hidden;
}

#indicators {
    display: flex;
    gap: 20px;
}
</style>
