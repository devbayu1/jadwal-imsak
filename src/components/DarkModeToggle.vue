<template>
  <button 
    @click="toggleDarkMode" 
    class="dark-mode-toggle"
    :class="{ 'dark': isDark }"
    aria-label="Toggle dark mode"
  >
    <svg v-if="isDark" class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <circle cx="12" cy="12" r="5"></circle>
      <line x1="12" y1="1" x2="12" y2="3"></line>
      <line x1="12" y1="21" x2="12" y2="23"></line>
      <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
      <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
      <line x1="1" y1="12" x2="3" y2="12"></line>
      <line x1="21" y1="12" x2="23" y2="12"></line>
      <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
      <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
    </svg>
    <svg v-else class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
    </svg>
  </button>
</template>

<script>
export default {
  name: 'DarkModeToggle',
  data() {
    return {
      isDark: false
    }
  },
  mounted() {
    // Periksa preferensi pengguna dari local storage atau preferensi sistem
    this.isDark = localStorage.getItem('darkMode') === 'true' || 
                 (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches);
    this.updateTheme();
  },
  methods: {
    toggleDarkMode() {
      this.isDark = !this.isDark;
      localStorage.setItem('darkMode', this.isDark);
      this.updateTheme();
    },
    updateTheme() {
      // Tambahkan atau hapus class 'dark-theme' di elemen html
      if (this.isDark) {
        document.documentElement.classList.add('dark-theme');
      } else {
        document.documentElement.classList.remove('dark-theme');
      }
    }
  }
}
</script>

<style scoped>
.dark-mode-toggle {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;
  border: none;
  outline: none;
  background-color: #f0f0f0;
  color: #333;
}

.dark-mode-toggle:hover {
  background-color: #e0e0e0;
}

.dark-mode-toggle.dark {
  background-color: #4a5568;
  color: #f0c14b;
}

.dark-mode-toggle.dark:hover {
  background-color: #2d3748;
}

.icon {
  width: 20px;
  height: 20px;
}
</style>