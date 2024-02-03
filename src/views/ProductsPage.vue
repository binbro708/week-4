<template>
  <template v-if="isLogin">
    <div class="alert alert-primary mt-5" role="alert">載入中...</div>
  </template>
  <template v-else>
    <div class="container">
      <div class="row py-3">
        <div class="col-md-7">
          <div class="d-flex justify-content-between">
            <h2>產品列表</h2>
            <button class="btn btn-primary" @click="openModal('new')">
              建立新的產品
            </button>
          </div>

          <table class="table table-hover mt-4">
            <thead>
              <tr>
                <th width="150">產品名稱</th>
                <th width="120">原價</th>
                <th width="120">售價</th>
                <th width="150">是否啟用</th>
                <th width="120">查看細節</th>
                <th width="120">編輯</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="product in products" :key="product.id">
                <td width="150">{{ product.title }}</td>
                <td width="120">
                  {{ product.origin_price }}
                </td>
                <td width="120">
                  {{ product.price }}
                </td>
                <td width="120">
                  <span v-if="product.is_enabled" class="text-success"
                    >啟用</span
                  >
                  <span v-else class="text-danger">未啟用</span>
                </td>
                <td width="120">
                  <button
                    type="button"
                    class="btn btn-primary"
                    @click="choseProduct(product)"
                  >
                    查看細節
                  </button>
                </td>
                <td>
                  <div class="btn-group">
                    <button
                      type="button"
                      class="btn btn-outline-primary btn-sm"
                      @click="openModal('edit', product)"
                    >
                      編輯
                    </button>
                    <button
                      type="button"
                      class="btn btn-outline-danger btn-sm"
                      @click="openModal('delete', product)"
                    >
                      刪除
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>

          <p>
            目前有 <span>{{ products.length }}</span> 項產品
          </p>
          <Pagination :pages="pagination" @emit-pages="getData" />
        </div>
        <div class="col-md-5">
          <h2>單一產品細節</h2>
          <template v-if="tempProduct.title">
            <div class="card mb-3">
              <img
                class="card-img-top primary-image"
                alt="主圖"
                :src="tempProduct.imagesUrl"
              />
              <div class="card-body">
                <h5 class="card-title">
                  {{ tempProduct.title }}
                  <span class="badge bg-primary ms-2">{{
                    tempProduct.category
                  }}</span>
                </h5>
                <p class="card-text">商品描述：{{ tempProduct.description }}</p>
                <p class="card-text">商品內容：{{ tempProduct.content }}</p>
                <div class="d-flex">
                  <p class="card-text me-2">{{ tempProduct.price }}</p>
                  <p class="card-text text-secondary">
                    <del>{{ tempProduct.origin_price }}</del>
                  </p>
                  {{ tempProduct.unit }} / 元
                </div>
              </div>
            </div>
            <template
              v-for="(img, index) in tempProduct.imagesUrl"
              :key="index"
            >
              <img v-if="img" :src="img" :alt="index + 1" class="images m-2" />
            </template>
          </template>
          <p v-else class="text-secondary">請選擇一個商品查看</p>
        </div>
      </div>
    </div>
  </template>
  <ProductsModal
    :product="tempProduct"
    :is-add="isAdd"
    :product-modal="productModal"
    @update="getData"
  />
  <DelProductsModal
    :product="tempProduct"
    :del-product-modal="delProductModal"
    @update="getData"
  />
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import { Modal } from "bootstrap";
const router = useRouter();
import ProductsModal from "../components/ProductsModal.vue";
import DelProductsModal from "../components/DelProductsModal.vue";
import Pagination from "../components/Pagination.vue";

const apiPath = "bintest";
const isLogin = ref(false);
const products = ref([]);
const isAdd = ref(false);
const pagination = ref({});

const tempProduct = ref({
  imagesUrl: [],
});

let productModal = ref(null);
let delProductModal = ref(null);

onMounted(() => {
  isLogin.value = true;

  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)token\s*=\s*([^;]*).*$)|^.*$/,
    "$1"
  );
  //把token塞進header裡面去驗證用
  axios.defaults.headers.common.Authorization = token;

  checkIsLogin();

  productModal.value = new Modal(document.getElementById("productModal"), {
    keyboard: false,
  });

  delProductModal.value = new Modal(
    document.getElementById("delProductModal"),
    {
      keyboard: false,
    }
  );
});

//確認login狀態
const checkIsLogin = () => {
  axios
    .post("https://vue3-course-api.hexschool.io/v2/api/user/check")
    .then(() => {
      getData();
    })
    .catch((err) => {
      alert(err.response.data.message);
      //沒登入踢回去登入
      router.push("/");
    });
};

//取得產品資料
const getData = (page = 1) => {
  isLogin.value = true;

  const url = `https://vue3-course-api.hexschool.io/v2/api/${apiPath}/admin/products?page=${page}`;
  axios
    .get(url)
    .then((response) => {
      isLogin.value = false;
      const { products: resProducts, pagination: resPagination } =
        response.data;

      products.value = resProducts;
      pagination.value = resPagination;
    })
    .catch((err) => {
      alert(err.response.data.message);
    });
};

//選擇產品
const choseProduct = (product) => {
  tempProduct.value = product;
};

//開彈窗
const openModal = (isNew, product) => {
  if (isNew === "new") {
    tempProduct.value = {
      imagesUrl: [],
    };
    isAdd.value = true;
    productModal.value.show();
  } else if (isNew === "edit") {
    tempProduct.value = { ...product };
    isAdd.value = false;
    productModal.value.show();
  } else if (isNew === "delete") {
    tempProduct.value = { ...product };
    delProductModal.value.show();
  }
};

//新增or編輯

//刪除產品
</script>

<style scoped>
img {
  object-fit: contain;
  max-width: 100%;
}

.primary-image {
  height: 300px;
}

.images {
  height: 150px;
}
</style>
