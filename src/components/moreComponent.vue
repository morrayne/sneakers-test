<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

const emit = defineEmits(["close-more"]);
const props = defineProps({
  bootId: Number,
  loginStatus: Object, // { username: string, ... }
});

const API = "https://30a5b8072c05076b.mokky.dev";

const stockItems = ref([]);
const sneaker = ref(null);
const error = ref(null);

const mainImg = ref(1);
const mainColor = ref(null);
const selectedSize = ref(null);

const fetchData = async () => {
  try {
    const [stockRes, sneakersRes] = await Promise.all([
      axios.get(`${API}/stock`),
      axios.get(`${API}/sneakers`),
    ]);

    stockItems.value = stockRes.data.filter((item) => item.id === props.bootId);
    sneaker.value =
      sneakersRes.data.find((item) => item.id === props.bootId) || null;

    if (sneaker.value) {
      mainColor.value = sneaker.value.colors[0];
    }
  } catch (err) {
    error.value = err.message;
  }
};

const getImageUrl = (id, color, imgnumber) => {
  const imgURL = `../assets/${id}/${color}-${imgnumber}.jpg`;
  return new URL(imgURL, import.meta.url).href;
};

onMounted(() => {
  if (props.bootId) {
    fetchData();
  }
});

// выбор размера (если пригодится)
const selectSize = (size) => {
  selectedSize.value = size;
};

/**
 * Универсальная функция добавления id товара в поле пользователя (basket | favourite).
 * Берёт username из props.loginStatus.username
 */
const addTo = async (type) => {
  try {
    if (!sneaker?.value?.id) return;

    const username = props?.loginStatus?.username;
    if (!username) {
      return;
    }

    // 1) Найти пользователя по username
    const { data: users } = await axios.get(
      `${API}/users?username=${encodeURIComponent(username)}`
    );
    const user = Array.isArray(users) ? users[0] : null;

    if (!user) {
      return;
    }

    // 2) Сформировать новый массив без дублей, добавляя id товара
    const current = Array.isArray(user[type]) ? user[type] : [];
    const next = current.includes(sneaker.value.id)
      ? current // уже есть — не добавляем дубликат
      : [...current, sneaker.value.id];

    // 3) PATCH по id пользователя
    await axios.patch(`${API}/users/${user.id}`, {
      [type]: next,
    });

    alert(
      type === "basket" ? "Товар добавлен в корзину!" : "Товар добавлен в wishlist!"
    );
  } catch (err) {
    console.error(err);
    alert("Ошибка при добавлении.");
  }
};
</script>

<template>
  <div class="more">
    <div class="more-inner" v-if="sneaker">
      <div class="top">
        <div class="top-left">
          <div class="img-holder">
            <img :src="getImageUrl(sneaker.id, mainColor, mainImg)" />
          </div>
        </div>
        <div class="top-right">
          <div class="img-holder">
            <img
              :src="getImageUrl(sneaker.id, mainColor, 1)"
              @click="mainImg = 1"
            />
          </div>
          <div class="img-holder">
            <img
              :src="getImageUrl(sneaker.id, mainColor, 2)"
              @click="mainImg = 2"
            />
          </div>
          <div class="img-holder">
            <img
              :src="getImageUrl(sneaker.id, mainColor, 3)"
              @click="mainImg = 3"
            />
          </div>
        </div>
      </div>

      <div class="mid">
        <p class="mid-name">
          {{ sneaker.name }}
        </p>

        <div class="pallete">
          <p>Color pallete</p>
          <button
            class="pal"
            v-for="color in sneaker.colors"
            :key="color"
            :style="{ background: color }"
            @click="mainColor = color"
          ></button>
        </div>

        <div class="pallete">
          <p>Size</p>
          <button
            class="size"
            v-for="size in [41, 42, 43, 44, 45]"
            :key="size"
            :style="{
              opacity: stockItems.some((item) => item.size === size) ? 1 : 0.3,
              border: selectedSize === size ? '2px solid black' : '1px solid gray',
            }"
            @click="selectSize(size)"
          >
            {{ size }}
          </button>
        </div>

        <div class="pallete">
          <p>Left in stock</p>
          <p v-if="stockItems.length">{{ stockItems.length }}</p>
          <p v-else>0</p>
        </div>
      </div>

      <div class="bot" v-if="props.loginStatus.isLoggedIn">
        <button @click="addTo('basket')">Add to the basket</button>
        <button @click="addTo('favourite')">Add to the wishlist</button>
      </div>
    </div>

    <div class="else" @click="emit('close-more')"></div>
  </div>
</template>

<style scoped>
.more {
  max-width: 1920px;
  width: 100vw;
  height: 100%;
  position: fixed;
  display: flex;
  z-index: 15;
  top: 0;
  background: #00000020;
  transition: 0.4s;
}
.more-inner {
  max-width: 960px;
  width: 50%;
  height: 100%;
  background: var(--main-bg);
  border-left: solid 1px var(--sub-text);
  padding: calc(var(--main-pad) / 4) calc(var(--main-pad) / 2);
  position: relative;
  left: 0;
  display: flex;
  flex-direction: column;
  transition: 0.4s;
}
.top {
  display: flex;
  gap: 12px;
}
.top-left {
  width: 60%;
}
.top-right {
  width: 15%;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.img-holder {
  width: 100%;
  aspect-ratio: 1 / 1;
  position: relative;
  overflow: hidden;
}
.img-holder img {
  position: absolute;
  width: 100%;
  bottom: 0;
}
.mid-name {
  font-size: 32px;
  font-weight: 700;
}
.pallete {
  display: flex;
  align-items: center;
  font-size: 24px;
  gap: 8px;
}
.pallete p {
  margin-right: 16px;
}
.pal {
  height: 28px;
  aspect-ratio: 1 / 1;
  border: solid 2px var(--main-text);
  border-radius: 6px;
}
.size {
  padding: 0 16px;
  border: solid 1px var(--main-text);
  border-radius: 4px;
  font-size: 18px;
}
.bot {
  margin-top: auto;
  display: flex;
  gap: 12px;
}
.bot button {
  width: 25%;
  padding: 4px 0;
  background: var(--main-text);
  color: var(--main-bg);
  border-radius: 4px;
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
