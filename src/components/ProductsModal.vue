<template>
  <!-- Modal -->
  <div
    id="productModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="productModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-xl">
      <div class="modal-content border-0">
        <div class="modal-header bg-dark text-white">
          <h5 id="productModalLabel" class="modal-title">
            <span v-if="isAdd">新增產品</span>
            <span v-else>編輯產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="row">
            <div class="col-sm-4">
              <div class="mb-3">
                <label for="imageUrl" class="form-label">主要圖片</label>
                <input
                  v-model="product.imageUrl"
                  type="text"
                  class="form-control mb-2"
                  placeholder="請輸入圖片連結"
                />
                <img class="img-fluid" :src="product.imageUrl" />
              </div>
              <h3 class="mb-3">多圖新增</h3>
              <!-- 我多圖陣列裡有東西才顯示(有按新增圖片) -->
              <div v-if="Array.isArray(product.imagesUrl)">
                <div
                  class="mb-1"
                  v-for="(image, index) in product.imagesUrl"
                  :key="index"
                >
                  <div class="mb-3">
                    <label for="imageUrl" class="form-label">圖片網址</label>
                    <input
                      v-model="product.imagesUrl[index]"
                      type="text"
                      class="form-control"
                      placeholder="請輸入圖片連結"
                    />
                  </div>
                  <img class="img-fluid" :src="image" />
                </div>
                <!-- 新增刪除會一直在最下面，不是每個圖各別控制，要刪只會刪最後一筆 -->
                <div
                  v-if="
                    !product.imagesUrl.length ||
                    product.imagesUrl[product.imagesUrl.length - 1]
                  "
                >
                  <button
                    class="btn btn-outline-primary btn-sm d-block w-100"
                    @click="product.imagesUrl.push('')"
                  >
                    新增圖片
                  </button>
                </div>
                <div v-else>
                  <!-- 因為有推空的，所以沒資料時也是要刪掉 -->
                  <button
                    class="btn btn-outline-danger btn-sm d-block w-100"
                    @click="product.imagesUrl.pop()"
                  >
                    刪除圖片
                  </button>
                </div>
              </div>
              <div v-else>
                <button
                  class="btn btn-outline-primary btn-sm d-block w-100"
                  @click="createImages"
                >
                  新增圖片
                </button>
              </div>
            </div>

            <div class="col-sm-8">
              <div class="mb-3">
                <label for="title" class="form-label">標題</label>
                <input
                  id="title"
                  v-model="product.title"
                  type="text"
                  class="form-control"
                  placeholder="請輸入標題"
                />
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="category" class="form-label">分類</label>
                  <input
                    id="category"
                    v-model="product.category"
                    type="text"
                    class="form-control"
                    placeholder="請輸入分類"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">單位</label>
                  <input
                    id="unit"
                    v-model="product.unit"
                    type="text"
                    class="form-control"
                    placeholder="請輸入單位"
                  />
                </div>
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="origin_price" class="form-label">原價</label>
                  <input
                    id="origin_price"
                    v-model.number="product.origin_price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入原價"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">售價</label>
                  <input
                    id="price"
                    v-model.number="product.price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入售價"
                  />
                </div>
              </div>
              <hr />

              <div class="mb-3">
                <label for="description" class="form-label">產品描述</label>
                <textarea
                  id="description"
                  v-model="product.description"
                  type="text"
                  class="form-control"
                  placeholder="請輸入產品描述"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <label for="content" class="form-label">說明內容</label>
                <textarea
                  id="description"
                  v-model="product.content"
                  type="text"
                  class="form-control"
                  placeholder="請輸入說明內容"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <div class="form-check">
                  <input
                    id="is_enabled"
                    v-model="product.is_enabled"
                    class="form-check-input"
                    type="checkbox"
                    :true-value="1"
                    :false-value="0"
                  />
                  <label class="form-check-label" for="is_enabled"
                    >是否啟用</label
                  >
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button type="button" class="btn btn-primary" @click="updateProduct">
            確認
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal -->
</template>

<script setup>
//拿外層傳進來的product跟isAdd
const props = defineProps(["product", "isAdd", "productModal"]);
const emit = defineEmits(["update"]);
import axios from "axios";
const apiPath = "bintest";

const updateProduct = () => {
  //新增
  let url = `https://vue3-course-api.hexschool.io/v2/api/${apiPath}/admin/product`;
  let http = "post";
  //編輯
  if (!props.isAdd) {
    url = `https://vue3-course-api.hexschool.io/v2/api/${apiPath}/admin/product/${props.product.id}`;
    http = "put";
  }

  axios[http](url, { data: props.product })
    .then((response) => {
      alert(response.data.message);
      //新增成功關彈窗
      props.productModal.hide();
      //重拿一次資料 也能直接推到原本資料裡面可以少打一次api
      //要去觸發外層的getData去update 原本是直接重打getData
      emit("update");
    })
    .catch((err) => {
      alert(err.response.data.message);
    });
};

const createImages = () => {
  props.product.imagesUrl = [];
  //推一筆空的 Array.isArray為true 也讓之後的v-modal直接把他替換掉 用index去看要換哪筆(邏輯上是會照順序)
  props.product.imagesUrl.push("");
};
</script>

<style lang="scss" scoped></style>
