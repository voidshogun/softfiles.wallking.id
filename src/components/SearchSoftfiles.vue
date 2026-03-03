<template>
  <div class="min-h-screen bg-[#fafafa] flex flex-col justify-center py-12 sm:px-6 lg:px-8 font-['Inter']">
    <div class="sm:mx-auto sm:w-full sm:max-w-md flex flex-col items-center">
      <img src="https://s3.nevaobjects.id/wallking-website/uploads/wlkg-dark.svg" alt="Wallking Logo" class="h-10 w-auto mb-2" />
      <h2 class="mt-4 text-center text-3xl font-extrabold text-gray-900 tracking-tight">
        Find Your Softfiles
      </h2>
      <p class="mt-2 text-center text-sm text-gray-600">
        Pilih studio dan masukkan Session ID Anda
      </p>
    </div>

    <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
      <div class="bg-white py-8 px-4 shadow-xl shadow-gray-200/50 sm:rounded-2xl sm:px-10 border border-gray-100">
        <form class="space-y-6" @submit.prevent="handleSubmit">
          
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-3">Layanan Studio</label>
            <div class="grid grid-cols-3 gap-3">
              <label 
                v-for="srv in services" :key="srv.id"
                class="relative rounded-xl border p-4 flex flex-col items-center justify-center cursor-pointer transition-all duration-200 focus:outline-none"
                :class="service === srv.id ? 'border-black bg-black text-white shadow-md transform scale-[1.02]' : 'border-gray-200 bg-white text-gray-900 hover:border-gray-300 hover:bg-gray-50'"
              >
                <input type="radio" :value="srv.id" v-model="service" class="sr-only" />
                <component :is="srv.icon" class="w-6 h-6 mb-2" :class="service === srv.id ? 'text-white' : 'text-gray-500'" />
                <span class="text-sm font-semibold">{{ srv.label }}</span>
              </label>
            </div>
          </div>

          <div>
            <label for="sessionId" class="block text-sm font-medium text-gray-700">
              {{ service === 'selfphoto' ? 'Nama Koleksi' : 'Session ID' }}
            </label>
            <div class="mt-2 relative">
              <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <Search class="h-5 w-5 text-gray-400" />
              </div>
              <input
                id="sessionId"
                v-model="sessionId"
                type="text"
                required
                class="appearance-none block w-full pl-10 pr-3 py-3 border border-gray-300 rounded-xl shadow-sm placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-black focus:border-black sm:text-sm transition-colors bg-gray-50 focus:bg-white"
                :placeholder="service === 'selfphoto' ? 'Cth: NONA-495b' : 'Cth: b8b285c5-9cce...'"
              />
            </div>
          </div>

          <div>
            <button
              type="submit"
              :disabled="!service || !sessionId"
              class="w-full flex justify-center py-3 px-4 border border-transparent rounded-xl shadow-sm text-sm font-bold text-white bg-black hover:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-black disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-200 mt-2"
            >
              Buka Softfiles
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { Camera, Train, Store, Search, Image } from 'lucide-vue-next'

const services = [
  { id: 'selfphoto', label: 'Selfphoto', icon: Image },
  { id: 'elevator', label: 'Elevator', icon: Camera },
  { id: 'subway', label: 'Subway', icon: Train },
  { id: 'bakery', label: 'Bakery', icon: Store },
  { id: 'booth', label: 'Booth', icon: Store }
]

const service = ref('elevator')
const sessionId = ref('')

const handleSubmit = () => {
  if (service.value && sessionId.value) {
    const cleanSessionId = sessionId.value.trim()
    window.location.href = `/${service.value}/${cleanSessionId}`
  }
}
</script>
