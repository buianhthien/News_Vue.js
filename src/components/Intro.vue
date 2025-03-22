<template>
  <div
    class="d-flex flex-column justify-content-center align-items-center vh-100"
    style="
      background-image: url('https://i.pinimg.com/736x/f9/5a/dc/f95adc4a984af221d53a13b7b2df0f29.jpg');
      background-size: cover;
      background-position: center;
    "
  >
    <h1 class="mb-4 text-white fw-bold">Chào mừng đến với Blog!</h1>
    <div>
      <button @click="$emit('navigate', 'Login')" class="btn btn-primary me-3">
        Đăng nhập
      </button>
      <button
        @click="$emit('navigate', 'Register')"
        class="btn btn-outline-primary"
      >
        Đăng ký
      </button>
    </div>
  </div>
</template>
