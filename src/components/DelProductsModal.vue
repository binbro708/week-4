<template>
  <div
    id="delProductModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="delProductModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content border-0">
        <div class="modal-header bg-danger text-white">
          <h5 id="delProductModalLabel" class="modal-title">
            <span>刪除產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          是否刪除
          <strong class="text-danger">{{ product.title }}</strong>
          商品(刪除後將無法恢復)。
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button type="button" class="btn btn-danger" @click="delProduct">
            確認刪除
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";

const props = defineProps(["product", "delProductModal"]);
const emit = defineEmits(["update"]);
const delProduct = () => {
  const apiPath = "bintest";

  const url = `https://vue3-course-api.hexschool.io/v2/api/${apiPath}/admin/product/${props.product.id}`;

  axios
    .delete(url)
    .then((response) => {
      alert(response.data.message);
      props.delProductModal.hide();
      //刪除後需清空防止留著原本選擇的產品
      props.product = {
        imagesUrl: [],
      };
      emit("update");
    })
    .catch((err) => {
      alert(err.response.data.message);
    });
};
</script>

<style lang="scss" scoped></style>
