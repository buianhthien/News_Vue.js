<script setup>
import { ref, computed } from "vue";

// Nhận dữ liệu bài viết từ component cha thông qua props
const props = defineProps({
  posts: {
    type: Array,
    required: true,
  },
});

// Biến trạng thái tìm kiếm
const searchTerm = ref("");
const currentPage = ref(1); // Trang hiện tại
const postsPerPage = 5; // Số bài viết trên mỗi trang

// Hàm cắt ngắn nội dung bài viết
function truncateText(text, maxLength) {
  if (!text) return "";
  return text.length > maxLength ? text.substring(0, maxLength) + "..." : text;
}

// Định nghĩa emit để phát sự kiện
const emit = defineEmits();

// Hàm xử lý khi nhấn "Đọc thêm" - dùng emit để thông báo về bài viết được chọn
function viewPost(post) {
  emit("viewPost", post); // Truyền dữ liệu bài viết lên component cha
  updateViews(post.id); // Cập nhật lượt xem
}

// Hàm định dạng ngày tháng
function formatDate(date) {
  const options = { year: "numeric", month: "2-digit", day: "2-digit" };
  return new Date(date).toLocaleDateString("vi-VN", options);
}

// Lọc các bài viết dựa trên từ khóa tìm kiếm
const filteredPosts = computed(() => {
  if (!searchTerm.value) return props.posts;
  const lowerSearchTerm = searchTerm.value.toLowerCase();
  return props.posts.filter(
    (post) =>
      post.title.toLowerCase().includes(lowerSearchTerm) ||
      post.content.toLowerCase().includes(lowerSearchTerm)
  );
});

// Phân trang: lấy các bài viết cho trang hiện tại
const paginatedPosts = computed(() => {
  const startIndex = (currentPage.value - 1) * postsPerPage;
  const endIndex = startIndex + postsPerPage;
  return filteredPosts.value.slice(startIndex, endIndex);
});

// Tổng số trang
const totalPages = computed(() => {
  return Math.ceil(filteredPosts.value.length / postsPerPage);
});

// Điều hướng trang
function goToPage(page) {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
  }
}

// Hàm tìm kiếm
function search() {
  console.log("Tìm kiếm: ", searchTerm.value);
}

// Hàm cập nhật số lượt xem trong localStorage
function updateViews(postId) {
  let views = localStorage.getItem(postId);
  if (views === null) {
    views = 0; // Nếu chưa có lượt xem, mặc định là 0
  }
  views++; // Tăng lượt xem
  localStorage.setItem(postId, views); // Lưu lại lượt xem vào localStorage
}

// Hàm lấy số lượt xem từ localStorage
function getViews(postId) {
  return parseInt(localStorage.getItem(postId)) || 0; // Lấy số lượt xem từ localStorage và convert sang số
}

// Lọc và sắp xếp các bài viết trong mục "Xem nhiều" theo lượt xem
const sortedPosts = computed(() => {
  return props.posts
    .map((post) => ({
      ...post,
      views: getViews(post.id), // Lấy số lượt xem từ localStorage
    }))
    .sort((a, b) => b.views - a.views); // Sắp xếp giảm dần theo lượt xem
});
</script>

<template>
  <div class="container mt-4">
    <div class="row">
      <!-- Cột chính (Danh sách bài viết) -->
      <div class="col-lg-8">
        <div v-if="filteredPosts.length === 0" class="text-center">
          <p>Không tìm thấy bài viết nào.</p>
        </div>
        <div
          v-for="post in paginatedPosts"
          :key="post.id"
          class="card mb-3"
          style="max-width: 100%"
        >
          <div class="row g-0">
            <!-- Ảnh bài viết -->
            <div class="col-md-4">
              <img
                v-if="post.image"
                :src="post.image"
                class="img-fluid rounded-start mt-3 ps-3"
                style="height: 180px"
                alt="Post image"
              />
              <div
                v-else
                class="d-flex align-items-center justify-content-center bg-light"
                style="height: 100%; min-height: 200px"
              >
                <span class="text-muted">350 x 200</span>
              </div>
            </div>
            <!-- Nội dung bài viết -->
            <div class="col-md-8">
              <div class="card-body">
                <h5 class="card-title">{{ post.title }}</h5>
                <p class="card-text">
                  <small class="text-muted">
                    Bởi: {{ post.author }} | {{ formatDate(post.date) }}
                  </small>
                </p>
                <p class="card-text">{{ truncateText(post.content, 100) }}</p>
                <p class="card-text">
                  <small class="text-muted">
                    Lượt xem: {{ getViews(post.id) }}
                  </small>
                </p>
                <button @click="viewPost(post)" class="btn btn-primary btn-sm">
                  Đọc thêm
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Phân trang -->
        <ul class="pagination pagination-sm justify-content-sm-center mt-3">
          <!-- Previous -->
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <a class="page-link" @click.prevent="goToPage(currentPage - 1)">
              Previous
            </a>
          </li>

          <!-- Trang số -->
          <li
            v-for="page in totalPages"
            :key="page"
            :class="{ active: currentPage === page }"
            class="page-item"
          >
            <a
              class="page-link"
              @click.prevent="goToPage(page)"
              :href="'#page' + page"
              >{{ page }}</a
            >
          </li>

          <!-- Trang cuối cùng -->
          <li
            class="page-item"
            :class="{ disabled: currentPage === totalPages }"
          >
            <a class="page-link" @click.prevent="goToPage(currentPage + 1)">
              Next
            </a>
          </li>
        </ul>
      </div>

      <!-- Sidebar và Thanh tìm kiếm -->
      <div class="col-lg-4">
        <div class="d-flex flex-column">
          <!-- Thanh tìm kiếm -->
          <div class="d-flex mb-4">
            <input
              v-model="searchTerm"
              type="text"
              class="form-control w-75"
              placeholder="Tìm kiếm bài viết..."
            />
            <button class="btn btn-primary ms-2" @click="search">
              Tìm kiếm
            </button>
          </div>

          <!-- Mục Xem nhiều -->
          <h5 class="mb-4">Xem nhiều</h5>
          <ul class="list-group list-group-flush">
            <li
              v-for="(post, index) in sortedPosts.slice(0, 5)"
              :key="'sidebar-' + post.id"
              class="list-group-item d-flex justify-content-between align-items-center"
            >
              <span>
                <strong>{{
                  index + 1 < 10 ? "0" + (index + 1) : index + 1
                }}</strong>
                {{ post.title }}
              </span>
              <span class="badge bg-primary rounded-pill">
                {{ post.views }} Lượt xem
              </span>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>
