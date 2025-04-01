<template>
  <div>
    <button @click="speakText" :disabled="isLoading">
      {{ isLoading ? 'Loading...' : '🔊 Speak' }}
    </button>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const props = defineProps({
  textToSpeak: String,
  voiceId: String,
  apiKey: String,
})

let client
const isLoading = ref(false)

onMounted(() => {
  if (window.ElevenLabs) {
    client = new window.ElevenLabs.ElevenLabsClient({ apiKey: props.apiKey })
  } else {
    console.error('ElevenLabs SDK not loaded')
  }
})

const speakText = async () => {
  if (!client) return alert('Client not ready.')
  if (!props.textToSpeak) return alert('No text provided.')

  isLoading.value = true
  try {
    const audio = await client.textToSpeech.convert({
      voiceId: props.voiceId,
      text: props.textToSpeak,
      modelId: 'eleven_monolingual_v1',
      outputFormat: 'mp3_44100_128',
    })

    const blob = new Blob([audio], { type: 'audio/mpeg' })
    const url = URL.createObjectURL(blob)
    const audioElement = new Audio(url)
    audioElement.play()
  } catch (err) {
    console.error(err)
    alert('Error converting text to speech.')
  } finally {
    isLoading.value = false
  }
}
</script>

<!-- Load the SDK from CDN -->
<script src="https://cdn.jsdelivr.net/npm/@elevenlabs/elevenlabs-js@latest/dist/index.umd.js"></script>

<style scoped>
button {
  padding: 8px 12px;
  font-size: 16px;
  border-radius: 6px;
  background-color: #6a0dad;
  color: white;
  border: none;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
