<script setup>
import { ref } from "vue";
import Swal from "sweetalert2"; // Import SweetAlert2

// Nhận props từ Parent Component (users)
const props = defineProps({
  users: {
    type: Array,
    required: true,
  },
});

// Define the emit function
const emit = defineEmits();

// Reactive data for registration
const name = ref("");
const email = ref("");
const password = ref("");
const passwordConfirm = ref("");

function handleRegister() {
  if (
    !name.value ||
    !email.value ||
    !password.value ||
    !passwordConfirm.value
  ) {
    Swal.fire({
      icon: "error",
      title: "Thông tin chưa đầy đủ!",
      text: "Vui lòng điền đầy đủ tất cả các trường.",
      confirmButtonText: "OK",
    });
    return;
  }

  // Kiểm tra từng email trong mảng users
  for (let i = 0; i < props.users.length; i++) {
    if (props.users[i].email === email.value) {
      Swal.fire({
        icon: "error",
        title: "Email đã tồn tại!",
        text: "Vui lòng sử dụng email khác.",
        confirmButtonText: "OK",
      });
      return;
    }
  }

  // Kiểm tra mật khẩu xác nhận
  if (password.value !== passwordConfirm.value) {
    Swal.fire({
      icon: "error",
      title: "Mật khẩu không khớp!",
      text: "Vui lòng kiểm tra lại mật khẩu xác nhận.",
      confirmButtonText: "OK",
    });
    return;
  }

  // Thêm người dùng mới và tạo ID duy nhất cho người dùng
  const newUser = {
    id: Date.now(), // Tạo ID duy nhất bằng cách sử dụng timestamp
    name: name.value,
    email: email.value,
    password: password.value,
  };
  console.log("user vừa đăng ký", newUser);
  // Thêm người dùng mới vào danh sách
  props.users.push(newUser);

  Swal.fire({
    icon: "success",
    title: "Đăng ký thành công!",
    text: "Chúc mừng bạn đã đăng ký thành công!",
    confirmButtonText: "OK",
  });

  emit("registerSuccess", "Login");
}
function goToLogin() {
  emit("switchToLogin");
}
</script>

<template>
  <div class="register-page">
    <div
      class="container d-flex justify-content-center align-items-center vh-100"
    >
      <div class="card w-100" style="max-width: 400px">
        <div class="card-body">
          <h2 class="text-center mb-4">Đăng ký</h2>
          <form @submit.prevent="handleRegister">
            <div class="mb-3">
              <label for="name" class="form-label">Họ và tên</label>
              <input
                id="name"
                v-model="name"
                type="text"
                class="form-control"
                placeholder="Nhập họ và tên"
              />
            </div>
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
            <div class="mb-3">
              <label for="passwordConfirm" class="form-label"
                >Xác nhận mật khẩu</label
              >
              <input
                id="passwordConfirm"
                v-model="passwordConfirm"
                type="password"
                class="form-control"
                placeholder="Nhập mật khẩu"
              />
            </div>
            <button type="submit" class="btn btn-primary w-100 mb-3">
              Đăng ký
            </button>
            <a
              class="nav-link text-primary mb-3 text-center"
              @click="goToLogin"
              href="#"
              >Đã có tài khoản, Đăng nhập?</a
            >
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Background image */
.register-page {
  background-image: url("https://i.pinimg.com/736x/f9/5a/dc/f95adc4a984af221d53a13b7b2df0f29.jpg"); /* Thay URL bằng hình ảnh bạn muốn */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  height: 100vh;
}

.card-body {
  background-color: rgba(255, 255, 255, 0.8); /* Tạo nền mờ cho form đăng ký */
  border-radius: 10px;
}

h2 {
  color: #007bff;
}
</style>
