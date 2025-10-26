<template>
    <div class="fixed inset-0 bg-gray-900 flex items-center justify-center z-50">
      <div class="max-w-md w-full mx-4">
        <!-- Main content card -->
        <div class="bg-gray-800 p-8 rounded-lg shadow-xl border border-violet-600">

          <!-- Icon/Spinner area -->
          <div class="flex items-center justify-center mb-8">
            <div class="relative w-20 h-20">
              <!-- Rotating spinner ring -->
              <div class="absolute inset-0 border-4 border-gray-700 rounded-full"></div>
              <div class="absolute inset-0 border-4 border-transparent border-t-violet-600 rounded-full animate-spin"></div>

              <!-- Center icon -->
              <div class="absolute inset-0 flex items-center justify-center">
                <svg class="w-8 h-8 text-violet-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"></path>
                </svg>
              </div>
            </div>
          </div>

          <!-- Text content -->
          <div class="text-center space-y-4">
            <h2 class="text-2xl font-bold text-violet-300">
              Processing Dependencies
            </h2>

            <!-- Status message -->
            <div class="min-h-[80px] flex items-center justify-center px-4">
              <p class="text-violet-200 text-base transition-all duration-300">
                {{ message || 'Analyzing your project...' }}
              </p>
            </div>

            <!-- Progress dots -->
            <div class="flex justify-center space-x-2 pt-2">
              <div class="w-2 h-2 bg-violet-600 rounded-full animate-bounce" style="animation-delay: 0ms"></div>
              <div class="w-2 h-2 bg-violet-600 rounded-full animate-bounce" style="animation-delay: 150ms"></div>
              <div class="w-2 h-2 bg-violet-600 rounded-full animate-bounce" style="animation-delay: 300ms"></div>
            </div>

            <!-- Additional info -->
            <p class="text-gray-400 text-sm pt-4">
              This may take a few moments...
            </p>
          </div>
        </div>
      </div>
    </div>
  </template>

<script setup>
defineProps({
  message: {
    type: String,
    default: ''
  }
})
</script>