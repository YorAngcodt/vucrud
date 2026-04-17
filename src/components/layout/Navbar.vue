<template>
  <nav class="navbar">
    <div class="navbar-container">
      <!-- Logo Vue di kiri -->
      <div class="navbar-left">
        <div class="logo">
          <img src="https://vuejs.org/images/logo.png" alt="Vue Logo" class="vue-logo" />
          <span class="logo-text">Vue CRUD</span>
        </div>
      </div>

      <!-- Menu Home & Stok di kanan -->
      <div class="navbar-right">
        <router-link to="/" class="nav-link">Home</router-link>
        <router-link to="/stok" class="nav-link">Stok</router-link>
      </div>
    </div>
  </nav>
</template>

<script setup>
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.navbar {
  background: linear-gradient(135deg, #42b883 0%, #35495e 100%);
  padding: 1rem 2rem;
  box-shadow: 0 2px 15px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.navbar-container {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Logo section kiri */
.navbar-left .logo {
  display: flex;
  align-items: center;
  gap: 12px;
}

.vue-logo {
  width: 40px;
  height: 35px;
  filter: brightness(0) invert(1);
}

.logo-text {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  letter-spacing: 1px;
}

/* Menu section kanan */
.navbar-right {
  display: flex;
  gap: 2rem;
  align-items: center;
}

.nav-link {
  color: white;
  text-decoration: none;
  font-size: 1.1rem;
  font-weight: 500;
  padding: 8px 16px;
  border-radius: 8px;
  transition: all 0.3s ease;
  position: relative;
}

.nav-link:hover {
  background-color: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

/* Active link styling */
.nav-link.router-link-active {
  background-color: rgba(255, 255, 255, 0.25);
  font-weight: bold;
}

.nav-link.router-link-active::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 50%;
  transform: translateX(-50%);
  width: 30px;
  height: 3px;
  background-color: white;
  border-radius: 2px;
}

/* Responsive */
@media (max-width: 768px) {
  .navbar {
    padding: 0.8rem 1rem;
  }
  
  .logo-text {
    font-size: 1.2rem;
  }
  
  .vue-logo {
    width: 30px;
    height: 26px;
  }
  
  .nav-link {
    font-size: 1rem;
    padding: 6px 12px;
  }
  
  .navbar-right {
    gap: 1rem;
  }
}
</style>