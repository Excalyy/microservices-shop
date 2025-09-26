<template>
  <footer class="bg-white text-black border-t border-gray-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <div class="grid grid-cols-1 md:grid-cols-4 gap-8">
        <!-- Информация о компании -->
        <div class="col-span-1">
          <div class="flex items-center space-x-2 mb-4">
            <div class="w-8 h-8 bg-black rounded-lg flex items-center justify-center">
              <span class="text-white font-bold text-sm">MS</span>
            </div>
            <span class="text-xl font-bold text-black">MicroShop</span>
          </div>
          <p class="text-gray-500 text-sm leading-relaxed">
            Ваш надежный онлайн-магазин, созданный на основе современной микросервисной архитектуры.
            Быстрый, надежный и безопасный опыт покупок.
          </p>
        </div>

        <!-- Быстрые ссылки -->
        <div class="col-span-1">
          <h3 class="text-lg font-semibold text-black mb-4">Быстрые ссылки</h3>
          <ul class="space-y-2">
            <li>
              <router-link to="/" class="text-gray-600 hover:text-black text-sm transition-colors">
                Главная
              </router-link>
            </li>
            <li>
              <router-link to="/catalog" class="text-gray-600 hover:text-black text-sm transition-colors">
                Каталог
              </router-link>
            </li>
            <li>
              <router-link to="/cart" class="text-gray-600 hover:text-black text-sm transition-colors">
                Корзина
              </router-link>
            </li>
            <li>
              <router-link to="/profile" class="text-gray-600 hover:text-black text-sm transition-colors">
                Мой аккаунт
              </router-link>
            </li>
          </ul>
        </div>

        <!-- Категории -->
        <div class="col-span-1">
          <h3 class="text-lg font-semibold text-black mb-4">Категории</h3>
          <ul class="space-y-2">
            <li>
              <a href="#" class="text-gray-600 hover:text-black text-sm transition-colors">
                Телевизоры
              </a>
            </li>
            <li>
              <a href="#" class="text-gray-600 hover:text-black text-sm transition-colors">
                Смартфоны
              </a>
            </li>
            <li>
              <a href="#" class="text-gray-600 hover:text-black text-sm transition-colors">
                Ноутбуки
              </a>
            </li>
            <li>
              <a href="#" class="text-gray-600 hover:text-black text-sm transition-colors">
                Роутеры
              </a>
            </li>
          </ul>
        </div>

        <!-- Контактная информация -->
        <div class="col-span-1">
          <h3 class="text-lg font-semibold text-black mb-4">Контакты</h3>
          <ul class="space-y-2 text-sm text-gray-600">
            <li class="flex items-center space-x-2">
              <svg class="h-4 w-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 7.89a1 1 0 001.415 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
              </svg>
              <span>support@microshop.com</span>
            </li>
            <li class="flex items-center space-x-2">
              <svg class="h-4 w-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
              </svg>
              <span>+1 (555) 123-4567</span>
            </li>
            <li class="flex items-center space-x-2">
              <svg class="h-4 w-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
              </svg>
              <span>ул. Торговая, 123, Город, Область 12345</span>
            </li>
          </ul>
        </div>
      </div>

      <!-- Нижняя панель -->
      <div class="border-t border-gray-200 mt-8 pt-8 flex flex-col md:flex-row justify-between items-center">
        <div class="text-sm text-gray-500">
          © {{ currentYear }} MicroShop. Все права защищены.
        </div>

        <div class="flex items-center space-x-6 mt-4 md:mt-0">
          <a href="#" class="text-sm text-gray-600 hover:text-black transition-colors">
            Политика конфиденциальности
          </a>
          <a href="#" class="text-sm text-gray-600 hover:text-black transition-colors">
            Условия использования
          </a>
          <a href="#" class="text-sm text-gray-600 hover:text-black transition-colors">
            Политика cookies
          </a>
        </div>
      </div>
    </div>
  </footer>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'AppFooter',
  setup() {
    const currentYear = computed(() => new Date().getFullYear())

    return {
      currentYear
    }
  }
}
</script>
