<template>
  <!-- Modal chỉnh sửa thông tin -->
  <div
    v-if="isModalVisible"
    class="modal fade show"
    tabindex="-1"
    aria-labelledby="exampleModalLabel"
    aria-hidden="false"
    style="display: block"
  >
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h1 class="modal-title fs-5" id="exampleModalLabel">Hồ sơ của tôi</h1>
          <button
            type="button"
            class="btn-close"
            @click="closeModal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="container-fluid align-items-center">
            <form @submit.prevent="updateUser" class="row gap-5">
              <div class="col-sm-7">
                <!-- Họ và tên -->
                <label class="form-label" for="nameuser">Họ và tên</label>
                <input
                  v-model="localUser.name"
                  class="form-control form-control-sm"
                  type="text"
                  name="nameuser"
                  id="nameuser"
                  placeholder="Nhập họ và tên"
                />
                <small v-if="errors.name" class="text-danger">{{
                  errors.name
                }}</small>

                <!-- Email -->
                <label class="form-label" for="email">Email</label>
                <input
                  v-model="localUser.email"
                  class="form-control form-control-sm"
                  type="email"
                  name="email"
                  id="email"
                  placeholder="Nhập email"
                />
                <small v-if="errors.email" class="text-danger">{{
                  errors.email
                }}</small>

                <!-- Mật khẩu -->
                <label class="form-label" for="password">Mật khẩu</label>
                <input
                  v-model="localUser.password"
                  class="form-control form-control-sm"
                  type="password"
                  name="password"
                  id="password"
                  placeholder="Nhập mật khẩu"
                />
                <small v-if="errors.password" class="text-danger">{{
                  errors.password
                }}</small>

                <!-- Xác nhận mật khẩu -->
                <label class="form-label" for="confirmPassword"
                  >Xác nhận mật khẩu</label
                >
                <input
                  v-model="localUser.confirmPassword"
                  class="form-control form-control-sm"
                  type="password"
                  name="confirmPassword"
                  id="confirmPassword"
                  placeholder="Xác nhận mật khẩu"
                />
                <small v-if="errors.confirmPassword" class="text-danger">{{
                  errors.confirmPassword
                }}</small>
              </div>

              <!-- Ảnh đại diện -->
              <div class="col-sm-4 text-center">
                <div
                  class="avatar-container"
                  :style="{
                    backgroundImage: localUser.profileImage
                      ? 'url(' + localUser.profileImage + ')'
                      : '',
                    backgroundColor: !localUser.profileImage ? '#f0f0f0' : '',
                    color: !localUser.profileImage ? '#777' : '',
                  }"
                  style="
                    width: 200px;
                    height: 200px;
                    background-size: cover;
                    background-position: center;
                    border-radius: 50%;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    font-size: 2rem;
                    text-align: center;
                    margin-left: 10px;
                  "
                >
                  <span v-if="!localUser.profileImage">Chưa có ảnh</span>
                </div>
                <div class="mt-3">
                  <label for="fileInput" class="form-label">Hình ảnh</label>
                  <input
                    type="file"
                    class="form-control form-control-sm"
                    id="fileInput"
                    @change="handleImageChange"
                  />
                </div>
              </div>
            </form>
          </div>
        </div>

        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="closeModal">
            Đóng
          </button>
          <button type="button" class="btn btn-primary" @click="updateUser">
            Lưu
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";
import Swal from "sweetalert2";

// Nhận props từ component cha
const props = defineProps({
  isModalVisible: Boolean,
  currentUser: Object,
});

// Định nghĩa emit để gọi các sự kiện
const emit = defineEmits(["closeModal", "saveChanges"]);

// Tạo đối tượng `localUser` để lưu thông tin người dùng tạm thời
const localUser = reactive({ ...props.currentUser });
console.log(localUser);
const errors = reactive({
  name: "",
  email: "",
  password: "",
  confirmPassword: "",
});

// Hàm kiểm tra tính hợp lệ của form
function validateForm() {
  let isValid = true;
  if (!localUser.name) {
    errors.name = "Họ và tên không được để trống.";
    isValid = false;
  } else {
    errors.name = "";
  }

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!localUser.email) {
    errors.email = "Email không được để trống.";
    isValid = false;
  } else if (!emailRegex.test(localUser.email)) {
    errors.email = "Email không hợp lệ.";
    isValid = false;
  } else {
    errors.email = "";
  }

  if (!localUser.password) {
    errors.password = "Mật khẩu không được để trống.";
    isValid = false;
  } else if (localUser.password.length < 6) {
    errors.password = "Mật khẩu phải có ít nhất 6 ký tự.";
    isValid = false;
  } else {
    errors.password = "";
  }

  if (!localUser.confirmPassword) {
    errors.confirmPassword = "Xác nhận mật khẩu không được để trống.";
    isValid = false;
  } else if (localUser.password !== localUser.confirmPassword) {
    errors.confirmPassword = "Mật khẩu không khớp.";
    isValid = false;
  } else {
    errors.confirmPassword = "";
  }

  return isValid;
}

// Hàm gọi khi cập nhật thông tin người dùng
// Lưu thông tin người dùng vào localStorage và emit sự kiện
function updateUser() {
  console.log("Thông tin người dùng trước khi gửi:", localUser);
  if (validateForm()) {
    // Emit thông tin người dùng đã được thay đổi
    emit("saveChanges", { ...localUser }); // Phát toàn bộ dữ liệu đã chỉnh sửa
    console.log("Sự kiện saveChanges đã được phát ra!");
    // // Lưu thông tin vào localStorage
    // localStorage.setItem("user", JSON.stringify(props.localUser));

    Swal.fire({
      icon: "success",
      title: "Cập nhật thành công!",
      text: "Thông tin của bạn đã được cập nhật.",
      confirmButtonText: "OK",
    });

    closeModal();
  }
}

// Hàm xử lý khi người dùng thay đổi ảnh đại diện
function handleImageChange(event) {
  const file = event.target.files[0];
  if (file) {
    const allowedTypes = ["image/jpeg", "image/png", "image/gif"];
    if (!allowedTypes.includes(file.type)) {
      Swal.fire({
        icon: "error",
        title: "Lỗi",
        text: "Chỉ hỗ trợ các định dạng ảnh JPEG, PNG, GIF.",
      });
      return;
    }

    if (file.size > 5000000) {
      // Giới hạn 5MB
      Swal.fire({
        icon: "error",
        title: "Lỗi",
        text: "Ảnh quá lớn. Vui lòng chọn ảnh nhỏ hơn 5MB.",
      });
      return;
    }

    const reader = new FileReader();
    reader.onload = function (e) {
      localUser.profileImage = e.target.result;
    };
    reader.readAsDataURL(file);
  }
}

// Đóng modal
function closeModal() {
  emit("closeModal");
}
</script>
