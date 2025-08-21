<script setup>
import { ref } from "vue";
import axios from "axios";
const emit = defineEmits(["close-login", "login-success"]);
const loginData = ref({
  isLoggedIn: false,
  loginMode: true,
  username: "",
  pass: "",
});
// Функция выхода из аккаунта
const logOut = () => {
  loginData.value.isLoggedIn = false;
  loginData.value.loginMode = true;
  loginData.value.username = "";
  loginData.value.pass = "";
  emit("login-success", loginData.value);
};
// Вход или регистрация
const handleSubmit = async () => {
  if (!loginData.value.username || !loginData.value.pass) {
    return;
  }
  try {
    // prettier-ignore
    const { data: users } = await axios.get("https://30a5b8072c05076b.mokky.dev/users");
    if (loginData.value.loginMode) {
      // prettier-ignore
      const user = users.find((u) => u.username === loginData.value.username && u.pass === loginData.value.pass);
      if (user) {
        // console.log("Успешный вход!");
        loginData.value.isLoggedIn = true;
        // Отправляем данные в родителя
        emit("login-success", loginData.value);
        emit("close-login");
      } else {
        // console.log("Неверный логин или пароль");
      }
    } else {
      // Регистрация
      // prettier-ignore
      const userExists = users.some((u) => u.username === loginData.value.username);
      if (userExists) {
        // console.log("Этот логин уже занят");
        return;
      } else {
        // prettier-ignore
        await axios.post("https://30a5b8072c05076b.mokky.dev/users", {
          username: loginData.value.username,
          pass: loginData.value.pass,
          favourite: [],
          basket: [],
          history: [],
          id: users.length === 0 ? 0 : users[users.length - 1].id + 1
        });
        // console.log("Регистрация прошла успешно!");
        loginData.value.loginMode = true;
      }
    }
  } catch (error) {
    console.log("Ошибка: " + (error.response?.data?.message || error.message));
  }
};
</script>

<template>
  <div class="login">
    <div class="login-inner">
      <!-- prettier-ignore -->
      <form @submit.prevent="handleSubmit">
        <span class="login-duo">
          <button type="button" @click="loginData.loginMode = true" :class="{ 'duo-selected': loginData.loginMode }">
            Login
          </button>
          <button type="button" @click="loginData.loginMode = false" :class="{ 'duo-selected': !loginData.loginMode }">
            Registration
          </button>
        </span>
        <input v-model="loginData.username" type="text" placeholder="Username" required />
        <input v-model="loginData.pass" type="password" placeholder="Password" required />
        <div class="duo-button">
          <button type="button" class="cancel" @click="logOut">Log out</button>
          <button type="submit" class="confirm">
            {{ loginData.loginMode ? "Log In" : "Register" }}
          </button>
        </div>
      </form>
    </div>
    <div class="else" @click="emit('close-login')"></div>
  </div>
</template>

<style scoped>
.login {
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
.login-inner {
  max-width: 960px;
  width: 50%;
  height: 100%;
  background: var(--main-bg);
  border-left: solid 1px var(--sub-text);
  padding: calc(var(--main-pad) / 2) var(--main-pad);
  position: relative;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  transition: 0.4s;
}
.else {
  flex: 1;
  height: 100%;
}
.left50vw {
  left: -50vw;
}
.login-duo {
  width: 100%;
  display: flex;
  gap: 4px;
  align-items: center;
  background: var(--sub-bg);
  padding: 6px 8px;
  border-bottom: solid 1px var(--sub-text);
}
.login-duo button {
  padding: 2px 10px;
  transition: 0.4s;
  border-radius: 4px;
  background: var(--sub-bg);
  color: var(--sub-text);
}
.login-duo .duo-selected {
  background: var(--sub-text);
  color: var(--main-bg);
}
.login form {
  width: 80%;
  display: flex;
  flex-direction: column;
  gap: 8px;
  background: var(--sub-bg);
  padding: 8px;
}
.login form input {
  width: 80%;
  height: 32px;
  background: var(--main-bg);
  border: none;
  outline: none;
  border-radius: 4px;
  padding: 4px 12px;
  font-size: 14px;
}
.login .duo-button {
  display: flex;
  gap: 8px;
}
.login .duo-button * {
  width: 50%;
  border-radius: 4px;
  padding: 4px 0;
  border: solid 1px var(--sub-text);
}
.login .cancel {
  background: var(--main-bg);
  color: var(--sub-text);
}
.login .confirm {
  background: var(--sub-text);
  color: var(--main-bg);
}
</style>
