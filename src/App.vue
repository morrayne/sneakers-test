<script setup>
import { ref, watch } from "vue";
import HeaderComponent from "./components/headerComponent.vue";
import LoginComponent from "./components/loginComponent.vue";
import RosterComponent from "./components/rosterComponent.vue";
import FavouriteComponent from "./components/favouriteComponent.vue";
import BasketComponent from "./components/basketComponent.vue";
import MoreComponent from "./components/moreComponent.vue";

// Открыть/закрыть окно
const showLogin = ref(false);
// prettier-ignore
const toggleLogin = () => {showLogin.value = !showLogin.value;};

// ФИЛЬТРАЦИЯ, ПОИСК И ПРОГРУЗКА СПИСКА КРОССОВОК
const sneakersUrl = ref("https://30a5b8072c05076b.mokky.dev/sneakers");
// prettier-ignore
const changeFil = (field, data) => { sortAndSearch.value.searchField = field; sortAndSearch.value.searchData = data};
// prettier-ignore
const changeOrder = (orderData) => {sortAndSearch.value.sortBy = orderData};
// Фильтрация и поиск
const sortAndSearch = ref({
  searchField: "",
  searchData: "",
  sortBy: "id",
});
// Запрос новой ссылки
// prettier-ignore
watch(sortAndSearch, (newValue) => {makeLink(newValue)}, { deep: true });
// Создание ссылки на готовый ресурс
const makeLink = (obj) => {
  sneakersUrl.value = "https://30a5b8072c05076b.mokky.dev/sneakers";
  // prettier-ignore
  if (obj.searchField && obj.searchData) {
    sneakersUrl.value = `${sneakersUrl.value}?${obj.searchField}=*${obj.searchData}&sortBy=${obj.sortBy}`
  } else {
    sneakersUrl.value = `${sneakersUrl.value}?sortBy=${obj.sortBy}`
  }
};
// 
const loginStatus = ref({});
const handleLoginSuccess = (data) => {
  loginStatus.value = data;
};
// 
const showFavourite = ref(false);
const toggleFavourite = () => {showFavourite.value = !showFavourite.value};
const showBasket = ref(false);
const toggleBasket = () => {showBasket.value = !showBasket.value};
const showMore = ref(false);
const selectedBootId = ref(null);
const toggleMore = (bootId) => {showMore.value = !showMore.value; selectedBootId.value = bootId;};
</script>

<template>
  <HeaderComponent @toggle-login="toggleLogin" @toggle-favourite="toggleFavourite" @toggle-basket="toggleBasket" @change-filter="changeFil" />
  <RosterComponent :sneakersUrl="sneakersUrl" :loginStatus="loginStatus" @change-order="changeOrder" @toggle-more="toggleMore" />
  <LoginComponent v-if="showLogin" @login-success="handleLoginSuccess" @close-login="toggleLogin" />
  <FavouriteComponent v-if="showFavourite" :loginStatus="loginStatus" @close-favourite="toggleFavourite"/>
  <BasketComponent v-if="showBasket" :loginStatus="loginStatus" @close-basket="toggleBasket"/>
  <MoreComponent v-if="showMore" :loginStatus="loginStatus" :bootId="selectedBootId" @close-more="toggleMore"/>
</template>

<style></style>
