<script setup>
import { ref, onMounted, nextTick } from "vue";
import { initFlowbite } from "flowbite";
import HelloWorld from "./components/HelloWorld.vue";
import TheWelcome from "./components/TheWelcome.vue";
import Login from "./components/Login.vue";

const user = ref(null);
const SESSION_STORAGE_KEY = "user_session";

const handleLoginSuccess = (loggedInUser, rememberMe = false) => {
  user.value = loggedInUser;
  // Guardar sesión en localStorage
  const sessionData = {
    user: loggedInUser,
    timestamp: Date.now(),
  };
  localStorage.setItem(SESSION_STORAGE_KEY, JSON.stringify(sessionData));

  nextTick(() => {
    initFlowbite();
  });
};

const handleLogout = () => {
  user.value = null;
  // Eliminar sesión del localStorage
  localStorage.removeItem(SESSION_STORAGE_KEY);
};

const restoreSession = () => {
  try {
    const sessionData = localStorage.getItem(SESSION_STORAGE_KEY);
    if (sessionData) {
      const parsed = JSON.parse(sessionData);
      user.value = parsed.user;
      nextTick(() => {
        initFlowbite();
      });
    }
  } catch (error) {
    console.error("Error al restaurar sesión:", error);
    localStorage.removeItem(SESSION_STORAGE_KEY);
  }
};

onMounted(() => {
  restoreSession();
});
</script>

<template>
  <div v-if="user" class="min-h-screen bg-gray-50 dark:bg-gray-900">
    <nav class="bg-white border-gray-200 dark:bg-gray-900 border-b">
      <div
        class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4"
      >
        <a href="#" class="flex items-center space-x-3 rtl:space-x-reverse">
          <span
            class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white"
            >App</span
          >
        </a>
        <div
          class="flex items-center md:order-2 space-x-3 md:space-x-0 rtl:space-x-reverse"
        >
          <button
            type="button"
            class="flex text-sm bg-gray-800 rounded-full md:me-0 focus:ring-4 focus:ring-gray-300 dark:focus:ring-gray-600"
            id="user-menu-button"
            aria-expanded="false"
            data-dropdown-toggle="user-dropdown"
            data-dropdown-placement="bottom"
          >
            <span class="sr-only">Open user menu</span>
            <div
              class="w-8 h-8 rounded-full bg-indigo-500 flex items-center justify-center text-white font-bold"
            >
              {{ user.name.charAt(0).toUpperCase() }}
            </div>
          </button>
          <!-- Dropdown menu -->
          <div
            class="z-50 hidden my-4 text-base list-none bg-white divide-y divide-gray-100 rounded-lg shadow dark:bg-gray-700 dark:divide-gray-600"
            id="user-dropdown"
          >
            <div class="px-4 py-3">
              <span class="block text-sm text-gray-900 dark:text-white">{{
                user.name
              }}</span>
              <span
                class="block text-sm text-gray-500 truncate dark:text-gray-400"
                >{{ user.email }}</span
              >
            </div>
            <ul class="py-2" aria-labelledby="user-menu-button">
              <li>
                <a
                  href="#"
                  class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600 dark:text-gray-200 dark:hover:text-white"
                  >Dashboard</a
                >
              </li>
              <li>
                <a
                  href="#"
                  class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600 dark:text-gray-200 dark:hover:text-white"
                  >Settings</a
                >
              </li>
              <li>
                <a
                  href="#"
                  @click.prevent="handleLogout"
                  class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600 dark:text-gray-200 dark:hover:text-white"
                  >Sign out</a
                >
              </li>
            </ul>
          </div>
        </div>
      </div>
    </nav>

    <main class="max-w-screen-xl mx-auto p-4">
      <div
        class="p-4 border-2 border-gray-200 border-dashed rounded-lg dark:border-gray-700 mt-4"
      >
        <h1 class="text-3xl font-extrabold text-gray-900 dark:text-white mb-4">
          Welcome back, {{ user.name }}!
        </h1>
        <TheWelcome />
      </div>
    </main>
  </div>

  <Login v-else @login-success="handleLoginSuccess" />
</template>
