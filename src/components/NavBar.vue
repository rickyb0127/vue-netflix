<template>
  <div class="navbar" id="navbar">
    <img class="img" src="../assets/logo.svg" />
    <div class="menu-items">
      <span class="menu-item-spacing">Home</span>
      <span class="menu-item-spacing">TV Shows</span>
      <span class="menu-item-spacing">Movies</span>
      <span class="menu-item-spacing">New & Popular</span>
      <span class="menu-item-spacing">My List</span>
      <span>Browse by Languages</span>
    </div>
    <div class="right-side-nav">
      <i class="fa-solid fa-magnifying-glass menu-item-spacing"></i>
      <span class="menu-item-spacing">DVD</span>
      <i class="fa-regular fa-bell menu-item-spacing"></i>
      <i class="fa-solid fa-user-ninja"></i>
      <i class="fa-solid fa-caret-down"></i>
    </div>
  </div>
</template>

<script>

export default {
  name: 'NavBar',
  methods: {
    handleScroll() {
      const elem = document.getElementById('navbar');

      if(window.scrollY > 30) {
        elem.classList.add('sticky');
      } else {
        elem.classList.remove('sticky');
      }
    }
  },
  created () {
    window.addEventListener('scroll', this.handleScroll);
  },
  unmounted () {
    window.removeEventListener('scroll', this.handleScroll);
  }
}
</script>

<style scoped>
.navbar {
  width: 100%;
  height: 68px;
  background-color: transparent;
  padding-left: 55px;
  padding-right: 55px;
  display: flex;
  align-items: center;
  color: white;
  position: absolute;
  top: 0;
  z-index: 1000;
}

.navbar.sticky {
  position: sticky;
  background-color: black;
}

.navbar .img {
  height: 25px;
  display: inline-block;
  vertical-align: middle;
  cursor: pointer;
}

.navbar .menu-items {
  padding-left: 50px;
  cursor: pointer;
}

.menu-item-spacing {
  padding-right: 20px;
}

.navbar .right-side-nav {
  margin-left: auto;
  margin-right: 110px;
  cursor: pointer;
}
</style>
