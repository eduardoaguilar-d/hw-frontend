<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const orders = ref([]);
const loading = ref(false);
const error = ref("");

const API_BASE_URL = "http://localhost:8000/api";

const fetchOrders = async () => {
  loading.value = true;
  error.value = "";

  try {
    const response = await axios.get(`${API_BASE_URL}/orders`);
    // Los pedidos ya vienen ordenados del más antiguo al más reciente
    orders.value = response.data.orders || [];
  } catch (e) {
    error.value = "Error al cargar los pedidos. Por favor, intenta de nuevo.";
    console.error("Error fetching orders:", e);
  } finally {
    loading.value = false;
  }
};

const updateOrderStatus = async (orderId, newStatus) => {
  try {
    const response = await axios.patch(
      `${API_BASE_URL}/orders/${orderId}/status`,
      { status: newStatus }
    );

    // Actualizar el pedido en la lista
    const index = orders.value.findIndex((order) => order.id === orderId);
    if (index !== -1) {
      orders.value[index] = response.data.order;
    }
  } catch (e) {
    error.value = `Error al actualizar el estado del pedido: ${e.response?.data?.message || e.message}`;
    console.error("Error updating order status:", e);
  }
};

const getNextStatus = (currentStatus) => {
  const statusFlow = {
    pending: "preparing",
    preparing: "completed",
    completed: null, // No hay siguiente estado
  };
  return statusFlow[currentStatus];
};

const getStatusLabel = (status) => {
  const labels = {
    pending: "Pendiente",
    preparing: "En Preparación",
    completed: "Completado",
  };
  return labels[status] || status;
};

const getStatusColor = (status) => {
  const colors = {
    pending: "bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-300",
    preparing: "bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-300",
    completed: "bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-300",
  };
  return colors[status] || "bg-gray-100 text-gray-800";
};

const getPaymentMethodLabel = (method) => {
  const labels = {
    efectivo: "Efectivo",
    transferencia: "Transferencia",
  };
  return labels[method] || method;
};

const getDeliveryTypeLabel = (type) => {
  const labels = {
    domicilio: "Domicilio",
    recoger: "Recoger",
  };
  return labels[type] || type;
};

const calculateTimeSinceReceived = (createdAt) => {
  if (!createdAt) return "0:00";
  
  const now = new Date();
  const created = new Date(createdAt);
  const diffMs = now - created;
  
  if (diffMs < 0) return "0:00";
  
  const totalSeconds = Math.floor(diffMs / 1000);
  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;
  
  if (hours > 0) {
    return `${hours}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  } else {
    return `${minutes}:${seconds.toString().padStart(2, '0')}`;
  }
};

const calculatePreparationTime = (startedAt) => {
  if (!startedAt) return null;
  
  const now = new Date();
  const started = new Date(startedAt);
  const diffMs = now - started;
  
  if (diffMs < 0) return null;
  
  const totalSeconds = Math.floor(diffMs / 1000);
  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;
  
  if (hours > 0) {
    return `${hours}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  } else {
    return `${minutes}:${seconds.toString().padStart(2, '0')}`;
  }
};

const filteredOrders = () => {
  const completed = orders.value.filter((o) => o.status === "completed");
  const active = orders.value.filter((o) => o.status !== "completed");
  
  // Ordenar completados por fecha (más reciente primero) y tomar solo los últimos 2
  const recentCompleted = completed
    .sort((a, b) => new Date(b.created_at) - new Date(a.created_at))
    .slice(0, 2);
  
  // Combinar activos con los últimos 2 completados, manteniendo orden original (más antiguo primero)
  const allOrders = [...active, ...recentCompleted];
  return allOrders.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));
};

onMounted(() => {
  fetchOrders();
  // Actualizar pedidos cada 5 segundos para obtener tiempos actualizados
  setInterval(fetchOrders, 5000);
  // Actualizar tiempos calculados cada segundo para que se vean en tiempo real
  setInterval(() => {
    // Forzar reactividad para actualizar los tiempos calculados
    orders.value = [...orders.value];
  }, 1000);
});
</script>

<template>
  <div class="w-full">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-2xl font-bold text-gray-900 dark:text-white">
        Gestión de Pedidos
      </h2>
      <button
        @click="fetchOrders"
        :disabled="loading"
        class="px-4 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50 transition-colors"
      >
        {{ loading ? "Cargando..." : "Actualizar" }}
      </button>
    </div>

    <div v-if="error" class="mb-4 p-4 bg-red-100 border border-red-400 text-red-700 rounded-lg">
      {{ error }}
    </div>

    <div v-if="loading && orders.length === 0" class="text-center py-8">
      <div class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-primary-600"></div>
      <p class="mt-2 text-gray-600 dark:text-gray-400">Cargando pedidos...</p>
    </div>

    <div v-else-if="orders.length === 0" class="text-center py-8">
      <p class="text-gray-600 dark:text-gray-400">No hay pedidos disponibles</p>
    </div>

    <div v-else class="flex flex-wrap gap-3 justify-center">
      <div
        v-for="order in filteredOrders()"
        :key="order.id"
        class="bg-white dark:bg-gray-800 rounded-lg shadow-md p-3 border border-gray-200 dark:border-gray-700"
        style="width: 300px;"
      >
        <!-- Header compacto -->
        <div class="mb-2">
          <div class="flex justify-between items-start mb-1">
            <div class="flex-1 min-w-0">
              <h3 class="text-base font-semibold text-gray-900 dark:text-white">
                #{{ order.id }}
              </h3>
              <div v-if="order.customer_name" class="text-xs text-gray-600 dark:text-gray-400 mt-0.5">
                👤 {{ order.customer_name }}
              </div>
            </div>
            <span
              :class="[
                'px-2 py-0.5 rounded-full text-xs font-medium whitespace-nowrap ml-2',
                getStatusColor(order.status),
              ]"
            >
              {{ getStatusLabel(order.status) }}
            </span>
          </div>
          
          <!-- Información de entrega y contacto -->
          <div v-if="order.delivery_type || order.phone" class="mb-1.5 space-y-0.5">
            <div v-if="order.delivery_type" class="text-xs text-gray-600 dark:text-gray-400">
              <span v-if="order.delivery_type === 'domicilio'" class="text-orange-600 dark:text-orange-400">
                🚚 {{ getDeliveryTypeLabel(order.delivery_type) }}
              </span>
              <span v-else class="text-blue-600 dark:text-blue-400">
                📍 {{ getDeliveryTypeLabel(order.delivery_type) }}
              </span>
            </div>
            <div v-if="order.address && order.delivery_type === 'domicilio'" class="text-xs text-gray-500 dark:text-gray-400 truncate">
              📍 {{ order.address }}
            </div>
            <div v-if="order.phone" class="text-xs text-gray-500 dark:text-gray-400">
              📞 {{ order.phone }}
            </div>
          </div>

          <!-- Tiempos y método de pago -->
          <div class="flex flex-wrap gap-2 text-xs text-gray-500 dark:text-gray-400">
            <span>
              ⏱️ {{ calculateTimeSinceReceived(order.created_at) }}
            </span>
            <span v-if="order.payment_method" class="capitalize">
              💳 {{ getPaymentMethodLabel(order.payment_method) }}
            </span>
            <span v-if="order.started_at && calculatePreparationTime(order.started_at)" class="text-blue-600 dark:text-blue-400">
              🔧 {{ calculatePreparationTime(order.started_at) }}
            </span>
          </div>

          <!-- Recompensas/Descuentos -->
          <div v-if="order.discount || order.reward_earned || order.rewards_progress" class="mt-1.5 p-1.5 bg-green-50 dark:bg-green-900/20 rounded text-xs">
            <div v-if="order.discount" class="text-green-700 dark:text-green-300 font-medium">
              🎁 Descuento: {{ order.discount }}%
            </div>
            <div v-if="order.discount_message" class="text-green-600 dark:text-green-400 mt-0.5">
              {{ order.discount_message }}
            </div>
            <div v-if="order.reward_earned" class="text-green-700 dark:text-green-300 mt-0.5">
              {{ order.reward_earned.message }}
            </div>
            <div v-if="order.completed_orders_count" class="text-green-600 dark:text-green-400 mt-0.5">
              Pedidos completados: {{ order.completed_orders_count }}
            </div>
            <div v-if="order.rewards_progress" class="text-green-600 dark:text-green-400 mt-0.5">
              {{ order.rewards_progress.message }}
            </div>
          </div>
        </div>

        <!-- Items compactos -->
        <div class="mb-2 space-y-1.5">
          <div
            v-for="item in order.items"
            :key="item.id"
            class="text-xs"
          >
            <div class="flex justify-between items-start">
              <div class="flex-1 min-w-0">
                <p class="font-medium text-gray-900 dark:text-white truncate">
                  {{ item.quantity }}x {{ item.product.name }}
                </p>
                <!-- Ingredientes excluidos -->
                <div v-if="item.excluded_ingredients && item.excluded_ingredients.length > 0" class="flex flex-wrap gap-0.5 mt-0.5">
                  <span class="text-[10px] text-red-600 dark:text-red-400 font-medium mr-0.5">Sin:</span>
                  <span
                    v-for="ingredient in item.excluded_ingredients"
                    :key="ingredient.id"
                    class="px-1 py-0.5 bg-red-100 dark:bg-red-900/30 text-red-800 dark:text-red-300 rounded text-[10px]"
                  >
                    {{ ingredient.name }}
                  </span>
                </div>
                <!-- Si no hay ingredientes excluidos, mostrar que tiene ingredientes por defecto -->
                <div v-else-if="item.product.default_ingredients && item.product.default_ingredients.length > 0" class="text-[10px] text-gray-500 dark:text-gray-400 mt-0.5">
                  Con ingredientes por defecto
                </div>
              </div>
              <span class="text-xs font-medium text-gray-600 dark:text-gray-400 ml-1 whitespace-nowrap">
                ${{ (parseFloat(item.product.price) * item.quantity).toFixed(2) }}
              </span>
            </div>
          </div>
        </div>

        <!-- Footer con total y botón -->
        <div class="pt-2 border-t border-gray-200 dark:border-gray-700">
          <div class="mb-2">
            <div class="flex justify-between items-center">
              <span class="text-xs text-gray-600 dark:text-gray-400">Subtotal:</span>
              <span class="text-xs text-gray-600 dark:text-gray-400">
                ${{ order.items.reduce((sum, item) => sum + parseFloat(item.product.price) * item.quantity, 0).toFixed(2) }}
              </span>
            </div>
            <div v-if="order.discount" class="flex justify-between items-center mt-0.5">
              <span class="text-xs text-green-600 dark:text-green-400">Descuento ({{ order.discount }}%):</span>
              <span class="text-xs text-green-600 dark:text-green-400">
                -${{ (order.items.reduce((sum, item) => sum + parseFloat(item.product.price) * item.quantity, 0) * order.discount / 100).toFixed(2) }}
              </span>
            </div>
            <div class="flex justify-between items-center mt-1 pt-1 border-t border-gray-200 dark:border-gray-700">
              <span class="text-sm font-semibold text-gray-900 dark:text-white">Total:</span>
              <span class="text-sm font-bold text-primary-600 dark:text-primary-400">
                ${{ (order.items.reduce((sum, item) => sum + parseFloat(item.product.price) * item.quantity, 0) * (1 - (order.discount || 0) / 100)).toFixed(2) }}
              </span>
            </div>
          </div>
          <button
            v-if="getNextStatus(order.status)"
            @click="updateOrderStatus(order.id, getNextStatus(order.status))"
            class="w-full px-3 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 active:bg-primary-800 transition-colors text-sm font-medium min-h-[44px]"
          >
            {{
              order.status === "pending"
                ? "Iniciar"
                : order.status === "preparing"
                ? "Completar"
                : ""
            }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

