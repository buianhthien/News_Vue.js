<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";

const props = defineProps({
  post: Object,
  currentUser: Object,
  posts: Array, // Danh sách bài viết hiện có
});

const emit = defineEmits(["savePost", "updatePosts"]);

const title = ref(props.post ? props.post.title : "");
const content = ref(props.post ? props.post.content : "");
const image = ref(props.post ? props.post.image : null);
const previewImage = ref(
  props.post && props.post.image ? props.post.image : ""
);

// Xử lý chọn hình ảnh
function handleImageUpload(event) {
  const file = event.target.files[0];
  if (file) {
    previewImage.value = URL.createObjectURL(file);
    image.value = file;
  }
}

// Kiểm tra tính hợp lệ của form
function validateForm() {
  if (!title.value.trim()) {
    Swal.fire({
      icon: "error",
      title: "Lỗi",
      text: "Vui lòng nhập tiêu đề.",
    });
    return false;
  }
  if (!content.value.trim()) {
    Swal.fire({
      icon: "error",
      title: "Lỗi",
      text: "Vui lòng nhập nội dung.",
    });
    return false;
  }
  if (!image.value && !props.post) {
    Swal.fire({
      icon: "error",
      title: "Lỗi",
      text: "Vui lòng chọn hình ảnh.",
    });
    return false;
  }

  // Kiểm tra trùng tiêu đề
  const isDuplicateTitle = props.posts.some(
    (existingPost) =>
      existingPost.title === title.value &&
      (!props.post || existingPost.id !== props.post.id) // Loại trừ bài viết đang chỉnh sửa
  );
  if (isDuplicateTitle) {
    Swal.fire({
      icon: "error",
      title: "Lỗi",
      text: "Tiêu đề bài viết đã tồn tại. Vui lòng chọn tiêu đề khác.",
    });
    return false;
  }

  return true;
}

function handleSubmit() {
  if (!validateForm()) return;

  if (!props.currentUser || !props.currentUser.id) {
    Swal.fire({
      icon: "error",
      title: "Lỗi",
      text: "Không tìm thấy thông tin người dùng.",
    });
    return;
  }

  const newPost = {
    id: props.post ? props.post.id : Date.now(),
    title: title.value,
    content: content.value,
    image: previewImage.value,
    author: props.currentUser.name,
    userId: props.currentUser.id, // Gán userId đúng từ currentUser
    date: new Date().toISOString(), // Thêm ngày hiện tại dưới dạng chuỗi ISO 8601
  };
  emit("updatePosts", newPost);
  emit("savePost", "Profile");
  console.log("Bài viết.........", newPost);
  console.log("currentUser..............", props.currentUser);
  Swal.fire({
    icon: "success",
    title: "Thành công",
    text: "Bài viết đã được lưu thành công!",
  });

  resetForm();
}

// Reset form
function resetForm() {
  if (props.post) {
    title.value = props.post.title;
    content.value = props.post.content;
    previewImage.value = props.post.image || "";
    image.value = null;
  } else {
    title.value = "";
    content.value = "";
    previewImage.value = "";
    image.value = null;
  }
}
</script>

<template>
  <div class="container">
    <h2>{{ props.post ? "Chỉnh sửa bài viết" : "Thêm bài viết" }}</h2>
    <div class="row">
      <div class="col-md-4 text-center">
        <div class="mb-3">
          <label for="image" class="form-label">Hình ảnh</label>
          <input
            type="file"
            id="image"
            class="form-control"
            accept="image/*"
            @change="handleImageUpload"
          />
          <img
            v-if="previewImage"
            :src="previewImage"
            class="img-fluid mt-3 border"
          />
        </div>
      </div>
      <div class="col-md-8">
        <form @submit.prevent="handleSubmit">
          <div class="mb-3">
            <label for="title" class="form-label">Tiêu đề</label>
            <input
              type="text"
              id="title"
              class="form-control"
              v-model="title"
            />
          </div>
          <div class="mb-3">
            <label for="content" class="form-label">Nội dung</label>
            <textarea
              id="content"
              class="form-control"
              v-model="content"
              rows="5"
            ></textarea>
          </div>
          <button type="submit" class="btn btn-primary">Lưu</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
img {
  max-width: 100%;
  max-height: 300px;
  object-fit: cover;
}
</style>
