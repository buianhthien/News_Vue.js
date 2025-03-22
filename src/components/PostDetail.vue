<script setup>
import { ref } from "vue";

const props = defineProps({
  post: Object,
  currentUser: Object,
});

// Kiểm tra giá trị currentUser và post
console.log("Current User in PostDetail:", props.currentUser);
console.log("Post in PostDetail:", props.post);

// Biến lưu danh sách bình luận
const comments = ref([]); // Khởi tạo biến comments là một mảng rỗng

// Lưu bình luận vào localStorage
function saveCommentToLocalStorage(postId, comment) {
  const storedComments = JSON.parse(localStorage.getItem("comments")) || {};

  if (!storedComments[postId]) {
    storedComments[postId] = [];
  }

  storedComments[postId].push(comment);

  // Lưu lại vào localStorage
  localStorage.setItem("comments", JSON.stringify(storedComments));
}

// Lấy bình luận từ localStorage
function getCommentsFromLocalStorage(postId) {
  const storedComments = JSON.parse(localStorage.getItem("comments")) || {};
  return storedComments[postId] || [];
}

// Lấy bình luận ngay khi component được khởi tạo
comments.value = getCommentsFromLocalStorage(props.post.id);

// Biến lưu nội dung bình luận mới
const newComment = ref("");
const replyCommentId = ref(null); // ID của bình luận đang được trả lời
const newReply = ref(""); // Nội dung trả lời

// Hàm định dạng ngày tháng
const formatDate = (date) => {
  const options = { year: "numeric", month: "2-digit", day: "2-digit" };
  return new Date(date).toLocaleDateString("vi-VN", options);
};

// Hàm gửi bình luận mới
const addComment = () => {
  if (newComment.value.trim() === "") return; // Kiểm tra nội dung không rỗng
  const comment = {
    id: Date.now(),
    author: props.currentUser.name, // Tên người dùng đang đăng nhập
    avatar: props.currentUser.avatar || "https://via.placeholder.com/50", // Ảnh đại diện của người dùng
    content: newComment.value,
    date: new Date().toISOString(),
    replies: [],
  };

  comments.value.push(comment); // Thêm bình luận vào mảng
  saveCommentToLocalStorage(props.post.id, comment); // Lưu vào localStorage
  newComment.value = ""; // Xóa nội dung sau khi gửi
};

// Hàm gửi trả lời
const addReply = (commentId) => {
  if (newReply.value.trim() === "") return; // Kiểm tra nội dung không rỗng
  const comment = comments.value.find((c) => c.id === commentId);
  if (comment) {
    // Thêm trả lời vào bình luận
    comment.replies.push({
      id: Date.now(),
      author: props.currentUser.name, // Tên người dùng đang đăng nhập
      avatar: props.currentUser.avatar || "https://via.placeholder.com/50", // Ảnh đại diện
      content: newReply.value,
      date: new Date().toISOString(),
    });

    // Cập nhật lại dữ liệu trong localStorage
    saveCommentToLocalStorage(props.post.id, comment);
  }
  replyCommentId.value = null; // Đóng form trả lời
  newReply.value = ""; // Xóa nội dung sau khi gửi
};
</script>
<template>
  <div class="container col-lg-8 border">
    <!-- Header -->
    <header class="mt-3">
      <h3>{{ post.title }}</h3>
      <p class="text-muted mb-2">
        <span
          >Tác giả: <strong>{{ post.author }}</strong></span
        >
        | <span>{{ formatDate(post.date) }}</span>
      </p>
    </header>
    <hr />

    <!-- Nội dung bài viết -->
    <div class="container-fluid p-3 mb-5">
      <h5>{{ post.subtitle }}</h5>
      <img :src="post.image" alt="" class="img-fluid mb-3" width="100%" />
      <p>{{ post.content }}</p>
    </div>

    <!-- Phần nhập bình luận -->
    <div class="bg-secondary p-4 mb-3">
      <h3>Bình luận bài viết tại đây</h3>
      <textarea
        v-model="newComment"
        class="form-control"
        rows="3"
        placeholder="Nhập bình luận của bạn..."
      ></textarea>
      <button @click="addComment" class="btn btn-sm btn-primary mt-3 mb-3">
        Gửi bình luận
      </button>
    </div>

    <!-- Danh sách bình luận -->
    <div class="container my-5">
      <h4>{{ comments.length }} Bình luận</h4>
      <div class="mt-4">
        <div v-for="comment in comments" :key="comment.id" class="d-flex mb-4">
          <!-- Avatar và nội dung bình luận chính -->
          <img
            :src="comment.avatar || 'https://via.placeholder.com/50'"
            alt="User Image"
            class="rounded-circle me-3"
            style="width: 50px; height: 50px"
          />
          <div>
            <h6 class="mb-1">{{ comment.author }}</h6>
            <small class="text-muted">{{ formatDate(comment.date) }}</small>
            <p class="mt-2 mb-3">{{ comment.content }}</p>
            <button
              @click="
                replyCommentId === comment.id
                  ? (replyCommentId = null)
                  : (replyCommentId = comment.id)
              "
              class="btn btn-outline-secondary btn-sm"
            >
              Reply
            </button>

            <!-- Form trả lời bình luận -->
            <div v-if="replyCommentId === comment.id" class="mt-3 ms-5  ">
              <textarea
                v-model="newReply"
                class="form-control"
                rows="2"
                placeholder="Nhập trả lời của bạn..."
              ></textarea>
              <button
                @click="addReply(comment.id)"
                class="btn btn-sm btn-primary mt-2"
              >
                Gửi trả lời
              </button>
            </div>

            <!-- Danh sách trả lời -->
            <div
              v-for="reply in comment.replies"
              :key="reply.id"
              class="d-flex mt-4 ms-3"
            >
              <img
                :src="reply.avatar || 'https://via.placeholder.com/50'"
                alt="User Image"
                class="rounded-circle me-3"
                style="width: 50px; height: 50px"
              />
              <div>
                <h6 class="mb-1">{{ reply.author }}</h6>
                <small class="text-muted">{{ formatDate(reply.date) }}</small>
                <p class="mt-2 mb-3">{{ reply.content }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
