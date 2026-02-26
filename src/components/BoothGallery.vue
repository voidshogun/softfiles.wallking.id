<template>
  <!-- Loading State -->
  <div v-if="loading" class="min-h-screen flex items-center justify-center">
    <div class="text-center">
      <div class="relative w-16 h-16 mx-auto mb-6">
        <div class="absolute inset-0 rounded-full border-2 border-brand-200"></div>
        <div class="absolute inset-0 rounded-full border-2 border-transparent border-t-brand-500 animate-spin"></div>
      </div>
      <p class="text-surface-700/60 text-sm tracking-wide uppercase">Memuat Memories Kamu…</p>
    </div>
  </div>

  <!-- Error State -->
  <div v-else-if="error || !sessionData" class="min-h-screen flex items-center justify-center px-4">
    <div class="text-center max-w-md">
      <div class="w-16 h-16 mx-auto mb-6 rounded-2xl bg-red-50 flex items-center justify-center">
        <svg class="w-8 h-8 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m9-.75a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 3.75h.008v.008H12v-.008Z" />
        </svg>
      </div>
      <h2 class="text-xl font-semibold text-surface-900 mb-2">Sesi tidak ditemukan</h2>
      <p class="text-surface-700/60 text-sm mb-4">Mungkin link kedaluwarsa atau terjadi masalah.</p>
      <a href="/" class="px-6 py-2.5 rounded-xl bg-brand-600 hover:bg-brand-700 text-white font-medium inline-block">Kembali</a>
    </div>
  </div>

  <!-- Main Content -->
  <div v-else class="min-h-screen pb-12">
    <!-- Header -->
    <header class="sticky top-0 z-50 backdrop-blur-xl bg-white/80 border-b border-surface-200/60 shadow-sm">
      <div class="max-w-3xl mx-auto px-4 py-4">
        <div class="flex items-center justify-between">
          <div class="flex items-center gap-3">
            <img
              src="https://s3.nevaobjects.id/wallking-website/assets/wallkingbooth/1740807126541-549920634-wallkingboothlogo%20%281%29.png"
              alt="Wallking Booth"
              class="h-8"
            />
          </div>
          <!-- Share button -->
          <button
            @click="shareSession"
            class="group flex items-center gap-2 px-4 py-2 rounded-xl bg-brand-600 hover:bg-brand-700 text-white text-sm font-medium transition-all duration-300 shadow-md shadow-brand-500/20 hover:shadow-lg hover:shadow-brand-500/30"
          >
            <svg v-if="copied" class="w-4 h-4 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="m4.5 12.75 6 6 9-13.5" />
            </svg>
            <svg v-else class="w-4 h-4 group-hover:-translate-y-0.5 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M7.217 10.907a2.25 2.25 0 1 0 0 2.186m0-2.186c.18.324.283.696.283 1.093s-.103.77-.283 1.093m0-2.186 9.566-5.314m-9.566 7.5 9.566 5.314m0 0a2.25 2.25 0 1 0 3.935 2.186 2.25 2.25 0 0 0-3.935-2.186Zm0-12.814a2.25 2.25 0 1 0 3.933-2.185 2.25 2.25 0 0 0-3.933 2.185Z" />
            </svg>
            <span class="hidden sm:inline">{{ copied ? 'Copied!' : 'Share' }}</span>
          </button>
        </div>
      </div>
    </header>

    <!-- Session Info -->
    <div class="max-w-3xl mx-auto px-4 mt-6">
      <div class="rounded-2xl bg-white border border-surface-200/60 p-5 shadow-sm">
        <p class="text-xs text-brand-500 font-semibold uppercase tracking-wider mb-3">Your Memories Softfiles</p>
        <div class="flex items-center gap-3">
          <div class="w-12 h-12 rounded-full bg-linear-to-br from-brand-400 to-brand-600 flex items-center justify-center text-white text-lg font-bold shadow-lg shadow-brand-600/30">
            {{ sessionData.branch?.charAt(0)?.toUpperCase() || 'B' }}
          </div>
          <div class="flex-1 min-w-0">
            <p class="text-xs text-surface-700/50">Location,</p>
            <h2 class="text-xl font-bold text-surface-900 truncate">{{ sessionData.branch || 'Photobooth Session' }}</h2>
            <div class="flex items-center gap-2 mt-0.5">
              <span class="text-xs text-surface-700/50">{{ sessionData?.media?.[0]?.created_at ? formatDate(sessionData.media[0].created_at) : 'Tanggal tidak tersedia' }}</span>
              <span class="text-xs text-surface-700/30">•</span>
              <span class="text-xs text-brand-600 font-medium">{{ individualPhotos.length }} foto</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Layout Photo Section -->
    <section v-if="layoutPhoto" class="max-w-3xl mx-auto px-4 mt-6">
      <div class="rounded-2xl overflow-hidden bg-white border border-surface-200/60 shadow-sm">
        <div class="relative group bg-surface-100 cursor-pointer aspect-auto overflow-hidden" @click="openViewer(layoutPhoto)">
          <img :src="layoutPhoto.url" class="w-full h-auto object-contain" />
          <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center backdrop-blur-[2px]">
            <div class="w-10 h-10 rounded-xl bg-white/20 backdrop-blur-md flex items-center justify-center text-white scale-90 group-hover:scale-100 transition-transform">
               <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3.75v4.5m0-4.5h4.5m-4.5 0L9 9M3.75 20.25v-4.5m0 4.5h4.5m-4.5 0L9 15M20.25 3.75h-4.5m4.5 0v4.5m0-4.5L15 9m5.25 11.25h-4.5m4.5 0v-4.5m0 4.5L15 15" />
               </svg>
            </div>
          </div>
        </div>
        <div class="flex items-center justify-between px-4 py-3 border-t border-surface-100">
          <p class="text-sm font-semibold text-surface-900">Print Layout</p>
          <a :href="getDownloadUrl(layoutPhoto.id)" class="shrink-0 flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-brand-50 hover:bg-brand-100 text-brand-600 text-xs font-medium transition-all duration-200 hover:scale-105 border border-brand-200/50">
            <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
            </svg>
            Download
          </a>
        </div>
      </div>
    </section>

    <!-- Session Video Section -->
    <section v-if="sessionVideo" class="max-w-3xl mx-auto px-4 mt-6">
      <div class="rounded-2xl overflow-hidden bg-white border border-surface-200/60 shadow-sm">
        <div class="relative bg-surface-100">
          <video
            class="w-full h-full object-cover"
            controls
            playsinline
            preload="metadata"
          >
            <source :src="sessionVideo.url" type="video/mp4">
          </video>
        </div>
        <div class="flex items-center justify-between px-4 py-3 border-t border-surface-100">
          <p class="text-sm font-semibold text-surface-900">Live Moment</p>
          <a :href="getDownloadUrl(sessionVideo.id)" class="shrink-0 flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-brand-50 hover:bg-brand-100 text-brand-600 text-xs font-medium transition-all duration-200 hover:scale-105 border border-brand-200/50">
            <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
            </svg>
            Download
          </a>
        </div>
      </div>
    </section>

    <!-- Individual Photos Section -->
    <section v-if="individualPhotos.length > 0" class="max-w-3xl mx-auto px-4 mt-8">
      <div class="flex items-center gap-2 mb-4">
        <svg class="w-5 h-5 text-brand-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
          <path stroke-linecap="round" stroke-linejoin="round" d="m2.25 15.75 5.159-5.159a2.25 2.25 0 0 1 3.182 0l5.159 5.159m-1.5-1.5 1.409-1.409a2.25 2.25 0 0 1 3.182 0l2.909 2.909M3.75 21h16.5A2.25 2.25 0 0 0 22.5 18.75V5.25A2.25 2.25 0 0 0 20.25 3H3.75A2.25 2.25 0 0 0 1.5 5.25v13.5A2.25 2.25 0 0 0 3.75 21Z" />
        </svg>
        <div class="flex flex-col">
          <h3 class="text-base font-semibold text-surface-900 leading-tight">Original Photos</h3>
          <p class="text-xs text-surface-700/50 mt-0.5">Tap foto untuk memperbesar</p>
        </div>
        <span class="ml-auto text-xs text-surface-700/40">{{ individualPhotos.length }} files</span>
      </div>
      <div class="grid grid-cols-1 gap-3">
        <div
          v-for="(photo, index) in individualPhotos"
          :key="photo.id"
          class="group relative rounded-2xl overflow-hidden bg-white border border-surface-200/60 hover:border-brand-300 transition-all duration-300 shadow-sm hover:shadow-md"
        >
          <!-- Thumbnail -->
          <div class="relative overflow-hidden cursor-pointer" @click="openViewer(photo)">
            <img
              :src="photo.url"
              :alt="'Photo ' + (Number(index) + 1)"
              class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
              loading="lazy"
            />
            <div class="absolute inset-0 bg-linear-to-t from-black/30 via-transparent to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
            <!-- Expand icon on hover -->
            <div class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
              <div class="w-8 h-8 rounded-lg bg-white/70 backdrop-blur-sm flex items-center justify-center shadow-sm">
                <svg class="w-4 h-4 text-surface-800" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3.75v4.5m0-4.5h4.5m-4.5 0L9 9M3.75 20.25v-4.5m0 4.5h4.5m-4.5 0L9 15M20.25 3.75h-4.5m4.5 0v4.5m0-4.5L15 9m5.25 11.25h-4.5m4.5 0v-4.5m0 4.5L15 15" />
                </svg>
              </div>
            </div>
          </div>
          <!-- Download bar -->
          <div class="flex items-center justify-between px-3 py-2.5">
            <p class="text-xs text-surface-700/50 truncate mr-2">Photo {{ Number(index) + 1 }}</p>
            <a
              :href="getDownloadUrl(photo.id)"
              class="shrink-0 flex items-center gap-1 px-2.5 py-1 rounded-lg bg-brand-50 hover:bg-brand-100 text-brand-600 text-[11px] font-medium transition-all duration-200 hover:scale-105 border border-brand-200/50"
              @click.stop
            >
              <svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
              </svg>
              Download
            </a>
          </div>
        </div>
      </div>
    </section>

    <!-- Review Section -->
    <section class="max-w-3xl mx-auto px-4 mt-12 mb-8">
      <div class="rounded-2xl bg-white border border-surface-200/60 shadow-md overflow-hidden">
        <div class="bg-surface-50 p-5 text-center border-b border-surface-200/60 relative overflow-hidden">
            <div class="absolute top-0 left-0 w-full h-full opacity-[0.03] pointer-events-none" style="background-image: url('https://www.transparenttextures.com/patterns/cubes.png');"></div>
            <h2 class="text-xl font-bold text-surface-900 relative z-10 mb-1">Review Wallking Booth!</h2>
            <p class="text-surface-700/60 text-sm relative z-10">Bantu kami jadi lebih baik lagi ya!</p>
        </div>

        <div v-if="reviewSuccess" class="p-10 text-center transition-all duration-500">
            <div class="w-20 h-20 rounded-full flex items-center justify-center mx-auto mb-5" :class="isPreviouslySubmitted ? 'bg-brand-50' : 'bg-green-50'">
                <svg v-if="isPreviouslySubmitted" class="w-10 h-10 text-brand-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M6.75 3v2.25M17.25 3v2.25M3 18.75V7.5a2.25 2.25 0 0 1 2.25-2.25h13.5A2.25 2.25 0 0 1 21 7.5v11.25m-18 0A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75m-18 0v-7.5A2.25 2.25 0 0 1 5.25 9h13.5A2.25 2.25 0 0 1 21 11.25v7.5" />
                </svg>
                <svg v-else class="w-10 h-10 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M9 12.75 11.25 15 15 9.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
                </svg>
            </div>
            <h3 class="text-xl font-bold text-surface-900 mb-2">
                {{ isPreviouslySubmitted ? 'Kamu sudah mengisi review' : 'Terima Kasih!' }}
            </h3>
            <p class="text-surface-700/60 text-sm max-w-sm mx-auto leading-relaxed">
                {{ isPreviouslySubmitted 
                ? 'Terima kasih! Masukan kamu sebelumnya sudah kami terima dan sangat membantu kami.' 
                : 'Review kamu sangat berarti buat kami agar bisa terus berkembang.' 
                }}
            </p>
        </div>

        <form v-else @submit.prevent="submitReview" class="p-6 md:p-8 space-y-6">
            <div class="text-center">
                <label class="block text-xs font-bold text-surface-700/50 uppercase tracking-wide mb-3">Rate Experience Kamu</label>
                <div class="flex justify-center gap-2">
                    <button 
                        v-for="star in 5" 
                        :key="star" 
                        type="button"
                        @click="form.rating = star"
                        class="transition-all hover:scale-110 focus:outline-none p-1"
                    >
                        <svg 
                           class="w-10 h-10 transition-colors duration-200" 
                           :class="star <= form.rating ? 'fill-brand-400 text-brand-400 drop-shadow-sm' : 'text-surface-200 fill-transparent'"
                           viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                           <path stroke-linecap="round" stroke-linejoin="round" d="M11.48 3.499a.562.562 0 0 1 1.04 0l2.125 5.111a.563.563 0 0 0 .475.345l5.518.442c.499.04.701.663.321.988l-4.204 3.602a.563.563 0 0 0-.182.557l1.285 5.385a.562.562 0 0 1-.84.61l-4.725-2.885a.562.562 0 0 0-.586 0L6.982 20.54a.562.562 0 0 1-.84-.61l1.285-5.386a.562.562 0 0 0-.182-.557l-4.204-3.602a.562.562 0 0 1 .321-.988l5.518-.442a.563.563 0 0 0 .475-.345L11.48 3.5Z" />
                        </svg>
                    </button>
                </div>
            </div>

            <div class="h-px bg-surface-100 w-full"></div>

            <div class="space-y-5">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                    <div>
                        <label class="block text-sm font-semibold text-surface-800 mb-1.5">Nama Lengkap <span class="text-brand-500">*</span></label>
                        <input v-model="form.fullName" type="text" required class="w-full bg-surface-50 border border-surface-200/80 rounded-xl px-4 py-3 text-surface-900 placeholder-surface-400 text-sm focus:border-brand-500 focus:ring-2 focus:ring-brand-500/20 active:outline-none focus:outline-none transition-all" placeholder="Nama kamu" />
                    </div>
                    <div>
                        <label class="block text-sm font-semibold text-surface-800 mb-1.5">No. WhatsApp <span class="text-brand-500">*</span></label>
                        <input v-model="form.whatsapp" type="tel" required class="w-full bg-surface-50 border border-surface-200/80 rounded-xl px-4 py-3 text-surface-900 placeholder-surface-400 text-sm focus:border-brand-500 focus:ring-2 focus:ring-brand-500/20 active:outline-none focus:outline-none transition-all" placeholder="08xxxxxxxx" />
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                    <div>
                        <label class="block text-sm font-semibold text-surface-800 mb-1.5">Alamat Email <span class="text-brand-500">*</span></label>
                        <input v-model="form.email" type="email" required class="w-full bg-surface-50 border border-surface-200/80 rounded-xl px-4 py-3 text-surface-900 placeholder-surface-400 text-sm focus:border-brand-500 focus:ring-2 focus:ring-brand-500/20 active:outline-none focus:outline-none transition-all" placeholder="email@contoh.com" />
                    </div>
                    <div>
                        <label class="block text-sm font-semibold text-surface-800 mb-1.5">Gender <span class="text-brand-500">*</span></label>
                        <div class="relative">
                            <select v-model="form.gender" required class="w-full bg-surface-50 border border-surface-200/80 rounded-xl px-4 py-3 text-surface-900 text-sm appearance-none focus:border-brand-500 focus:ring-2 focus:ring-brand-500/20 active:outline-none focus:outline-none transition-all">
                                <option value="" disabled>Pilih Gender</option>
                                <option value="Laki-laki">Laki-laki</option>
                                <option value="Perempuan">Perempuan</option>
                            </select>
                            <div class="absolute inset-y-0 right-0 flex items-center px-4 pointer-events-none text-surface-400">
                                <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                  <path stroke-linecap="round" stroke-linejoin="round" d="m19.5 8.25-7.5 7.5-7.5-7.5" />
                                </svg>
                            </div>
                        </div>
                    </div>
                </div>

                <div>
                    <label class="block text-sm font-semibold text-surface-800 mb-2">Tau Wallking Booth dari mana? <span class="text-brand-500">*</span></label>
                    <div class="grid grid-cols-2 md:grid-cols-3 gap-2">
                        <label 
                            v-for="opt in sourceOptions" 
                            :key="opt" 
                            class="flex items-center gap-2.5 p-2.5 border rounded-xl cursor-pointer transition-all hover:bg-brand-50/50 hover:border-brand-200"
                            :class="form.sources.includes(opt) ? 'border-brand-500 bg-brand-50/50 ring-1 ring-brand-500/30' : 'border-surface-200/80 bg-white'"
                        >
                            <input type="checkbox" :value="opt" v-model="form.sources" class="w-4 h-4 text-brand-500 border-surface-300 rounded focus:ring-brand-500" />
                            <span class="text-sm font-medium" :class="form.sources.includes(opt) ? 'text-brand-700' : 'text-surface-700'">{{ opt }}</span>
                        </label>
                    </div>
                </div>

                <div>
                    <label class="block text-sm font-semibold text-surface-800 mb-1.5">Review / Saran (Optional)</label>
                    <textarea v-model="form.reviewText" rows="3" class="w-full bg-surface-50 border border-surface-200/80 rounded-xl px-4 py-3 text-surface-900 placeholder-surface-400 text-sm resize-none focus:border-brand-500 focus:ring-2 focus:ring-brand-500/20 active:outline-none focus:outline-none transition-all" placeholder="Ceritain pengalaman seru kamu..."></textarea>
                </div>

                <div class="bg-brand-50/30 p-4 rounded-xl border border-brand-100/50">
                    <label class="block text-sm font-medium text-surface-800 mb-2">Izinkan foto digunakan untuk konten?</label>
                    <div class="flex gap-5">
                        <label class="flex items-center gap-2 cursor-pointer group">
                            <input type="radio" v-model="form.permission" value="yes" class="w-4 h-4 border-surface-300 text-brand-500 focus:ring-brand-500" />
                            <span class="text-sm text-surface-700">Ya, Boleh banget</span>
                        </label>
                        <label class="flex items-center gap-2 cursor-pointer group">
                            <input type="radio" v-model="form.permission" value="no" class="w-4 h-4 border-surface-300 text-brand-500 focus:ring-brand-500" />
                            <span class="text-sm text-surface-700">Jangan dulu deh</span>
                        </label>
                    </div>
                </div>
            </div>

            <button 
                type="submit" 
                :disabled="isSubmitting"
                class="w-full bg-brand-600 hover:bg-brand-700 text-white font-semibold py-3.5 rounded-xl transition-all shadow-md shadow-brand-500/20 hover:shadow-lg active:scale-[0.98] disabled:opacity-70 disabled:cursor-not-allowed flex items-center justify-center gap-2"
            >
                <span v-if="isSubmitting" class="flex items-center gap-2">
                    <div class="animate-spin rounded-full h-4 w-4 border-b-2 border-white"></div>
                    Mengirim...
                </span>
                <span v-else class="flex items-center gap-2">
                    Kirim Review 
                    <svg class="w-4 h-4 mb-0.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M6 12 3.269 3.125A59.769 59.769 0 0 1 21.485 12 59.768 59.768 0 0 1 3.27 20.875L5.999 12Zm0 0h7.5" />
                    </svg>
                </span>
            </button>
        </form>
      </div>
    </section>

    <!-- Footer -->
    <footer class="max-w-3xl mx-auto px-4 mt-8 text-center">
      <div class="h-px bg-linear-to-r from-transparent via-surface-200 to-transparent mb-6"></div>
      <p class="text-xs text-surface-700/40">2025 © Powered by <span class="text-brand-500 font-medium">Wallking Labs</span></p>
    </footer>

    <!-- Viewer Lightbox -->
    <Teleport to="body">
      <Transition name="lightbox">
        <div
          v-if="isViewerOpen"
          class="fixed inset-0 z-100 bg-black/90 backdrop-blur-lg flex items-center justify-center"
          @click.self="closeViewer"
        >
          <!-- Close button -->
          <button
            @click="closeViewer"
            class="absolute top-4 right-4 z-10 w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors"
          >
            <svg class="w-5 h-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
            </svg>
          </button>

          <!-- Top center counter -->
          <div class="absolute top-6 left-1/2 -translate-x-1/2 z-10">
              <span class="text-white/80 font-medium text-sm drop-shadow-md">
                 {{ activeIndex + 1 }} / {{ viewerList.length }}
             </span>
          </div>

          <!-- Navigation -->
          <button
            @click="prevMedia"
            class="absolute left-3 md:left-6 z-10 w-12 h-12 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors hover:scale-105"
          >
            <svg class="w-6 h-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5 8.25 12l7.5-7.5" />
            </svg>
          </button>
          <button
            @click="nextMedia"
            class="absolute right-3 md:right-6 z-10 w-12 h-12 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors hover:scale-105"
          >
            <svg class="w-6 h-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="m8.25 4.5 7.5 7.5-7.5 7.5" />
            </svg>
          </button>

          <!-- Image -->
          <img
            :key="activeMedia?.id"
            :src="activeMedia?.url"
            class="max-h-[85vh] max-w-[90vw] object-contain rounded-lg shadow-2xl transition-all duration-300"
          />

          <!-- Bottom Action -->
          <div class="absolute bottom-6 left-1/2 -translate-x-1/2 flex justify-center w-full px-4">
             <a
               :href="getDownloadUrl(activeMedia?.id)"
               class="flex items-center gap-2 px-6 py-3 rounded-xl bg-white text-surface-900 text-sm font-bold shadow-lg transition-transform hover:scale-105"
             >
                <svg class="w-4.5 h-4.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
                </svg>
                Download Photo
             </a>
          </div>

        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, reactive } from 'vue';

const props = defineProps<{
  sessionId: string;
}>();

const loading = ref(true);
const error = ref<string | null>(null);
const sessionData = ref<any>(null);

const API_BASE = 'https://booth-api.wallking.id';

// Fetch Data
onMounted(async () => {
  try {
    const res = await fetch(`${API_BASE}/session/${props.sessionId}`);
    if (!res.ok) throw new Error('Sesi tidak ditemukan atau sudah kadaluarsa.');
    sessionData.value = await res.json();
  } catch (e: any) {
    error.value = e.message || 'Failed to load session.';
  } finally {
    loading.value = false;
  }
});

// Computeds for Media
const layoutPhoto = computed(() =>
  sessionData.value?.media.find((m: any) =>
    m.key.includes('layout-') && m.key.match(/\.(jpg|jpeg|png|webp)$/i)
  )
);

const individualPhotos = computed(() =>
  sessionData.value?.media.filter((m: any) => m.type === 'photo' && !m.key.includes('layout-')) || []
);

const sessionVideo = computed(() => sessionData.value?.media.find((m: any) => m.type === 'video'));

// Helpers
const getDownloadUrl = (id: number) => `${API_BASE}/download/${id}`;

const formatDate = (dateStr: string): string => {
  if (!dateStr) return '';
  const d = new Date(dateStr);
  return d.toLocaleDateString('id-ID', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    timeZone: 'Asia/Jakarta',
  });
};

// Share Logic
const copied = ref(false);
const shareSession = async () => {
  const shareData = {
    title: `${sessionData.value?.branch || 'Booth Session'} — Wallking Booth`,
    text: `Check out my memories at Wallking Booth!`,
    url: window.location.href,
  };

  if (navigator.share) {
    try {
      await navigator.share(shareData);
    } catch {
      // Ignored
    }
  } else {
    try {
      await navigator.clipboard.writeText(window.location.href);
      copied.value = true;
      setTimeout(() => { copied.value = false; }, 2000);
    } catch {
      const textArea = document.createElement('textarea');
      textArea.value = window.location.href;
      document.body.appendChild(textArea);
      textArea.select();
      document.execCommand('copy');
      document.body.removeChild(textArea);
      copied.value = true;
      setTimeout(() => { copied.value = false; }, 2000);
    }
  }
};

// Viewer Logic
const isViewerOpen = ref(false);
const activeMedia = ref<any>(null);

const openViewer = (media: any) => { 
  activeMedia.value = media;
  isViewerOpen.value = true;
  document.body.style.overflow = 'hidden';
};

const closeViewer = () => { 
  isViewerOpen.value = false;
  activeMedia.value = null;
  document.body.style.overflow = '';
};

const viewerList = computed(() => {
  const list = [];
  if (layoutPhoto.value) list.push(layoutPhoto.value);
  if (individualPhotos.value) list.push(...individualPhotos.value);
  return list;
});

const activeIndex = computed(() => {
  return viewerList.value.findIndex((m: any) => m.id === activeMedia.value?.id);
});

const nextMedia = () => {
  if (viewerList.value.length === 0) return;
  if (activeIndex.value < viewerList.value.length - 1) {
    activeMedia.value = viewerList.value[activeIndex.value + 1];
  } else {
    activeMedia.value = viewerList.value[0];
  }
};

const prevMedia = () => {
  if (viewerList.value.length === 0) return;
  if (activeIndex.value > 0) {
    activeMedia.value = viewerList.value[activeIndex.value - 1];
  } else {
    activeMedia.value = viewerList.value[viewerList.value.length - 1];
  }
};

const handleKeydown = (e: KeyboardEvent) => {
  if (!isViewerOpen.value) return;
  if (e.key === 'Escape') closeViewer();
  if (e.key === 'ArrowRight') nextMedia();
  if (e.key === 'ArrowLeft') prevMedia();
};

onMounted(() => {
  document.addEventListener('keydown', handleKeydown);
});

// Review State
const isSubmitting = ref(false);
const reviewSuccess = ref(false);
const isPreviouslySubmitted = ref(false);

const form = reactive({
  rating: 0,
  fullName: '',
  whatsapp: '',
  email: '',
  gender: '',
  sources: [] as string[],
  reviewText: '',
  permission: 'yes'
});
const sourceOptions = ['Teman', 'Instagram', 'TikTok', 'Google Search', 'Lainnya'];

onMounted(() => {
  if (typeof window !== 'undefined') {
    const hasReviewed = localStorage.getItem(`reviewed_${props.sessionId}`);
    if (hasReviewed) {
      isPreviouslySubmitted.value = true;
      reviewSuccess.value = true;
    }
  }
});

const submitReview = async () => {
  if (form.rating === 0) return alert('Mohon beri bintang rating (1-5)');
  if (!form.fullName || !form.whatsapp || !form.email || !form.gender) {
    return alert('Mohon lengkapi data diri yang wajib diisi');
  }

  isSubmitting.value = true;

  try {
    const res = await fetch(`${API_BASE}/review`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        sessionId: props.sessionId,
        branch: sessionData.value?.branch,
        ...form
      })
    });
    
    if (!res.ok) {
        if (res.status === 409) {
            isPreviouslySubmitted.value = true;
            reviewSuccess.value = true;
            localStorage.setItem(`reviewed_${props.sessionId}`, 'true');
        } else {
            throw new Error('Gagal mengirim review');
        }
    } else {
        localStorage.setItem(`reviewed_${props.sessionId}`, 'true');
        reviewSuccess.value = true;
    }
  } catch (err: any) {
    alert('Gagal mengirim review. Silakan coba lagi.');
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<style scoped>
.lightbox-enter-active,
.lightbox-leave-active {
  transition: opacity 0.3s ease;
}
.lightbox-enter-from,
.lightbox-leave-to {
  opacity: 0;
}
</style>
