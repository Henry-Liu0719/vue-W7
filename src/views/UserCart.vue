<script setup>
import axios from 'axios';

import { useToastMessageStore } from "@/stores/toastMessage";

import { ref } from 'vue';
import { useRouter } from 'vue-router';

const toastMessageStore = useToastMessageStore()
const { pushMessage }  = toastMessageStore

const router = useRouter();

const products = ref([]);
const product = ref({});
const status = ref({
  loadingItem: '',
});

const formRef = ref(null);

const form = ref({
  user: {
    name: '',
    email: '',
    tel: '',
    address: '',
  },
  message: '',
});
const cart = ref({});
const isLoading = ref(false);
const coupon_code = ref('');

const getProducts = () => {
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/products`;
  isLoading.value = true;
  axios.get(url).then((response) => {
    products.value = response.data.products;
    isLoading.value = false;
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '錯誤訊息',
      content: error.response.data.message,
    })
  });
};

const getProduct = (id) => {
  router.push(`/user/product/${id}`);
};

const addToCart = (id, qty = 1) => {
  isLoading.value = true;
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/cart`;
  status.value.loadingItem = id;
  const cart = {
    product_id: id,
    qty,
  };

  axios.post(url, { data: cart }).then((response) => {
    status.value.loadingItem = '';
    isLoading.value = false;
    pushMessage({
      style: 'success',
      title: '加入購物車',
      content: response.data.message,
    })
    getCart();
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '加入購物車',
      content: error.response.data.message,
    })
  });
};

const deleteAllCarts = () => {
  isLoading.value = true;
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/carts`;
  axios.delete(url).then((response) => {
    pushMessage({
      style: 'success',
      title: '清除購物車',
      content: response.data.message,
    })
    getCart();
    isLoading.value = false;
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '清除購物車',
      content: error.response.data.message,
    })
  });
};

const getCart = () => {
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/cart`;
  isLoading.value = true;
  axios.get(url).then((response) => {
    cart.value = response.data.data;
    isLoading.value = false;
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '錯誤訊息',
      content: error.response.data.message,
    })
  });
};

const removeCartItem = (id) => {
  status.value.loadingItem = id;
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/cart/${id}`;
  isLoading.value = true;
  axios.delete(url).then((response) => {
    pushMessage({
      style: 'success',
      title: '移除購物車品項',
      content: response.data.message,
    })
    status.value.loadingItem = '';
    isLoading.value = false;
    getCart();
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '移除購物車品項',
      content: error.response.data.message,
    })
  });
};

const updateCart = (data) => {
  isLoading.value = true;
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/cart/${data.id}`;
  const cart = {
    product_id: data.product_id,
    qty: data.qty,
  };

  axios.put(url, { data: cart }).then((response) => {
    isLoading.value = false;
    pushMessage({
      style: 'success',
      title: '更新購物車',
      content: response.data.message,
    })
    getCart();
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '更新購物車',
      content: error.response.data.message,
    })
  });
}

const addCouponCode = () => {
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/coupon`;
  const coupon = {
    code: coupon_code.value,
  };
  isLoading.value = true;
  axios.post(url, { data: coupon }).then((response) => {
    pushMessage({
      style: 'success',
      title: '加入優惠券',
      content: response.data.message,
    })
    getCart();
    isLoading.value = false;
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '加入優惠券',
      content: error.response.data.message,
    })
  });
};

const createOrder = () => {
  isLoading.value = true;
  const url = `${import.meta.env.VITE_API}/api/${import.meta.env.VITE_PATH}/order`;
  axios.post(url, { data: form.value }).then((response) => {
    pushMessage({
      style: 'success',
      title: '建立訂單',
      content: response.data.message,
    })
    router.push(`/user/checkout/${response.data.orderId}`);
    formRef.value.resetForm();
    isLoading.value = false;
  }).catch((error) => {
    isLoading.value = false;
    pushMessage({
      style: 'danger',
      title: '建立訂單',
      content: error.response.data.message,
    })
  });
};

getProducts();
getCart();
</script>

<template>
  <VueLoading :active="isLoading" :z-index="1060" />
  <div class="container">
    <div class="mt-4">
      <!-- 產品列表 -->
      <table class="table align-middle">
        <thead>
          <tr>
            <th>圖片</th>
            <th>商品名稱</th>
            <th>價格</th>
            <th>功能</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in products" :key="item.id">
            <td style="width: 200px">
              <div style="
                  height: 100px;
                  background-size: cover;
                  background-position: center;
                  " :style="{ backgroundImage: `url(${item.imageUrl})` }"></div>
            </td>
            <td>
              <a href="#" class="text-dark">{{ item.title }}</a>
            </td>
            <td>
              <div class="h5" v-if="!item.price">
                {{ item.origin_price }} 元
              </div>
              <del class="h6" v-if="item.price">原價 {{ item.origin_price }} 元</del>
              <div class="h5" v-if="item.price">
                現在只要 {{ item.price }} 元
              </div>
            </td>
            <td>
              <div class="btn-group btn-group-sm">
                <button type="button" class="btn btn-outline-secondary"
                  :disabled="status.loadingItem === item.id || !item.is_enabled" @click="getProduct(item.id)">
                  <span class="spinner-border spinner-grow-sm" v-if="status.loadingItem === item.id"></span>
                  查看更多
                </button>
                <button type="button" class="btn btn-outline-danger" @click="addToCart(item.id)"
                  :disabled="status.loadingItem === item.id || !item.is_enabled">
                  <span class="spinner-border spinner-grow-sm" v-if="status.loadingItem === item.id"></span>
                  加到購物車
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
      <!--購物車列表 -->
      <div class="text-end">
        <button class="btn btn-outline-danger" type="button" @click="deleteAllCarts">
          清空購物車
        </button>
      </div>
      <table class="table align-middle">
        <thead>
          <tr>
            <th></th>
            <th>品名</th>
            <th style="width: 110px">數量</th>
            <th>單價</th>
          </tr>
        </thead>
        <tbody>
          <template v-if="cart.carts">
            <tr v-for="item in cart.carts" :key="item.id">
              <td>
                <button type="button" class="btn btn-outline-danger btn-sm" :disabled="status.loadingItem === item.id"
                  @click="removeCartItem(item.id)">
                  <i class="bi bi-x"></i>
                </button>
              </td>
              <td>
                {{ item.product.title }}
                <div class="text-success" v-if="item.coupon">已套用優惠券</div>
              </td>
              <td>
                <div class="input-group input-group-sm">
                  <input type="number" class="form-control" min="1" v-model.number="item.qty" @blur="updateCart(item)" />
                  <div class="input-group-text">/ {{ item.product.unit }}</div>
                </div>
              </td>
              <td class="text-end">
                <small v-if="cart.final_total !== cart.total" class="text-success">折扣價：</small>
                {{ $filters.currency(item.final_total) }}
              </td>
            </tr>
          </template>
        </tbody>
        <tfoot>
          <tr>
            <td colspan="3" class="text-end">總計</td>
            <td class="text-end">{{ $filters.currency(cart.total) }}</td>
          </tr>
          <tr v-if="cart.final_total !== cart.total">
            <td colspan="3" class="text-end text-success">折扣價</td>
            <td class="text-end text-success">
              {{ $filters.currency(cart.final_total) }}
            </td>
          </tr>
        </tfoot>
      </table>
      <div class="input-group mb-3 input-group-sm">
        <input type="text" class="form-control" v-model="coupon_code" placeholder="請輸入優惠碼" />
        <div class="input-group-append">
          <button class="btn btn-outline-secondary" type="button" @click="addCouponCode">
            套用優惠碼
          </button>
        </div>
      </div>
    </div>

    <div class="my-5 row justify-content-center">
      <VeeForm ref="formRef" class="col-md-6" v-slot="{ errors }" @submit="createOrder">
        <div class="mb-3">
          <label for="email" class="form-label">Email</label>
          <VeeField id="email" name="email" type="email" class="form-control" :class="{ 'is-invalid': errors['email'] }"
            placeholder="請輸入 Email" rules="email|required" v-model="form.user.email" />
          <ErrorMessage name="email" class="invalid-feedback" />
        </div>

        <div class="mb-3">
          <label for="name" class="form-label">收件人姓名</label>
          <VeeField id="name" name="姓名" type="text" class="form-control" :class="{ 'is-invalid': errors['姓名'] }"
            placeholder="請輸入姓名" rules="required" v-model="form.user.name" />
          <ErrorMessage name="姓名" class="invalid-feedback" />
        </div>

        <div class="mb-3">
          <label for="tel" class="form-label">收件人電話</label>
          <VeeField id="tel" name="電話" type="tel" class="form-control" :class="{ 'is-invalid': errors['電話'] }"
            placeholder="請輸入電話" rules="required|min:8|max:10" v-model="form.user.tel" />
          <ErrorMessage name="電話" class="invalid-feedback" />
        </div>

        <div class="mb-3">
          <label for="address" class="form-label">收件人地址</label>
          <VeeField id="address" name="地址" type="text" class="form-control" :class="{ 'is-invalid': errors['地址'] }"
            placeholder="請輸入地址" rules="required" v-model="form.user.address" />
          <ErrorMessage name="地址" class="invalid-feedback" />
        </div>

        <div class="mb-3">
          <label for="message" class="form-label">留言</label>
          <textarea name="" id="message" class="form-control" cols="30" rows="10" v-model="form.message"></textarea>
        </div>
        <div class="text-end">
          <button type="submit" class="btn btn-danger">送出訂單</button>
        </div>
      </VeeForm>
    </div>
  </div>
</template>
