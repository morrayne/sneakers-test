<script setup>
import { ref, watch, onMounted } from "vue";
import axios from "axios";
const emit = defineEmits(["change-order", "toggle-more"]);
const sneakers = ref([]);
const stock = ref([]);
// Получение пропсов
const props = defineProps({
  sneakersUrl: String,
  loginStatus: Object,
});
// Получение путей для изображений
const getImageUrl = (id, color, imgnumber) => { 
  const imgURL = `/src/assets/${id}/${color}-${imgnumber}.jpg`; 
  return new URL(imgURL, import.meta.url).href; 
};

// Запрос на сервер
const fetchData = async (place, link) => {
  try {
    // prettier-ignore
    const { data } = await axios.get(link);
    place.value = data;
  } catch (error) {
    console.log("Ошибка: " + (error.response?.data?.message || error.message));
  }
};
// Изменение сортировки
const order = ref("");
const onSelectChange = (event) => {
  order.value = event.target.value;
  emit("change-order", order.value);
};
// prettier-ignore
// Прорисовка списка кроссовок
watch(() => props.sneakersUrl, (newUrl) => {fetchData(sneakers, newUrl)}, {immediate: true});
onMounted(() => {
  fetchData(stock, "https://30a5b8072c05076b.mokky.dev/stock");
});
//
//
//
const addToSomething = async (itemId, what) => {
  if (!props.loginStatus.isLoggedIn) {
    // console.log("Сначала войдите в аккаунт");
    return;
  }

  try {
    // 1. Получаем всех пользователей
    const { data: users } = await axios.get(
      "https://30a5b8072c05076b.mokky.dev/users"
    );
    // 2. Находим текущего пользователя по username
    const user = users.find((u) => u.username === props.loginStatus.username);
    if (!user) {
      // console.log("Пользователь не найден");
      return;
    }
    // 3. Создаём или обновляем массив favourite
    const updatedFav = user[what] ? [...user[what]] : [];
    if (!updatedFav.includes(itemId)) {
      updatedFav.push(itemId);
    }
    // 4. Отправляем PATCH-запрос на обновление пользователя
    await axios.patch(`https://30a5b8072c05076b.mokky.dev/users/${user.id}`, {
      [what]: updatedFav,
    });
    // console.log("Добавлено в избранное!");
  } catch (error) {
    console.log("Ошибка при добавлении в избранное:", error.message);
  }
};
</script>

<template>
  <select class="order" @change="onSelectChange($event)">
    <option value="id" selected>Appearence date</option>
    <option value="gender">Gender</option>
    <option value="colors">Color</option>
    <option value="brand">Brand</option>
  </select>
  <div class="roster">
    <div class="roster-item" v-for="boot in sneakers">
    <!-- <div class="roster-item" v-for="boot in sneakers"> -->
      <!-- prettier-ignore -->
      <div class="roster-item-top" @click="emit('toggle-more', boot.id)">
        <img :src="getImageUrl(boot.id, boot.colors[0], 1)" />
        <img :src="getImageUrl(boot.id, boot.colors[0], 2)" class="hide-img"/>
      </div>
      <span class="roster-item-mid">
        <span class="all-colors">
          <p class="all-colors-gender" v-if="boot.gender === 0">Male /</p>
          <p class="all-colors-gender" v-if="boot.gender === 1">Female /</p>
          <p class="all-colors-gender" v-if="boot.gender === 2">Uni /</p>
          <p v-for="(col, index) in boot.colors" :key="col">
            {{ index === 0 ? col : `, ${col}` }}
          </p>
          <p class="all-colors-gender" style="margin-left: 4px;">{{ ` ${boot.id}` }}</p>
        </span>
        <p class="item-name">{{ boot.name }}</p>
      </span>
      <div class="roster-item-bot" v-if="props.loginStatus.isLoggedIn">
        <button class="in-fav" @click="addToSomething(boot.id, 'favourite')">
          favourite
        </button>
        <button class="in-bas" @click="addToSomething(boot.id, 'basket')">
          basket
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.order {
  width: 360px;
  height: 28px;
  margin: 16px var(--main-pad) 0;
  border: solid 1px var(--sub-text);
  border-radius: 4px;
  outline: none;
  background: var(--card);
}
.roster {
  width: 100%;
  margin-top: 16px;
  padding: 0 var(--main-pad);
  display: flex;
  flex-wrap: wrap;
  gap: 2%;
  row-gap: 32px;
}
.roster-item {
  width: 23.5%;
  padding: 16px;
  background: var(--card);
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  gap: 2px;
  cursor: default;
  box-shadow: rgba(0, 0, 0, 0) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
  transition: 0.4s;
}
.roster-item:hover {
  box-shadow: rgba(0, 0, 0, 0.2) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}
.roster-item-top {
  width: 100%;
  aspect-ratio: 1 / 1;
  position: relative;
  overflow: hidden;
}
.roster-item-top img {
  width: 100%;
  bottom: 0;
  left: 0;
  position: absolute;
  transition: 0.2s;
}
.hide-img:hover {
  opacity: 0;
}
.roster-item-mid {
  display: flex;
  flex-direction: column;
  /* gap: 2px; */
}
.roster-item-mid .item-name {
  font-size: 22px;
  font-weight: 700;
  white-space: nowrap;
  cursor: text;
}
.no-size {
  opacity: 0.4;
}
.all-colors {
  display: flex;
}
.all-colors-gender {
  margin-right: 4px;
}
.all-colors p {
  font-size: 14px;
  color: var(--sub-text);
}
.ifsale * {
  width: fit-content !important;
  position: relative !important;
  margin: 0 4px;
  font-size: 18px;
}
.sale {
  scale: 0.8;
  text-decoration: line-through;
  color: var(--sub-text);
}
.nosale {
  font-weight: 500;
}
.roster-item .roster-item-bot {
  width: 100%;
  height: 36px;
  margin-top: 6px;
  display: flex;
  gap: 8px;
}
.roster-item .roster-item-bot button {
  width: 50%;
  height: 100%;
  border-radius: 4px;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: 0.4s;
  border: solid 1px var(--main-text);
}
.roster-item .roster-item-bot button:hover {
  scale: 1.05;
}
.roster-item .roster-item-bot .in-fav {
  color: var(--main-text);
  background: var(--main-bg);
}
.roster-item .roster-item-bot .in-bas {
  background: var(--main-text);
  color: var(--main-bg);
}
.left:hover ~ .in-bas {
  opacity: 0;
}
</style>
