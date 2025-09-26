<!-- frontend/src/views/CatalogView.vue -->
<template>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
    <!-- Header -->
    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between mb-10">
      <div>
        <h1 class="text-4xl font-extrabold text-gray-900 mb-3">Каталог товаров</h1>
        <p class="text-gray-600 text-lg">Откройте для себя нашу удивительную коллекцию товаров</p>
      </div>
      <div class="mt-4 lg:mt-0">
        <p class="text-sm text-gray-500">
          Показано {{ startItem }}-{{ endItem }} из {{ totalProducts }} товаров
        </p>
      </div>
    </div>

    <div class="lg:grid lg:grid-cols-4 lg:gap-8">
      <!-- Filters Sidebar -->
      <div class="lg:col-span-1">
        <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 sticky top-8">
          <h3 class="text-xl font-semibold text-gray-900 mb-4">Фильтры</h3>

          <!-- Search -->
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">Поиск</label>
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Поиск товаров..."
              class="w-full px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
              @keyup.enter="applyFilters"
            >
          </div>

          <!-- Categories -->
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">Категория</label>
            <select
              v-model="selectedCategory"
              class="w-full px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
              @change="applyFilters"
            >
              <option value="">Все категории</option>
              <option v-for="category in categories" :key="category.id" :value="category.id">
                {{ category.name }} ({{ category.products_count }})
              </option>
            </select>
          </div>

          <!-- Price Range -->
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">Диапазон цен</label>
            <div class="grid grid-cols-2 gap-2">
              <input
                v-model="priceRange.min"
                type="number"
                placeholder="Мин"
                class="px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
                @blur="applyFilters"
              >
              <input
                v-model="priceRange.max"
                type="number"
                placeholder="Макс"
                class="px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
                @blur="applyFilters"
              >
            </div>
          </div>

          <!-- In Stock Only -->
          <div class="mb-6">
            <label class="flex items-center">
              <input
                v-model="inStockOnly"
                type="checkbox"
                class="rounded border-gray-200 text-black focus:ring-black focus:ring-offset-0"
                @change="applyFilters"
              >
              <span class="ml-2 text-sm text-gray-700">Только в наличии</span>
            </label>
          </div>

          <!-- Sort By -->
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">Сортировка</label>
            <select
              v-model="sortBy"
              class="w-full px-4 py-2 border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-black text-sm"
              @change="applyFilters"
            >
              <option value="-created_at">Сначала новые</option>
              <option value="created_at">Сначала старые</option>
              <option value="name">Название А-Я</option>
              <option value="-name">Название Я-А</option>
              <option value="price">Цена по возрастанию</option>
              <option value="-price">Цена по убыванию</option>
            </select>
          </div>

          <!-- Clear Filters -->
          <BaseButton
            variant="outline"
            size="sm"
            block
            @click="clearFilters"
          >
            Сбросить фильтры
          </BaseButton>
        </div>
      </div>

      <!-- Products Grid -->
      <div class="lg:col-span-3 mt-8 lg:mt-0">
        <!-- Loading State -->
        <div v-if="loading" class="flex justify-center py-16">
          <LoadingSpinner size="lg" text="Загрузка товаров..." />
        </div>

        <!-- Products Grid -->
        <div v-else-if="products.length > 0" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
          <ProductCard
            v-for="product in products"
            :key="product.id"
            :product="product"
            @add-to-cart="handleAddToCart"
          />
        </div>

        <!-- Empty State -->
        <div v-else class="text-center py-16 bg-white rounded-xl shadow-sm">
          <svg class="mx-auto h-16 w-16 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
          <h3 class="mt-3 text-xl font-semibold text-gray-900">Товары не найдены</h3>
          <p class="mt-2 text-sm text-gray-500">Попробуйте изменить параметры поиска или фильтры.</p>
          <div class="mt-6">
            <BaseButton variant="primary" @click="clearFilters">
              Сбросить фильтры
            </BaseButton>
          </div>
        </div>

        <!-- Pagination -->
        <div v-if="totalPages > 1" class="mt-8 flex items-center justify-between">
          <div class="flex items-center">
            <p class="text-sm text-gray-600">
              Показано
              <span class="font-medium">{{ startItem }}</span>
              до
              <span class="font-medium">{{ endItem }}</span>
              из
              <span class="font-medium">{{ totalProducts }}</span>
              результатов
            </p>
          </div>

          <div class="flex items-center space-x-2">
            <BaseButton
              variant="outline"
              size="sm"
              :disabled="currentPage === 1"
              @click="changePage(currentPage - 1)"
            >
              Назад
            </BaseButton>

            <div class="flex items-center space-x-1">
              <button
                v-for="page in visiblePages"
                :key="page"
                @click="changePage(page)"
                :class="[
                  'px-4 py-2 text-sm font-medium rounded-lg transition-colors duration-200',
                  page === currentPage
                    ? 'bg-black text-white'
                    : 'text-gray-600 hover:bg-gray-100'
                ]"
              >
                {{ page }}
              </button>
            </div>

            <BaseButton
              variant="outline"
              size="sm"
              :disabled="currentPage === totalPages"
              @click="changePage(currentPage + 1)"
            >
              Вперед
            </BaseButton>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import BaseButton from '../components/common/BaseButton.vue'
import LoadingSpinner from '../components/common/LoadingSpinner.vue'
import ProductCard from '../components/product/ProductCard.vue'
import { useProductsStore } from '../store/products.js'
import { useCartStore } from '../store/cart.js'
import { useAuthStore } from '../store/auth.js'
import { useToast } from '../composables/useToast.js'

export default {
  name: 'CatalogView',
  components: {
    BaseButton,
    LoadingSpinner,
    ProductCard
  },
  setup() {
    const route = useRoute()
    const router = useRouter()
    const productsStore = useProductsStore()
    const cartStore = useCartStore()
    const authStore = useAuthStore()
    const { showSuccess, showError } = useToast()

    // Reactive data
    const loading = ref(false)
    const searchQuery = ref('')
    const selectedCategory = ref('')
    const priceRange = ref({ min: '', max: '' })
    const inStockOnly = ref(false)
    const sortBy = ref('-created_at')
    const currentPage = ref(1)

    // Computed properties
    const products = computed(() => productsStore.products)
    const categories = computed(() => productsStore.categories)
    const totalProducts = computed(() => productsStore.pagination.totalCount)
    const totalPages = computed(() => productsStore.pagination.totalPages)

    const startItem = computed(() => {
      if (totalProducts.value === 0) return 0
      return (currentPage.value - 1) * 20 + 1
    })

    const endItem = computed(() => {
      const end = currentPage.value * 20
      return end > totalProducts.value ? totalProducts.value : end
    })

    const visiblePages = computed(() => {
      const pages = []
      const total = totalPages.value
      const current = currentPage.value

      if (total <= 7) {
        for (let i = 1; i <= total; i++) {
          pages.push(i)
        }
      } else {
        if (current <= 4) {
          for (let i = 1; i <= 5; i++) {
            pages.push(i)
          }
          pages.push('...')
          pages.push(total)
        } else if (current >= total - 3) {
          pages.push(1)
          pages.push('...')
          for (let i = total - 4; i <= total; i++) {
            pages.push(i)
          }
        } else {
          pages.push(1)
          pages.push('...')
          for (let i = current - 1; i <= current + 1; i++) {
            pages.push(i)
          }
          pages.push('...')
          pages.push(total)
        }
      }

      return pages
    })

    // Methods
    const initializeFromQuery = () => {
      searchQuery.value = route.query.search || ''
      selectedCategory.value = route.query.category || ''
      priceRange.value.min = route.query.min_price || ''
      priceRange.value.max = route.query.max_price || ''
      inStockOnly.value = route.query.in_stock === 'true'
      sortBy.value = route.query.sort || '-created_at'
      currentPage.value = parseInt(route.query.page) || 1
    }

    const updateQueryParams = () => {
      const query = {}

      if (searchQuery.value) query.search = searchQuery.value
      if (selectedCategory.value) query.category = selectedCategory.value
      if (priceRange.value.min) query.min_price = priceRange.value.min
      if (priceRange.value.max) query.max_price = priceRange.value.max
      if (inStockOnly.value) query.in_stock = 'true'
      if (sortBy.value !== '-created_at') query.sort = sortBy.value
      if (currentPage.value > 1) query.page = currentPage.value

      router.replace({ query })
    }

    const fetchProducts = async () => {
      try {
        loading.value = true

        const params = {
          page: currentPage.value,
          ordering: sortBy.value
        }

        if (searchQuery.value) params.search = searchQuery.value
        if (selectedCategory.value) params.category = selectedCategory.value
        if (priceRange.value.min) params.min_price = priceRange.value.min
        if (priceRange.value.max) params.max_price = priceRange.value.max
        if (inStockOnly.value) params.in_stock = 'true'

        const result = await productsStore.fetchProducts(params)
        if (!result.success) {
          showError('Не удалось загрузить товары')
        }
      } catch (error) {
        console.error('Ошибка при загрузке товаров:', error)
        showError('Не удалось загрузить товары')
      } finally {
        loading.value = false
      }
    }

    const applyFilters = () => {
      currentPage.value = 1
      updateQueryParams()
      fetchProducts()
    }

    const changePage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        currentPage.value = page
        updateQueryParams()
        fetchProducts()
        // Scroll to top
        window.scrollTo({ top: 0, behavior: 'smooth' })
      }
    }

    const clearFilters = () => {
      searchQuery.value = ''
      selectedCategory.value = ''
      priceRange.value = { min: '', max: '' }
      inStockOnly.value = false
      sortBy.value = '-created_at'
      currentPage.value = 1

      router.replace({ query: {} })
      fetchProducts()
    }

    const handleAddToCart = async (product) => {
      try {
        if (!authStore.isAuthenticated) {
          showError('Пожалуйста, войдите в систему, чтобы добавлять товары в корзину')
          router.push('/login')
          return
        }

        const result = await cartStore.addToCart(product.id, 1)
        if (result.success) {
          showSuccess(`${product.name} добавлен в корзину!`)
        } else {
          showError(result.error || 'Не удалось добавить товар в корзину')
        }
      } catch (error) {
        console.error('Ошибка при добавлении в корзину:', error)
        showError('Не удалось добавить товар в корзину')
      }
    }

    // Watchers
    watch(() => route.query, () => {
      initializeFromQuery()
      fetchProducts()
    })

    // Lifecycle
    onMounted(async () => {
      initializeFromQuery()

      // Fetch categories first
      await productsStore.fetchCategories()

      // Then fetch products
      await fetchProducts()
    })

    return {
      // Data
      loading,
      searchQuery,
      selectedCategory,
      priceRange,
      inStockOnly,
      sortBy,
      currentPage,

      // Computed
      products,
      categories,
      totalProducts,
      totalPages,
      startItem,
      endItem,
      visiblePages,

      // Methods
      applyFilters,
      changePage,
      clearFilters,
      handleAddToCart
    }
  }
}
</script>
