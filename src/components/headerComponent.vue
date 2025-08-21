<script setup>
import { ref } from "vue";
const emit = defineEmits(["toggle-login", 'toggle-favourite', 'toggle-basket', "change-filter"]);
// Настройка критериев поиска
const newURL = ref({
  field: "",
  fieldValue: "",
});
// Смена поиска
const changeFilter = (event, field, fieldValue) => {
  if (event && event.target.value) {
    newURL.value.field = "name";
    newURL.value.fieldValue = event.target.value;
  } else if (field && fieldValue) {
    newURL.value.field = field;
    newURL.value.fieldValue = fieldValue;
  } else {
    newURL.value.field = "";
    newURL.value.fieldValue = "";
  }
  emit("change-filter", newURL.value.field, newURL.value.fieldValue);
};
</script>

<template>
  <header>
    <div class="header-top header-part">
      <div class="header-cover">
        <h1>
          <img src="/vue.svg" alt="Vue logo" />
          <img src="/vite.svg" alt="Vite logo" />
          <p>VUE SNEAKERS</p>
          <span>in association with VITE</span>
        </h1>
      </div>
    </div>
    <div class="header-mid">
      <div class="header-cover">
        <!-- prettier-ignore -->
        <input type="text" placeholder="Nike P-6000" @input="changeFilter($event)"/>
        <button @click="emit('toggle-favourite')" style="margin-left: auto">
          <!-- <img src="/remembrance.webp" alt="Profile" /> -->
          <p>Favourite</p>
        </button>
        <button @click="emit('toggle-basket')">
          <!-- <img src="/remembrance.webp" alt="Profile" /> -->
          <p>Basket</p>
        </button>
        <button @click="emit('toggle-login')">
          <!-- <img src="/remembrance.webp" alt="Profile" /> -->
          <p>Your profile</p>
        </button>
      </div>
    </div>
    <div class="header-bot header-part">
      <!-- prettier-ignore -->
      <div class="header-cover">
        <button class="brand-filter" @click="changeFilter(false, 'brand', 'NIKE')">
          <img src="/logo/nike.svg" alt="Nike" />
          <p>Nike</p>
        </button>
        <button class="brand-filter" @click="changeFilter(false, 'brand', 'ADIDAS')">
          <img src="/logo/adidas.svg" alt="Adidas" />
          <p>Adidas</p>
        </button>
        <button class="brand-filter" @click="changeFilter(false, 'brand', 'PUMA')">
          <img src="/logo/puma.svg" alt="Puma" />
          <p>Puma</p>
        </button>
        <button class="brand-filter" @click="changeFilter(false, 'gender', '0')">
          <p>Male</p>
        </button>
        <button class="brand-filter" @click="changeFilter(false, 'gender', '1')">
          <p>Female</p>
        </button>
        <button class="brand-filter" @click="changeFilter(false, '', '')">
          <p>Clear filters</p>
        </button>
        <!-- <button class="brand-filter" @click="emit('change-url', saleUrl)"><p>Sale</p></button> -->
      </div>
    </div>
  </header>
</template>

<style scoped>
header {
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  z-index: 10;
  background: var(--main-bg);
}
header .header-top,
header .header-bot {
  height: var(--header-height);
  padding: 0 var(--main-pad);
}
h1 {
  display: flex;
  gap: 6px;
}
h1 img {
  height: 18px;
}
h1 p {
  font-size: 16px;
  font-weight: 700;
}
h1 span {
  height: fit-content;
  margin-top: auto;
  font-size: 12px;
  color: var(--sub-text);
  font-weight: 500;
}
header .header-mid {
  height: calc(var(--header-height) * 1.5);
  background: var(--sub-bg);
}
.header-cover {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
}
.header-mid .header-cover {
  padding: 0 var(--main-pad);
  gap: 12px;
}
.header-mid .header-cover input {
  width: 360px;
  height: var(--header-height);
  background: var(--main-bg);
  border: none;
  outline: none;
  border-radius: 4px;
  padding: 4px 12px;
  font-size: 14px;
}
.header-mid .header-cover button {
  display: flex;
  gap: 8px;
  font-size: 14px;
  align-items: center;
  background: var(--main-text);
  color: var(--main-bg);
  padding: 4px 12px;
  border-radius: 8px;
}
.header-mid .header-cover button img {
  height: 24px;
}
.header-bot {
  height: calc(var(--header-height) * 1.2) !important;
}
.header-bot .header-cover {
  border-bottom: solid 1px var(--sub-text);
  gap: 4px;
}
.brand-filter {
  height: fit-content;
  display: flex;
  align-items: center;
  gap: 4px;
  background: var(--sub-bg);
  border-radius: 4px;
  padding: 0 10px;
  transition: 0.4s;
  box-shadow: rgba(0, 0, 0, 0) 0px 4px 16px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}
.brand-filter:hover {
  box-shadow: rgba(0, 0, 0, 0.2) 0px 4px 16px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}
.brand-filter img {
  height: 18px;
}
.brand-filter p {
  font-size: 14px;
}
</style>
