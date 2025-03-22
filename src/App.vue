<script setup>
import { reactive, ref } from "vue";
import Swal from "sweetalert2";
import Intro from "./components/Intro.vue";
import Login from "./components/Login.vue";
import Register from "./components/Register.vue";
import Home from "./components/Home.vue";
import Profile from "./components/Profile.vue";
import PostDetail from "./components/PostDetail.vue";
import PostForm from "./components/PostForm.vue";
import EditAccount from "./components/EditAccount.vue";

// Reactive state for users, posts, and the current user
const users = reactive([]); // Danh sách người dùng
const  posts = reactive([]); // Danh sách bài viết
const currentUser = ref(null); // Người dùng hiện tại
const currentPage = ref("Login"); // Trang hiện tại
const selectedPost = ref(null); // Bài viết được chọn

// Điều hướng giữa các trang
function navigate(page, payload = null) {
  currentPage.value = page;
  selectedPost.value = payload;

  console.log("Chuyển sang trang:", page); 
  console.log("Bài viết đã chọn:", selectedPost.value); 
}
function getUserPosts() {
  if (!currentUser.value) return []; 
  return posts.filter((post) => post.userId === currentUser.value.id);
}

function handleLogin(newUser) {
  console.log("Đăng nhập người dùng:", newUser);

  // Cập nhật currentUser
  currentUser.value = newUser;
  console.log("currentUser sau khi đăng nhập:", currentUser.value);

  // Lọc lại bài viết của người dùng
  const userPosts = getUserPosts();
  console.log("Bài viết của người dùng sau khi đăng nhập:", userPosts);

  // Chuyển sang trang Home
  navigate("Home");
}
// function handleLogin(user) {
//   currentUser.value = user;
// }

// Xử lý đăng xuất
function logout() {
  console.log("Đăng xuất người dùng:", currentUser.value); // Kiểm tra người dùng cũ
  Swal.fire({
    title: "Bạn có chắc chắn muốn đăng xuất?",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "Đăng xuất",
    cancelButtonText: "Hủy",
    reverseButtons: true,
  }).then((result) => {
    if (result.isConfirmed) {
      currentUser.value = null;
      navigate("Login");

      // Làm mới danh sách bài viết hiển thị trên Profile (xóa bài viết nếu cần)
      console.log("Danh sách bài viết sau khi đăng xuất:", posts);
      Swal.fire("Đã đăng xuất", "Bạn đã đăng xuất thành công.", "success");
    }
  });
}

// Cập nhật danh sách bài viết
function updatePosts(postData) {
  if (Array.isArray(postData)) {
    posts.splice(0, posts.length, ...postData);
  } else {
    const index = posts.findIndex((p) => p.id === postData.id);
    if (index === -1) {
      posts.unshift(postData);
    } else {
      posts[index] = postData;
    }
  }
}

// function getUserPosts() {
//   if (!currentUser.value) return [];
//   return posts.filter((post) => post.userId === currentUser.value.id); // Sử dụng userId
// }

// Lưu thay đổi thông tin người dùng
function handleSaveChanges(updatedUser) {
  updatedUser.avatar = updatedUser.profileImage;
  currentUser.value = { ...updatedUser };
  if (updatedUser.profileImage) {
    currentUser.value.avatar = updatedUser.profileImage;
  }
  const index = users.findIndex((user) => user.id === updatedUser.id);
  if (index !== -1) {
    users[index] = { ...updatedUser };
  }
  Swal.fire({
    title: "Thành công",
    text: "Thông tin tài khoản đã được cập nhật.",
    icon: "success",
    confirmButtonText: "OK",
  });
}
</script>

<template>
  <div>
    <!-- Thanh điều hướng -->
    <header
      v-if="currentUser"
      class="navbar navbar-expand-lg navbar-light bg-secondary-subtle"
    >
      <div class="container-fluid">
        <a class="navbar-brand text-white" href="#">
          <img
            src="../src/assets/images/FPT_Polytechnic.png"
            width="150"
            alt=""
          />
        </a>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
          aria-controls="navbarNav"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav">
            <li class="nav-item">
              <button
                @click="navigate('Home')"
                class="btn btn-link text-dark"
                style="text-decoration: none"
              >
                <i class="fa-solid fa-list"></i> Bài viết
              </button>
            </li>
            <li class="nav-item">
              <button
                @click="navigate('Profile')"
                class="btn btn-link text-dark"
                style="text-decoration: none"
              >
                <i class="fa-solid fa-circle-info"></i> Trang cá nhân
              </button>
            </li>
          </ul>
        </div>

        <!-- Hiển thị tên người dùng và menu -->
        <div class="dropdown">
          <button
            class="btn btn-outline-light text-dark"
            type="button"
            data-bs-toggle="dropdown"
            aria-expanded="false"
          >
            Xin chào, {{ currentUser.name }}
          </button>
          <ul class="dropdown-menu dropdown-menu-end">
            <li>
              <a class="dropdown-item" href="#" @click="logout">
                <i class="fa-solid fa-right-from-bracket"></i> Đăng xuất
              </a>
            </li>
          </ul>
        </div>
      </div>
    </header>

    <!-- Nội dung chính -->
    <div>
      <Intro v-if="currentPage === 'Intro'" @navigate="navigate" />
      <Login
        v-else-if="currentPage === 'Login'"
        @switchToRegister="navigate('Register')"
        :users="users"
        @loginSuccess="handleLogin"
      />
      <Register
        v-else-if="currentPage === 'Register'"
        @switchToLogin="navigate('Login')"
        :users="users"
        @registerSuccess="navigate"
      />
      <Home
        v-else-if="currentPage === 'Home'"
        :posts="posts"
        @viewPost="(post) => navigate('PostDetail', post)"
      />
      <Profile
        v-else-if="currentPage === 'Profile'"
        :key="JSON.stringify(currentUser)"
        :currentUser="currentUser"
        :posts="getUserPosts()"
        @editPost="(post) => navigate('PostForm', post)"
        @editUser="handleSaveChanges"
        @updatePosts="updatePosts"
      />
      <PostDetail
        v-else-if="currentPage === 'PostDetail'"
        :post="selectedPost"
        :currentUser="currentUser"
      />
      <PostForm
        v-else-if="currentPage === 'PostForm'"
        :post="selectedPost"
        :currentUser="currentUser"
        :posts="posts"
        @savePost="(page) => navigate(page)"
        @updatePosts="updatePosts"
      />
      <EditAccount
        v-else-if="currentPage === 'EditAccount' && currentUser"
        :currentUser="currentUser"
        @saveChanges="handleSaveChanges"
      />
    </div>
  </div>
</template>

<style scoped>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}
.navbar {
  margin-bottom: 20px;
}
</style>
