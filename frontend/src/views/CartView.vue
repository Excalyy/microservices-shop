<!-- frontend/src/views/CartView.vue -->
<template>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
    <!-- Заголовок -->
    <div class="flex items-center justify-between mb-10">
      <h1 class="text-4xl font-extrabold text-gray-900">Корзина покупок</h1>
      <router-link
        to="/catalog"
        class="text-gray-600 hover:text-black flex items-center transition-colors duration-200"
      >
        <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16l-4-4m0 0l4-4m-4 4h18" />
        </svg>
        Продолжить покупки
      </router-link>
    </div>

    <!-- Состояние загрузки -->
    <div v-if="loading" class="flex justify-center py-16">
      <LoadingSpinner size="lg" text="Загрузка корзины..." />
    </div>

    <!-- Пустая корзина -->
    <div v-else-if="isEmpty" class="text-center py-16 bg-white rounded-xl shadow-sm">
      <svg class="mx-auto h-16 w-16 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4m0 0L7 13m0 0l-2.5 5M7 13l-2.5 5m0 0h10.5" />
      </svg>
      <h3 class="mt-3 text-xl font-semibold text-gray-900">Ваша корзина пуста</h3>
      <p class="mt-2 text-gray-500">Начните добавлять товары в корзину!</p>
      <div class="mt-6">
        <BaseButton
          variant="primary"
          @click="$router.push('/catalog')"
        >
          Начать покупки
        </BaseButton>
      </div>
    </div>

    <!-- Содержимое корзины -->
    <div v-else class="lg:grid lg:grid-cols-12 lg:gap-8">
      <!-- Товары в корзине -->
      <div class="lg:col-span-8">
        <div class="bg-white rounded-xl shadow-sm border border-gray-100">
          <div class="px-6 py-5 border-b border-gray-100">
            <h2 class="text-xl font-semibold text-gray-900">
              Товары в корзине ({{ totalItems }} {{ getItemsText(totalItems) }})
            </h2>
          </div>

          <div class="divide-y divide-gray-100">
            <div
              v-for="item in cartItems"
              :key="item.id"
              class="p-6 flex"
            >
              <!-- Изображение товара -->
              <div class="flex-shrink-0">
                <ProductImage
                  :product="getProductForItem(item)"
                  size="sm"
                  container-class="w-24 h-24 rounded-lg"
                />
              </div>

              <!-- Информация о товаре -->
              <div class="ml-6 flex-1">
                <div class="flex items-start justify-between">
                  <div>
                    <h3 class="text-lg font-semibold text-gray-900">
                      <router-link
                        :to="`/products/${item.product_id}`"
                        class="hover:text-black transition-colors duration-200"
                      >
                        {{ item.product_name }}
                      </router-link>
                    </h3>
                    <p class="mt-1 text-sm text-gray-500">
                      ${{ formatPrice(item.price) }} за шт.
                    </p>

                    <!-- Предупреждение о наличии -->
                    <div
                      v-if="item.product_info && item.quantity > item.product_info.stock_quantity"
                      class="mt-2 flex items-center text-amber-500"
                    >
                      <svg class="w-5 h-5 mr-1.5" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd" />
                      </svg>
                      <span class="text-sm">Осталось только {{ item.product_info.stock_quantity }} шт.</span>
                    </div>

                    <!-- Предупреждение об изменении цены -->
                    <div
                      v-if="item.product_info && item.price !== item.product_info.current_price"
                      class="mt-2 flex items-center text-blue-500"
                    >
                      <svg class="w-5 h-5 mr-1.5" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z" clip-rule="evenodd" />
                      </svg>
                      <span class="text-sm">
                        Цена изменена на ${{ formatPrice(item.product_info.current_price) }}
                      </span>
                    </div>
                  </div>

                  <!-- Кнопка удаления -->
                  <button
                    @click="removeItem(item.id)"
                    class="ml-4 text-gray-400 hover:text-red-500 transition-colors duration-200"
                    :disabled="updatingItems.has(item.id)"
                  >
                    <span class="sr-only">Удалить</span>
                    <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                    </svg>
                  </button>
                </div>

                <!-- Управление количеством -->
                <div class="mt-4 flex items-center space-x-4">
                  <label class="sr-only">Количество</label>
                  <div class="flex items-center border border-gray-200 rounded-lg overflow-hidden">
                    <button
                      @click="updateQuantity(item.id, item.quantity - 1)"
                      :disabled="item.quantity <= 1 || updatingItems.has(item.id)"
                      class="p-2.5 text-gray-600 hover:text-black disabled:opacity-40 disabled:cursor-not-allowed transition-colors duration-200"
                    >
                      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4" />
                      </svg>
                    </button>

                    <input
                      :value="item.quantity"
                      @blur="updateQuantity(item.id, $event.target.value)"
                      type="number"
                      min="1"
                      :max="item.product_info?.stock_quantity || 999"
                      class="w-16 text-center border-none focus:ring-0 focus:outline-none text-gray-900 text-sm"
                      :disabled="updatingItems.has(item.id)"
                    >

                    <button
                      @click="updateQuantity(item.id, item.quantity + 1)"
                      :disabled="updatingItems.has(item.id) || (item.product_info && item.quantity >= item.product_info.stock_quantity)"
                      class="p-2.5 text-gray-600 hover:text-black disabled:opacity-40 disabled:cursor-not-allowed transition-colors duration-200"
                    >
                      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
                      </svg>
                    </button>
                  </div>

                  <div class="flex items-center text-sm text-gray-600">
                    <span v-if="updatingItems.has(item.id)" class="flex items-center">
                      <div class="spinner mr-2"></div>
                      Обновление...
                    </span>
                    <span v-else>
                      Подитог: ${{ formatPrice(item.subtotal) }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Кнопка очистки корзины -->
        <div class="mt-6 flex justify-end">
          <BaseButton
            variant="outline"
            @click="clearAllItems"
            :loading="clearingCart"
          >
            Очистить корзину
          </BaseButton>
        </div>
      </div>

      <!-- Сводка заказа -->
      <div class="mt-8 lg:mt-0 lg:col-span-4">
        <div class="bg-white rounded-xl shadow-sm border border-gray-100 sticky top-8">
          <div class="px-6 py-5 border-b border-gray-100">
            <h2 class="text-xl font-semibold text-gray-900">Сводка заказа</h2>
          </div>

          <div class="p-6">
            <div class="space-y-4">
              <div class="flex items-center justify-between">
                <dt class="text-sm text-gray-600">Подитог</dt>
                <dd class="text-sm font-medium text-gray-900">${{ formatPrice(totalAmount) }}</dd>
              </div>

              <div class="flex items-center justify-between">
                <dt class="text-sm text-gray-600">Доставка</dt>
                <dd class="text-sm font-medium text-gray-900">Бесплатно</dd>
              </div>

              <div class="flex items-center justify-between">
                <dt class="text-sm text-gray-600">Налог</dt>
                <dd class="text-sm font-medium text-gray-900">${{ formatPrice(totalAmount * 0.1) }}</dd>
              </div>

              <div class="border-t border-gray-100 pt-4 flex items-center justify-between">
                <dt class="text-lg font-semibold text-gray-900">Итого</dt>
                <dd class="text-lg font-semibold text-gray-900">${{ formatPrice(totalAmount * 1.1) }}</dd>
              </div>
            </div>

            <div class="mt-6">
              <BaseButton
                variant="primary"
                size="lg"
                block
                @click="proceedToCheckout"
                :disabled="!isAuthenticated || isEmpty"
              >
                Перейти к оформлению
              </BaseButton>

              <p v-if="!isAuthenticated" class="mt-2 text-sm text-center text-gray-600">
                Пожалуйста, <router-link to="/login" class="text-black hover:text-gray-700 font-medium transition-colors duration-200">войдите в систему</router-link> для оформления заказа
              </p>
            </div>

            <!-- Промокод -->
            <div class="mt-6 border-t border-gray-100 pt-6">
              <label for="promo-code" class="block text-sm font-medium text-gray-700">
                Промокод
              </label>
              <div class="mt-1 flex space-x-2">
                <input
                  id="promo-code"
                  v-model="promoCode"
                  type="text"
                  placeholder="Введите промокод"
                  class="flex-1 px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
                >
                <BaseButton
                  variant="outline"
                  size="sm"
                  @click="applyPromoCode"
                  :disabled="!promoCode.trim()"
                >
                  Применить
                </BaseButton>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Модальное окно оформления заказа -->
    <CheckoutModal
      v-if="showCheckoutModal"
      :cart-items="cartItems"
      :total-amount="totalAmount * 1.1"
      @close="showCheckoutModal = false"
      @order-placed="handleOrderPlaced"
    />
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import BaseButton from '../components/common/BaseButton.vue'
import LoadingSpinner from '../components/common/LoadingSpinner.vue'
import ProductImage from '../components/common/ProductImage.vue'
import CheckoutModal from '../components/checkout/CheckoutModal.vue'
import { useCartStore } from '../store/cart.js'
import { useAuthStore } from '../store/auth.js'
import { useToast } from '../composables/useToast.js'

export default {
  name: 'CartView',
  components: {
    BaseButton,
    LoadingSpinner,
    ProductImage,
    CheckoutModal
  },
  setup() {
    const router = useRouter()
    const cartStore = useCartStore()
    const authStore = useAuthStore()
    const { showSuccess, showError } = useToast()

    // Реактивные данные
    const updatingItems = ref(new Set())
    const clearingCart = ref(false)
    const promoCode = ref('')
    const showCheckoutModal = ref(false)

    // Вычисляемые свойства
    const cartItems = computed(() => cartStore.cartItems)
    const totalItems = computed(() => cartStore.totalItems)
    const totalAmount = computed(() => cartStore.totalAmount)
    const isEmpty = computed(() => cartStore.isEmpty)
    const loading = computed(() => cartStore.loading)
    const isAuthenticated = computed(() => authStore.isAuthenticated)

    // Методы
    const formatPrice = (price) => {
      return parseFloat(price).toFixed(2)
    }

    const getItemsText = (count) => {
      if (count % 10 === 1 && count % 100 !== 11) return 'товар'
      if ([2, 3, 4].includes(count % 10) && ![12, 13, 14].includes(count % 100)) return 'товара'
      return 'товаров'
    }

    const getProductForItem = (item) => {
      // Создаем объект товара для компонента ProductImage
      return {
        id: item.product_id,
        name: item.product_name,
        category_name: item.product_info?.category_name || 'Товар',
        image_url: item.product_info?.image_url || null
      }
    }

    const updateQuantity = async (itemId, newQuantity) => {
      const quantity = parseInt(newQuantity)

      if (quantity < 1) return

      try {
        updatingItems.value.add(itemId)

        const result = await cartStore.updateCartItem(itemId, quantity)
        if (!result.success) {
          showError(result.error || 'Не удалось обновить количество')
        }
      } catch (error) {
        console.error('Ошибка при обновлении количества:', error)
        showError('Не удалось обновить количество')
      } finally {
        updatingItems.value.delete(itemId)
      }
    }

    const removeItem = async (itemId) => {
      try {
        updatingItems.value.add(itemId)

        const result = await cartStore.removeCartItem(itemId)
        if (result.success) {
          showSuccess('Товар удален из корзины')
        } else {
          showError(result.error || 'Не удалось удалить товар')
        }
      } catch (error) {
        console.error('Ошибка при удалении товара:', error)
        showError('Не удалось удалить товар')
      } finally {
        updatingItems.value.delete(itemId)
      }
    }

    const clearAllItems = async () => {
      try {
        clearingCart.value = true

        const result = await cartStore.clearCart()
        if (result.success) {
          showSuccess('Корзина очищена успешно')
        } else {
          showError(result.error || 'Не удалось очистить корзину')
        }
      } catch (error) {
        console.error('Ошибка при очистке корзины:', error)
        showError('Не удалось очистить корзину')
      } finally {
        clearingCart.value = false
      }
    }

    const applyPromoCode = () => {
      // TODO: Реализовать логику промокодов
      showSuccess('Функционал промокодов скоро будет доступен!')
    }

    const proceedToCheckout = () => {
      if (!isAuthenticated.value) {
        router.push('/login?redirect=/cart')
        return
      }

      if (isEmpty.value) {
        showError('Ваша корзина пуста')
        return
      }

      // Показать модальное окно оформления заказа
      showCheckoutModal.value = true
    }

    const handleOrderPlaced = (orderData) => {
      showCheckoutModal.value = false
      showSuccess(`Заказ #${orderData.id} успешно оформлен!`)

      // Опционально перенаправить на страницу подтверждения заказа
      router.push(`/orders/${orderData.id}`)
    }

    // Жизненный цикл
    onMounted(async () => {
      await cartStore.fetchCart()
    })

    return {
      // Данные
      updatingItems,
      clearingCart,
      promoCode,
      showCheckoutModal,

      // Вычисляемые свойства
      cartItems,
      totalItems,
      totalAmount,
      isEmpty,
      loading,
      isAuthenticated,

      // Методы
      formatPrice,
      getItemsText,
      getProductForItem,
      updateQuantity,
      removeItem,
      clearAllItems,
      applyPromoCode,
      proceedToCheckout,
      handleOrderPlaced
    }
  }
}
</script>
