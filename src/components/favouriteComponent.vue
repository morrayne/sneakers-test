<script setup>
import { ref, onMounted, watch } from "vue";
import axios from "axios";

const emit = defineEmits(["close-favourite"]);
const props = defineProps({ loginStatus: Object });

const currentUser = ref(null);
const users = ref([]);
const error = ref(null);
const favs = ref([]);

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
      fetchFavourites();
    }
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при получении данных:", err.message);
  }
};

// Получение списка избранных кроссовок
const fetchFavourites = async () => {
  try {
    const response = await axios.get(
      "https://30a5b8072c05076b.mokky.dev/sneakers"
    );
    const sneakers = response.data;
    if (currentUser.value?.favourite?.length) {
      favs.value = sneakers.filter((s) =>
        currentUser.value.favourite.includes(s.id)
      );
    }
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при получении данных:", err.message);
  }
};

// Удаление из избранного
const removeFavourite = async (id) => {
  try {
    favs.value = favs.value.filter((item) => item.id !== id);
    currentUser.value.favourite = currentUser.value.favourite.filter(
      (fid) => fid !== id
    );
    await axios.patch(
      `https://30a5b8072c05076b.mokky.dev/users/${currentUser.value.id}`,
      { favourite: currentUser.value.favourite }
    );
    // console.log(`Удалён из избранного: ${id}`);
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при удалении:", err.message);
  }
};

// Добавление в корзину
const addToBasket = async (id) => {
  try {
    // Если корзины ещё нет, создаём массив
    if (!currentUser.value.basket) {
      currentUser.value.basket = [];
    }

    // Проверяем, чтобы товар не был добавлен дважды
    if (!currentUser.value.basket.includes(id)) {
      currentUser.value.basket.push(id);

      // Отправляем на сервер
      await axios.patch(
        `https://30a5b8072c05076b.mokky.dev/users/${currentUser.value.id}`,
        { basket: currentUser.value.basket }
      );

      // console.log(`Добавлен в корзину: ${id}`);
    } else {
      // console.log(`Товар ${id} уже есть в корзине`);
    }
  } catch (err) {
    error.value = err.message;
    // console.error("Ошибка при добавлении в корзину:", err.message);
  }
};

// Вызов при монтировании
onMounted(() => {
  if (props.loginStatus?.username) {
    fetchUsers();
  }
});

watch(
  () => props.loginStatus?.username,
  (newUsername) => {
    if (newUsername) {
      fetchUsers();
    }
  }
);

// Получение путей для изображений
const getImageUrl = (id, color, imgnumber) => {
  // Путь относительно текущего файла компонента
  const imgPath = new URL(`../assets/${id}/${color}-${imgnumber}.jpg`, import.meta.url);
  return imgPath.href;
};


</script>

<template>
  <div class="favourite">
    <div class="favourite-inner">
      <div class="fav-holder">
        <div class="fav-item" v-for="item in favs" v-if="favs.length !== 0">
          <div class="fav-name">
            <p>{{ item.name }}</p>
          </div>
          <button class="fav-remove" @click="removeFavourite(item.id)">
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
          <div class="fav-left">
            <img :src="getImageUrl(item.id, item.colors[0], 1)" />
          </div>
          <div class="fav-right">
            <p v-for="(col, index) in item.colors" :key="col">
              {{ index === 0 ? col : `, ${col}` }}
            </p>
            <button @click="addToBasket(item.id)">Add to basket</button>
          </div>
        </div>
        <p v-else style="width: 100%; text-align: center">
          Log in or add something to the wishlist
        </p>
      </div>
    </div>
    <div class="else" @click="emit('close-favourite')"></div>
  </div>
</template>

<style scoped>
.favourite {
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
.favourite-inner {
  max-width: 960px;
  width: 50%;
  background: var(--main-bg);
  border-right: solid 1px var(--sub-text);
  padding: calc(var(--main-pad) / 4) calc(var(--main-pad) / 2);
  display: flex;
  gap: 2%;
  transition: 0.4s;
}
.fav-holder {
  width: 100%;
  height: fit-content;
  display: flex;
  flex-wrap: wrap;
  gap: 2%;
  row-gap: 12px;
}
.fav-item {
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
.fav-item:hover {
  box-shadow: rgba(0, 0, 0, 0.1) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}
.fav-name {
  position: absolute;
  z-index: 3;
  white-space: nowrap;
}
.fav-remove {
  position: absolute;
  z-index: 3;
  right: 12px;
}
.fav-remove svg {
  width: 24px;
  height: 24px;
}
.fav-left {
  height: 100%;
  aspect-ratio: 1 / 1;
  overflow: hidden;
  position: relative;
  background: #000;
}
.fav-left img {
  width: 100%;
  position: absolute;
  bottom: 0;
  width: fit-content;
}
.fav-right {
  flex: 1;
  height: fit-content;
  margin: auto 0;
  display: flex;
  gap: 4px;
  flex-wrap: wrap;
}
.fav-right p {
  width: fit-content;
  height: fit-content;
  font-size: 14px;
}
.fav-right button {
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
</style>
