<script setup>
import { ref } from "vue";
import Swal from "sweetalert2"; // Import SweetAlert2

// Define props to accept users from the parent
const props = defineProps({
  users: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits();
const email = ref("");
const password = ref("");

function handleLogin() {
  // Kiểm tra nếu chưa nhập đầy đủ thông tin
  if (!email.value || !password.value) {
    Swal.fire({
      icon: "error",
      title: "Thông tin chưa đầy đủ!",
      text: "Vui lòng nhập cả email và mật khẩu.",
      confirmButtonText: "OK",
    });
    return; // Dừng lại nếu chưa nhập đầy đủ thông tin
  }

  // Tìm người dùng có email và mật khẩu trùng khớp
  const user = props.users.find(
    (u) => u.email === email.value && u.password === password.value
  );

  // Nếu tìm thấy người dùng, đăng nhập thành công
  if (user) {
    // Lưu thông tin người dùng vào localStorage
    localStorage.setItem("currentUser", JSON.stringify(user));

    // Lưu bài viết của người dùng vào localStorage (nếu có)
    const userPosts = user.posts || []; // Giả sử mỗi người dùng có một mảng `posts`
    localStorage.setItem("currentUserPosts", JSON.stringify(userPosts));

    // Emit sự kiện login success
    emit("loginSuccess", user);
  } else {
    // Nếu thông tin đăng nhập sai
    Swal.fire({
      icon: "error",
      title: "Thông tin đăng nhập không chính xác!",
      text: "Vui lòng kiểm tra lại email và mật khẩu.",
      confirmButtonText: "OK",
    });
  }
}

function goToRegister() {
  emit("switchToRegister");
}
</script>

<template>
  <div class="login-page">
    <div
      class="container d-flex justify-content-center align-items-center vh-100"
    >
      <div class="card w-100" style="max-width: 400px">
        <div class="card-body">
          <h2 class="text-center mb-4">Đăng nhập</h2>
          <form @submit.prevent="handleLogin">
            <div class="mb-3">
              <label for="email" class="form-label">Email</label>
              <input
                id="email"
                v-model="email"
                type="email"
                class="form-control"
                placeholder="Nhập email"
              />
            </div>
            <div class="mb-3">
              <label for="password" class="form-label">Mật khẩu</label>
              <input
                id="password"
                v-model="password"
                type="password"
                class="form-control"
                placeholder="Nhập mật khẩu"
              />
            </div>
            <button type="submit" class="btn btn-primary w-100 mb-3">
              Đăng nhập
            </button>

            <a
              class="nav-link text-primary mb-3 text-center"
              @click="goToRegister"
              href="#"
              >Chưa có tài khoản, Đăng ký?</a
            >
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Set background image for the login page */
.login-page {
  background-image: url("https://i.pinimg.com/736x/f9/5a/dc/f95adc4a984af221d53a13b7b2df0f29.jpg"); /* Thay URL bằng hình ảnh bạn muốn */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  height: 100vh;
}

.card-body {
  background-color: rgba(
    255,
    255,
    255,
    0.8
  ); /* Tạo nền mờ cho form đăng nhập */
  border-radius: 10px;
}

h2 {
  color: #007bff;
}
</style>
