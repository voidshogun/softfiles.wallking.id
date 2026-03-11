<template>
  <!-- Loading State -->
  <div v-if="loading" class="min-h-screen flex items-center justify-center">
    <div class="text-center">
      <div class="relative w-16 h-16 mx-auto mb-6">
        <div class="absolute inset-0 rounded-full border-2 border-brand-200"></div>
        <div class="absolute inset-0 rounded-full border-2 border-transparent border-t-brand-500 animate-spin"></div>
      </div>
      <p class="text-surface-700/60 text-sm tracking-wide uppercase">Memuat Softfiles Kamu…</p>
    </div>
  </div>

  <!-- Error State -->
  <div v-else-if="error" class="min-h-screen flex items-center justify-center px-4">
    <div class="text-center max-w-md">
      <div class="w-16 h-16 mx-auto mb-6 rounded-2xl bg-red-50 flex items-center justify-center">
        <svg class="w-8 h-8 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m9-.75a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 3.75h.008v.008H12v-.008Z" />
        </svg>
      </div>
      <h2 class="text-xl font-semibold text-surface-900 mb-2">Sesi tidak ditemukan</h2>
      <p class="text-surface-700/60 text-sm">{{ error }}</p>
    </div>
  </div>

  <!-- Main Content -->
  <div v-else-if="session" class="min-h-screen pb-12">
    <!-- Header -->
    <header class="sticky top-0 z-50 backdrop-blur-xl bg-white/80 border-b border-surface-200/60 shadow-sm">
      <div class="max-w-3xl mx-auto px-4 py-4">
        <div class="flex items-center justify-between">
          <div class="flex items-center gap-3">
            <!-- Brand logo -->
            <img
              src="https://s3.nevaobjects.id/wallking-website/uploads/wlkg-dark.svg"
              alt="Wallking"
              class="h-8"
            />
          </div>
          <!-- Share button -->
          <button
            @click="shareSession"
            class="group flex items-center gap-2 px-4 py-2 rounded-xl bg-brand-600 hover:bg-brand-700 text-white text-sm font-medium transition-all duration-300 shadow-md shadow-brand-500/20 hover:shadow-lg hover:shadow-brand-500/30"
          >
            <svg class="w-4 h-4 group-hover:scale-110 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M7.217 10.907a2.25 2.25 0 1 0 0 2.186m0-2.186c.18.324.283.696.283 1.093s-.103.77-.283 1.093m0-2.186 9.566-5.314m-9.566 7.5 9.566 5.314m0 0a2.25 2.25 0 1 0 3.935 2.186 2.25 2.25 0 0 0-3.935-2.186Zm0-12.814a2.25 2.25 0 1 0 3.933-2.185 2.25 2.25 0 0 0-3.933 2.185Z" />
            </svg>
            <span class="hidden sm:inline">Share</span>
          </button>
        </div>
      </div>
    </header>

    <!-- Session Info -->
    <div class="max-w-3xl mx-auto px-4 mt-6">
      <div class="rounded-2xl bg-white border border-surface-200/60 p-5 shadow-sm">
        <p class="text-xs text-brand-500 font-semibold uppercase tracking-wider mb-3">Your Moments Softfiles</p>
        <div class="flex items-center gap-3">
          <div class="w-12 h-12 rounded-full bg-linear-to-br from-brand-400 to-brand-600 flex items-center justify-center text-white text-lg font-bold shadow-lg shadow-brand-600/30">
            {{ session.name?.charAt(0)?.toUpperCase() || '?' }}
          </div>
          <div class="flex-1 min-w-0">
            <p class="text-xs text-surface-700/50">Terimakasih,</p>
            <h2 class="text-xl font-bold text-surface-900 truncate">{{ session.name }}</h2>
            <div class="flex items-center gap-3 mt-0.5">
              <span class="text-xs text-surface-700/50">{{ formatDate(session.createdAt) }}</span>
              <span class="text-xs text-surface-700/30">•</span>
              <span class="text-xs text-brand-600 font-medium">{{ session.photoCount }} foto</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Video Player (first video only) -->
    <section v-if="session.videos && session.videos.length > 0" class="max-w-3xl mx-auto px-4 mt-6">
      <div class="rounded-2xl overflow-hidden bg-white border border-surface-200/60 shadow-sm">
        <div class="relative bg-surface-100">
          <video
            :src="session.videos[0].url"
            class="w-full h-full object-cover"
            controls
            playsinline
            preload="metadata"
          ></video>
        </div>
        <div class="flex items-center justify-between px-4 py-3">
          <p class="text-sm text-surface-700">Your Moment Video</p>
          <a
            :href="getDownloadLink(session.videos[0].filename, 'video')"
            class="shrink-0 flex items-center gap-1.5 px-3 py-1.5 rounded-lg bg-brand-50 hover:bg-brand-100 text-brand-600 text-xs font-medium transition-all duration-200 hover:scale-105 border border-brand-200/50"
          >
            <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
            </svg>
            Download
          </a>
        </div>
      </div>
    </section>

    <!-- Photos Section -->
    <section v-if="session.photos && session.photos.length > 0" class="max-w-3xl mx-auto px-4 mt-8">
      <div class="flex items-center gap-2 mb-4">
        <svg class="w-5 h-5 text-brand-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
          <path stroke-linecap="round" stroke-linejoin="round" d="m2.25 15.75 5.159-5.159a2.25 2.25 0 0 1 3.182 0l5.159 5.159m-1.5-1.5 1.409-1.409a2.25 2.25 0 0 1 3.182 0l2.909 2.909M3.75 21h16.5A2.25 2.25 0 0 0 22.5 18.75V5.25A2.25 2.25 0 0 0 20.25 3H3.75A2.25 2.25 0 0 0 1.5 5.25v13.5A2.25 2.25 0 0 0 3.75 21Z" />
        </svg>
        <div class="flex flex-col">
          <h3 class="text-base font-semibold text-surface-900 leading-tight">Foto</h3>
          <p class="text-xs text-surface-700/50 mt-0.5">Tap foto untuk memperbesar</p>
        </div>
        <span class="ml-auto text-xs text-surface-700/40">{{ session.photos.length }} file{{ session.photos.length > 1 ? 's' : '' }}</span>
      </div>
      <div class="grid grid-cols-2 gap-3">
        <div
          v-for="(photo, index) in session.photos"
          :key="'photo-' + index"
          class="group relative rounded-2xl overflow-hidden bg-white border border-surface-200/60 hover:border-brand-300 transition-all duration-300 shadow-sm hover:shadow-md"
        >
          <!-- Thumbnail -->
          <div class="relative overflow-hidden cursor-pointer" @click="openLightbox(index)">
            <img
              :src="photo.thumbnailUrl"
              :alt="'Photo ' + (index + 1)"
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
            <p class="text-xs text-surface-700/50 truncate mr-2">{{ formatFileSize(photo.size) }}</p>
            <a
              :href="getDownloadLink(photo.filename, 'photo')"
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

    <!-- Lightbox -->
    <Teleport to="body">
      <Transition name="lightbox">
        <div
          v-if="lightboxOpen"
          class="fixed inset-0 z-100 bg-black/90 backdrop-blur-lg flex items-center justify-center"
          @click.self="closeLightbox"
        >
          <!-- Close button -->
          <button
            @click="closeLightbox"
            class="absolute top-4 right-4 z-10 w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors"
          >
            <svg class="w-5 h-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
            </svg>
          </button>

          <!-- Navigation -->
          <button
            v-if="lightboxIndex > 0"
            @click="lightboxIndex--"
            class="absolute left-3 z-10 w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors"
          >
            <svg class="w-5 h-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5 8.25 12l7.5-7.5" />
            </svg>
          </button>
          <button
            v-if="session.photos && lightboxIndex < session.photos.length - 1"
            @click="lightboxIndex++"
            class="absolute right-3 z-10 w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center transition-colors"
          >
            <svg class="w-5 h-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="m8.25 4.5 7.5 7.5-7.5 7.5" />
            </svg>
          </button>

          <!-- Image -->
          <img
            v-if="session.photos && session.photos[lightboxIndex]"
            :src="session.photos[lightboxIndex].url"
            :alt="'Photo ' + (lightboxIndex + 1)"
            class="max-h-[90vh] max-w-[90vw] object-contain rounded-lg shadow-2xl"
          />

          <!-- Bottom bar -->
          <div class="absolute bottom-4 left-1/2 -translate-x-1/2 flex items-center gap-3">
            <span class="text-sm text-white/50">{{ lightboxIndex + 1 }} / {{ session.photos?.length }}</span>
            <a
              v-if="session.photos && session.photos[lightboxIndex]"
              :href="getDownloadLink(session.photos[lightboxIndex].filename, 'photo')"
              class="flex items-center gap-1.5 px-4 py-2 rounded-xl bg-brand-600 hover:bg-brand-500 text-white text-sm font-medium transition-colors"
            >
              <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
              </svg>
              Download
            </a>
          </div>
        </div>
      </Transition>
    </Teleport>

    <ExpiryNotice />

    <!-- Footer -->
    <footer class="max-w-3xl mx-auto px-4 mt-8 pb-8 text-center">
      <div class="h-px bg-linear-to-r from-transparent via-surface-200 to-transparent mb-6"></div>
      <p class="text-xs text-surface-700/40">2023-2026 © Powered by <span class="text-brand-500 font-medium">Wallking Labs</span></p>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import ExpiryNotice from './ExpiryNotice.vue';

const API_BASE = 'https://bakery.wallking.id';

interface MediaItem {
  filename: string;
  url: string;
  thumbnailUrl?: string;
  size: number;
  customerName: string;
}

interface SessionData {
  sessionId: string;
  name: string;
  createdAt: string;
  photoCount: number;
  videoCount: number;
  totalSize: string;
  totalSizeBytes: number;
  archiveLink: string;
  photos: MediaItem[];
  videos: MediaItem[];
}

const props = defineProps<{
  sessionId: string;
}>();

const session = ref<SessionData | null>(null);
const loading = ref(true);
const error = ref<string | null>(null);

// Lightbox state
const lightboxOpen = ref(false);
const lightboxIndex = ref(0);

const getDownloadLink = (filename: string, type: string): string => {
  return `${API_BASE}/download-${type}/${filename}`;
};

const formatDate = (dateStr: string): string => {
  const d = new Date(dateStr);
  return d.toLocaleDateString('id-ID', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    timeZone: 'Asia/Jakarta',
  });
};

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '—';
  const k = 1024;
  const sizes = ['B', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(1)) + ' ' + sizes[i];
};

const openLightbox = (index: number) => {
  lightboxIndex.value = index;
  lightboxOpen.value = true;
  document.body.style.overflow = 'hidden';
};

const closeLightbox = () => {
  lightboxOpen.value = false;
  document.body.style.overflow = '';
};

const shareSession = async () => {
  const shareData = {
    title: `${session.value?.name} — Wallking`,
    text: `Check out ${session.value?.name}'s softfiles from Wallking Studio!`,
    url: window.location.href,
  };

  if (navigator.share) {
    try {
      await navigator.share(shareData);
    } catch {
      // User cancelled sharing, do nothing
    }
  } else {
    // Fallback: copy link to clipboard
    try {
      await navigator.clipboard.writeText(window.location.href);
      alert('Link copied to clipboard!');
    } catch {
      // Final fallback
      const textArea = document.createElement('textarea');
      textArea.value = window.location.href;
      document.body.appendChild(textArea);
      textArea.select();
      document.execCommand('copy');
      document.body.removeChild(textArea);
      alert('Link copied to clipboard!');
    }
  }
};

const handleKeydown = (e: KeyboardEvent) => {
  if (!lightboxOpen.value) return;
  if (e.key === 'Escape') closeLightbox();
  if (e.key === 'ArrowLeft' && lightboxIndex.value > 0) lightboxIndex.value--;
  if (e.key === 'ArrowRight' && session.value?.photos && lightboxIndex.value < session.value.photos.length - 1) lightboxIndex.value++;
};

onMounted(async () => {
  document.addEventListener('keydown', handleKeydown);

  try {
    const res = await fetch(`${API_BASE}/sessions/${props.sessionId}`);
    if (!res.ok) throw new Error('Sesi tidak ditemukan atau sudah kadaluarsa.');
    session.value = await res.json();
  } catch (e: any) {
    error.value = e.message || 'Failed to load session.';
  } finally {
    loading.value = false;
  }
});
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
