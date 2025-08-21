<script setup>
import { ref, onMounted, watch } from "vue";
import axios from "axios";

const emit = defineEmits(["close-basket"]);
const props = defineProps({
  loginStatus: Object,
});

const currentUser = ref(null);
const users = ref([]);
const error = ref(null);
const baskets = ref([]);
const basketTotal = ref(0); // сумма корзины

// Получение списка пользователей
const fetchUsers = async () => {
  error.value = null;
  try {
    const response = await axios.get(
      "https://30a5b8072c05076b.mokky.dev/users"
    );
    users.value = response.data;
    currentUser.value = users.value.find(
      (user) => user.username === props.loginStatus?.username
    );
    if (currentUser.value) {
      // console.log("Найден пользователь:", currentUser.value);
      fetchBasket();
    } else {
      // console.log("Пользователь не найден");
    }
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при получении данных:", err.message);
  }
};

// Получение корзины
const fetchBasket = async () => {
  try {
    const response = await axios.get(
      "https://30a5b8072c05076b.mokky.dev/sneakers"
    );
    const sneakers = response.data;
    if (currentUser.value?.basket?.length) {
      baskets.value = sneakers.filter((s) =>
        currentUser.value.basket.includes(s.id)
      );
    }
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при получении данных:", err.message);
  }
};

// Удаление из корзины
const removeBasket = async (id) => {
  try {
    baskets.value = baskets.value.filter((item) => item.id !== id);
    currentUser.value.basket = currentUser.value.basket.filter(
      (fid) => fid !== id
    );
    await axios.patch(
      `https://30a5b8072c05076b.mokky.dev/users/${currentUser.value.id}`,
      { basket: currentUser.value.basket }
    );
    // console.log(`Удалён из корзины: ${id}`);
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при удалении:", err.message);
  }
};

// Подсчёт суммы корзины
const findBasketCost = async () => {
  try {
    const response = await axios.get(
      "https://30a5b8072c05076b.mokky.dev/stock"
    );
    const stock = response.data;

    // считаем сумму всех товаров
    let total = baskets.value.reduce((sum, basketItem) => {
      const product = stock.find((s) => s.id === basketItem.id);
      return product ? sum + product.cost : sum;
    }, 0);

    basketTotal.value = total;
    // console.log("Общая сумма корзины:", total);
  } catch (err) {
    console.error("Ошибка при получении данных:", err.message);
  }
};

// следим за изменениями корзины и пересчитываем стоимость
watch(
  () => baskets.value,
  () => {
    if (baskets.value.length) {
      findBasketCost();
    } else {
      basketTotal.value = 0;
    }
  },
  { deep: true }
);

// Вызов при монтировании
onMounted(() => {
  if (props.loginStatus?.username) {
    fetchUsers();
  }
});

// Получение путей для изображений
const getImageUrl = (id, color, imgnumber) => {
  const imgURL = `/assets/${id}/${color}-${imgnumber}.jpg`;
  return new URL(imgURL, import.meta.url).href;
};

const addToHistory = async () => {
  if (!currentUser.value) {
    alert("Сначала войдите в аккаунт");
    return;
  }

  if (!currentUser.value.basket?.length) {
    alert("Корзина пуста");
    return;
  }

  try {
    // 1) старый history или пустой массив
    const prevHistory = Array.isArray(currentUser.value.history)
      ? currentUser.value.history
      : [];

    // 2) новые id из корзины
    const newHistory = [...prevHistory, ...currentUser.value.basket];

    // 3) PATCH запрос: обновляем history и чистим basket
    await axios.patch(
      `https://30a5b8072c05076b.mokky.dev/users/${currentUser.value.id}`,
      {
        history: newHistory,
        basket: [],
      }
    );

    // 4) локально обновляем состояние
    currentUser.value.history = newHistory;
    currentUser.value.basket = [];
    baskets.value = [];
    basketTotal.value = 0;

    alert("Покупка оформлена, товары добавлены в историю!");
  } catch (err) {
    console.error(err);
    alert("Ошибка при оформлении покупки!");
  }
};
</script>

<template>
  <div class="basket">
    <div class="basket-inner">
      <div class="basket-holder">
        <div
          class="basket-item"
          v-for="item in baskets"
          :key="item.id"
          v-if="baskets.length !== 0"
        >
          <div class="basket-name">
            <p>{{ item.name }}</p>
          </div>
          <button class="basket-remove" @click="removeBasket(item.id)">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="200"
              height="200"
              viewBox="0 0 24 24"
            >
              <path
                fill="none"
                stroke="currentColor"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M18 6L6 18M6 6l12 12"
              />
            </svg>
          </button>
          <div class="basket-left">
            <img :src="getImageUrl(item.id, item.colors[0], 1)" />
          </div>
          <div class="basket-right">
            <p v-for="(col, index) in item.colors" :key="col">
              {{ index === 0 ? col : `, ${col}` }}
            </p>
          </div>
        </div>
        <p v-else style="width: 100%; text-align: center">
          Log in or add something to the basket
        </p>
      </div>
      <button class="buy" @click="addToHistory" v-if="basketTotal !== 0">
        {{ `Buy your basket for ${basketTotal}$` }}
      </button>
    </div>
    <div class="else" @click="$emit('close-basket')"></div>
  </div>
</template>

<style scoped>
.basket {
  max-width: 1920px;
  width: 100vw;
  height: 100%;
  position: fixed;
  display: flex;
  flex-direction: row-reverse;
  z-index: 15;
  top: 0;
  background: #00000020;
  transition: 0.4s;
}
.basket-inner {
  max-width: 960px;
  width: 50%;
  background: var(--main-bg);
  border-right: solid 1px var(--sub-text);
  padding: calc(var(--main-pad) / 4) calc(var(--main-pad) / 2);
  display: flex;
  flex-direction: column;
  gap: 2%;
  transition: 0.4s;
}
.basket-holder {
  width: 100%;
  height: fit-content;
  display: flex;
  flex-wrap: wrap;
  gap: 2%;
  row-gap: 12px;
}
.basket-item {
  width: 49%;
  height: 120px;
  background: var(--card);
  border-radius: 8px;
  padding: 8px;
  display: flex;
  position: relative;
  gap: 8px;
  box-shadow: rgba(0, 0, 0, 0) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
  transition: 0.4s;
}
.basket-item:hover {
  box-shadow: rgba(0, 0, 0, 0.1) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}
.basket-name {
  position: absolute;
  z-index: 3;
  white-space: nowrap;
}
.basket-remove {
  position: absolute;
  z-index: 3;
  right: 12px;
}
.basket-remove svg {
  width: 24px;
  height: 24px;
}
.basket-left {
  height: 100%;
  aspect-ratio: 1 / 1;
  overflow: hidden;
  position: relative;
  background: #000;
}
.basket-left img {
  width: 100%;
  position: absolute;
  bottom: 0;
  width: fit-content;
}
.basket-right {
  flex: 1;
  height: fit-content;
  margin: auto 0;
  display: flex;
  gap: 4px;
  flex-wrap: wrap;
}
.basket-right p {
  width: fit-content;
  height: fit-content;
  font-size: 14px;
}
.basket-right button {
  width: 100%;
  padding: 2px 8px;
  border-radius: 4px;
  background: var(--main-text);
  color: var(--main-bg);
}
/*  */
.else {
  flex: 1;
  height: 100%;
}
.left50vw {
  left: -50vw;
}
.buy {
  width: fit-content;
  color: var(--main-bg);
  background: var(--main-text);
  padding: 4px 32px;
  margin: 0 auto;
  border-radius: 4px;
}
</style>
