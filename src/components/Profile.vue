<script setup>
import { ref } from "vue"; // Import các thành phần cần thiết
import EditAccount from "./EditAccount.vue";
import Swal from "sweetalert2";

// Nhận props từ component cha
const props = defineProps({
  currentUser: Object,
  posts: Array,
});

// Định nghĩa các emit
const emit = defineEmits(["editPost", "updatePosts", "editUser"]);

const isModalVisible = ref(false); // Trạng thái hiển thị modal

// Mở modal chỉnh sửa thông tin người dùng
function editUser() {
  isModalVisible.value = true;
}

// Đóng modal chỉnh sửa thông tin người dùng
function closeModal() {
  isModalVisible.value = false;
}

// Hàm lưu thay đổi thông tin người dùng
function handleSaveChanges(updatedUser) {
  // Phát sự kiện để thông báo thay đổi lên App.vue
  emit("editUser", updatedUser);

  // Cập nhật bài viết nếu cần thiết
  filterUserPosts();

  // Đóng modal
  closeModal();
}

// Lọc bài viết của người dùng hiện tại và sắp xếp theo thứ tự mới nhất
const userPosts = ref([]);

function filterUserPosts() {
  if (props.currentUser) {
    console.log("User đang đăng nhập:", props.currentUser); // Kiểm tra currentUser

    userPosts.value = props.posts
      .filter((post) => post.userId === props.currentUser.id)
      .sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt)); // Sắp xếp bài viết theo ngày

    console.log("Bài viết của người dùng hiện tại:", userPosts.value);
  } else {
    userPosts.value = [];
    console.log("Không có người dùng đăng nhập, xóa bài viết.");
  }
}

// Phân trang bài viết
const currentPage = ref(1);
const postsPerPage = 5;

function getTotalPages() {
  return Math.ceil(userPosts.value.length / postsPerPage);
}

function getPaginatedPosts() {
  const startIndex = (currentPage.value - 1) * postsPerPage;
  return userPosts.value.slice(startIndex, startIndex + postsPerPage);
}

function changePage(page) {
  if (page < 1 || page > getTotalPages()) return;
  currentPage.value = page;
  console.log("Đang ở trang:", currentPage.value);

  // Lọc lại bài viết khi thay đổi trang
  filterUserPosts();
}

// Xóa bài viết
function deletePost(postId) {
  Swal.fire({
    title: "Bạn có chắc chắn?",
    text: "Hành động này không thể hoàn tác!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: "#d33",
    confirmButtonText: "Xóa",
    cancelButtonText: "Hủy",
  }).then((result) => {
    if (result.isConfirmed) {
      // Xóa bài viết khỏi userPosts
      userPosts.value = userPosts.value.filter((post) => post.id !== postId);

      // Phát ra sự kiện để cập nhật danh sách bài viết
      const updatedPosts = props.posts.filter((post) => post.id !== postId);
      emit("updatePosts", updatedPosts);

      Swal.fire("Đã xóa!", "Bài viết đã được xóa.", "success");

      // Gọi lại filterUserPosts sau khi xóa
      filterUserPosts();
    }
  });
}

// Chỉnh sửa bài viết
function editPost(post) {
  emit("editPost", post);
}

// Hàm cắt nội dung bài viết để hiển thị ngắn gọn
function truncateContent(content, length = 100) {
  if (content && content.length > length) {
    return content.slice(0, length) + "...";
  }
  return content;
}

// Gọi filterUserPosts khi component được hiển thị
filterUserPosts();
</script>

<template>
  <div class="container">
    <div class="row">
      <!-- Thông tin người dùng (Sidebar) -->
      <div class="col-md-4">
        <div class="card sticky-sidebar" v-if="props.currentUser">
          <div class="card-body text-center">
            <div class="d-flex justify-content-center">
              <img
                v-if="props.currentUser.avatar"
                :src="props.currentUser.avatar"
                alt="Avatar"
                class="rounded-circle mb-3"
                style="width: 150px; height: 150px; object-fit: cover"
              />
              <div
                v-else
                class="avatar-placeholder rounded-circle mb-3 d-flex justify-content-center align-items-center bg-secondary text-white"
                style="
                  width: 150px;
                  height: 150px;
                  font-size: 2rem;
                  text-align: center;
                "
              >
                <span>{{
                  props.currentUser.name ? props.currentUser.name[0] : "A"
                }}</span>
              </div>
            </div>
            <h5 class="card-title">Thông tin cá nhân</h5>
            <p>
              <strong>Họ tên:</strong>
              {{ props.currentUser.name || "Chưa cập nhật" }}
            </p>
            <p>
              <strong>Email:</strong>
              {{ props.currentUser.email || "Chưa cập nhật" }}
            </p>
            <button class="btn btn-secondary mt-2" @click="editUser">
              Chỉnh sửa thông tin
            </button>
          </div>
        </div>
      </div>

      <!-- Danh sách bài viết -->
      <div class="col-md-8">
        <div class="d-flex justify-content-between align-items-center mb-3">
          <h5>Bài viết của bạn</h5>
          <button class="btn btn-primary" @click="$emit('editPost', null)">
            Thêm bài viết
          </button>
        </div>
        <div v-if="userPosts.length === 0">Chưa có bài viết nào.</div>
        <div v-else>
          <div
            class="card mb-3"
            v-for="post in getPaginatedPosts()"
            :key="post.id"
          >
            <div class="row g-0">
              <div class="col-md-4">
                <img
                  :src="post.image"
                  class="img-fluid rounded-start p-3 pe-0"
                  style="height: 180px"
                />
              </div>
              <div class="col-md-8">
                <div class="card-body">
                  <h5 class="card-title">{{ post.title }}</h5>
                  <p class="card-text">{{ truncateContent(post.content) }}</p>
                  <div class="d-flex justify-content-end">
                    <button
                      class="btn btn-warning me-2"
                      @click="editPost(post)"
                    >
                      Sửa
                    </button>
                    <button class="btn btn-danger" @click="deletePost(post.id)">
                      Xóa
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Phân trang -->
          <div class="d-flex justify-content-center mt-3">
            <ul class="pagination pagination-sm">
              <li class="page-item" :class="{ disabled: currentPage === 1 }">
                <button class="page-link" @click="changePage(currentPage - 1)">
                  Previous
                </button>
              </li>
              <li
                v-for="page in getTotalPages()"
                :key="page"
                class="page-item"
                :class="{ active: currentPage === page }"
              >
                <button class="page-link" @click="changePage(page)">
                  {{ page }}
                </button>
              </li>
              <li
                class="page-item"
                :class="{ disabled: currentPage === getTotalPages() }"
              >
                <button class="page-link" @click="changePage(currentPage + 1)">
                  Next
                </button>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal chỉnh sửa thông tin người dùng -->
  <EditAccount
    v-if="isModalVisible"
    :currentUser="currentUser"
    :isModalVisible="isModalVisible"
    @closeModal="closeModal"
    @saveChanges="handleSaveChanges"
  />
</template>
