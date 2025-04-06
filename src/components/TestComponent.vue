<template>
    <div class="p-6 max-w-md mx-auto text-center">
      <div class="text-xl font-bold mb-4">Havacılık Harf Testi</div>
  
      <button v-if="!isRunning && !showResult" @click="startTest" class="bg-blue-500 text-white px-4 py-2 rounded">
        Teste Başla
      </button>
  
      <div v-if="isRunning">
        <p class="text-lg mb-2">Şu anki harf: <strong class="uppercase">{{ currentLetter }}</strong></p>
        <audio :src="currentAudio" ref="audio" autoplay />
  
        <p class="text-sm text-gray-500 mb-2">Cevap süresi: {{ countdown }} saniye</p>
  
        <div class="flex justify-center gap-4 mt-2 mb-2">
          <button
            class="bg-white text-black border border-gray-400 px-4 py-2 rounded"
            :disabled="!canAnswer"
            @click="checkAnswer('vowel')"
          >
            ⚪ Beyaz Buton
          </button>
          <button
            class="bg-red-600 text-white px-4 py-2 rounded"
            :disabled="!canAnswer"
            @click="checkAnswer('consonant')"
          >
            🔴 Kırmızı Buton
          </button>
        </div>
  
        <p class="text-sm text-gray-600 mb-2">Harf geçmişi: {{ streak.map(l => l.toUpperCase()).join(', ') }}</p>
      </div>
  
      <div v-if="showResult" class="mt-6">
        <p class="text-xl font-bold mb-2">✅ Test Tamamlandı</p>
        <p>✅ Doğru: {{ score.correct }}</p>
        <p>❌ Yanlış: {{ score.wrong }}</p>
        <button @click="resetTest" class="mt-4 bg-blue-500 text-white px-4 py-2 rounded">Baştan Başla</button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  
  const fullLetterList = [
    'alfa', 'bravo', 'charlie', 'delta', 'echo', 'foxtrot', 'golf',
    'hotel', 'india', 'juliet', 'kilo', 'lima', 'mike', 'november',
    'oscar', 'papa', 'quebec', 'romeo', 'sierra', 'tango', 'uniform',
    'victor', 'whiskey', 'x-ray', 'yankee', 'zulu'
  ]
  
  const vowelSet = new Set(['alfa', 'echo', 'india', 'oscar', 'uniform'])
  
  const isRunning = ref(false)
  const showResult = ref(false)
  const currentIndex = ref(0)
  const currentLetter = ref('')
  const currentAudio = ref('')
  const streak = ref([])
  const score = ref({ correct: 0, wrong: 0 })
  const canAnswer = ref(false)
  const hasAnswered = ref(false)
  const letters = ref([])
  const countdown = ref(0)
  
  function shuffleArray(array) {
    return array.slice().sort(() => Math.random() - 0.5)
  }
  
  async function startTest() {
    isRunning.value = true
    showResult.value = false
    score.value = { correct: 0, wrong: 0 }
    streak.value = []
    letters.value = shuffleArray(fullLetterList).slice(0, 20)
    currentIndex.value = 0
    await playNext()
  }
  
  async function playNext() {
    if (currentIndex.value >= letters.value.length) {
      isRunning.value = false
      showResult.value = true
      return
    }
  
    const letter = letters.value[currentIndex.value]
    currentLetter.value = letter
    currentAudio.value = `/audio/${letter}.mp3`
    hasAnswered.value = false
    canAnswer.value = false
  
    streak.value.push(letter)
    if (streak.value.length > 3) streak.value.shift()
  
    await wait(1000)
  
    canAnswer.value = true
    countdown.value = 1
  
    const interval = setInterval(() => {
      countdown.value--
      if (countdown.value <= 0) {
        clearInterval(interval)
      }
    }, 1000)
  
    await wait(1000)
  
    canAnswer.value = false
    currentIndex.value++
    await wait(500)
    await playNext()
  }
  
  function checkAnswer(type) {
    if (!canAnswer.value || hasAnswered.value) return
  
    const lastThree = streak.value.slice(-3)
    const allVowel = lastThree.every(l => vowelSet.has(l))
    const allConsonant = lastThree.every(l => !vowelSet.has(l))
  
    const isCorrect =
      (type === 'vowel' && allVowel) || (type === 'consonant' && allConsonant)
  
    if (isCorrect) {
      score.value.correct++
    } else {
      score.value.wrong++
    }
  
    hasAnswered.value = true
    canAnswer.value = false
  }
  
  function resetTest() {
    isRunning.value = false
    showResult.value = false
    score.value = { correct: 0, wrong: 0 }
    streak.value = []
    letters.value = []
  }
  
  function wait(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
  </script>
  
  <style scoped>
  button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
  </style>
  