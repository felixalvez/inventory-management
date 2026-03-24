<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Budget Control Card -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
          <span class="budget-display">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
        </div>
        <div class="slider-wrapper">
          <input
            type="range"
            class="budget-slider"
            :min="1000"
            :max="50000"
            :step="500"
            v-model.number="budget"
          />
          <div class="slider-labels">
            <span>{{ currencySymbol }}1,000</span>
            <span>{{ currencySymbol }}50,000</span>
          </div>
        </div>

        <!-- Summary Stats Bar -->
        <div class="stats-grid summary-stats">
          <div class="stat-card info">
            <div class="stat-label">{{ t('restocking.summary.totalItems') }}</div>
            <div class="stat-value">{{ recommendations.length }}</div>
          </div>
          <div class="stat-card info">
            <div class="stat-label">{{ t('restocking.summary.totalUnits') }}</div>
            <div class="stat-value">{{ totalUnits.toLocaleString() }}</div>
          </div>
          <div class="stat-card warning">
            <div class="stat-label">{{ t('restocking.summary.totalSpend') }}</div>
            <div class="stat-value">{{ currencySymbol }}{{ totalSpend.toLocaleString() }}</div>
          </div>
          <div :class="['stat-card', budgetRemaining >= 0 ? 'success' : 'danger']">
            <div class="stat-label">{{ t('restocking.summary.budgetRemaining') }}</div>
            <div class="stat-value">{{ currencySymbol }}{{ budgetRemaining.toLocaleString() }}</div>
          </div>
        </div>
      </div>

      <!-- Recommendations Table Card -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendations') }} ({{ recommendations.length }})</h3>
          <button
            class="place-order-btn"
            :disabled="recommendations.length === 0 || !!submittedOrder || submitting"
            @click="placeOrder"
          >
            {{ submitting ? t('common.loading') : t('restocking.placeOrder') }}
          </button>
        </div>

        <div v-if="recommendations.length === 0" class="empty-state">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="table-container">
          <table class="restock-table">
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th class="col-number">{{ t('restocking.table.forecastedDemand') }}</th>
                <th class="col-number">{{ t('restocking.table.unitCost') }}</th>
                <th class="col-number">{{ t('restocking.table.recommendedQty') }}</th>
                <th class="col-number">{{ t('restocking.table.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.sku">
                <td><span class="badge info">{{ item.sku }}</span></td>
                <td>{{ item.name }}</td>
                <td class="col-number">{{ item.forecasted_demand.toLocaleString() }}</td>
                <td class="col-number">{{ currencySymbol }}{{ item.unit_cost.toLocaleString() }}</td>
                <td class="col-number"><strong>{{ item.recommended_quantity.toLocaleString() }}</strong></td>
                <td class="col-number"><strong>{{ currencySymbol }}{{ item.line_total.toLocaleString() }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Success Card -->
      <div v-if="submittedOrder" class="card success-card">
        <div class="success-content">
          <div class="success-icon">&#10003;</div>
          <div>
            <h3 class="card-title">{{ t('restocking.orderPlaced') }}</h3>
            <p class="success-detail">
              {{ t('restocking.orderNumber') }}: <strong>{{ submittedOrder.order_number }}</strong>
            </p>
            <p class="success-detail">
              {{ t('restocking.expectedDelivery') }}: <strong>{{ formatDate(submittedOrder.expected_delivery) }}</strong>
            </p>
            <p v-if="submittedOrder.lead_time_days" class="success-detail">
              {{ t('restocking.leadTime') }}: <strong>{{ submittedOrder.lead_time_days }} days</strong>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency, currentLocale } = useI18n()

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    const budget = ref(10000)
    const demandForecasts = ref([])
    const inventoryItems = ref([])
    const loading = ref(false)
    const error = ref(null)
    const submitting = ref(false)
    const submittedOrder = ref(null)

    const loadData = async () => {
      loading.value = true
      error.value = null
      try {
        const [forecasts, inventory] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        demandForecasts.value = forecasts
        inventoryItems.value = inventory
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const recommendations = computed(() => {
      // Join demand forecasts with inventory items by SKU
      const joined = demandForecasts.value
        .map(forecast => {
          const item = inventoryItems.value.find(i => i.sku === forecast.item_sku)
          if (!item) return null
          return {
            sku: forecast.item_sku,
            name: item.name,
            forecasted_demand: forecast.forecasted_demand,
            unit_cost: item.unit_cost
          }
        })
        .filter(Boolean)

      // Sort by forecasted_demand descending
      joined.sort((a, b) => b.forecasted_demand - a.forecasted_demand)

      // Greedy allocation
      let remainingBudget = budget.value
      const result = []

      for (const item of joined) {
        if (remainingBudget <= 0) break
        const qty = Math.min(
          Math.floor(remainingBudget / item.unit_cost),
          item.forecasted_demand
        )
        if (qty > 0) {
          const line_total = qty * item.unit_cost
          result.push({
            sku: item.sku,
            name: item.name,
            forecasted_demand: item.forecasted_demand,
            unit_cost: item.unit_cost,
            recommended_quantity: qty,
            line_total
          })
          remainingBudget -= line_total
        }
      }

      return result
    })

    const totalSpend = computed(() =>
      recommendations.value.reduce((sum, item) => sum + item.line_total, 0)
    )

    const totalUnits = computed(() =>
      recommendations.value.reduce((sum, item) => sum + item.recommended_quantity, 0)
    )

    const budgetRemaining = computed(() => budget.value - totalSpend.value)

    const formatDate = (dateString) => {
      const date = new Date(dateString)
      if (isNaN(date.getTime())) return dateString
      const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
      return date.toLocaleDateString(locale, {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    const placeOrder = async () => {
      submitting.value = true
      try {
        const response = await api.submitRestockOrder({
          items: recommendations.value.map(item => ({
            sku: item.sku,
            name: item.name,
            quantity: item.recommended_quantity,
            unit_cost: item.unit_cost
          })),
          total_budget: budget.value
        })
        submittedOrder.value = response
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
        console.error(err)
      } finally {
        submitting.value = false
      }
    }

    onMounted(() => loadData())

    return {
      t,
      currencySymbol,
      budget,
      loading,
      error,
      submitting,
      submittedOrder,
      recommendations,
      totalSpend,
      totalUnits,
      budgetRemaining,
      formatDate,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

.budget-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #2563eb;
  letter-spacing: -0.025em;
}

.slider-wrapper {
  padding: 0.5rem 0 1.25rem;
}

.budget-slider {
  width: 100%;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
  margin-bottom: 0.5rem;
}

.slider-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.813rem;
  color: #64748b;
  margin-top: 0.375rem;
}

.summary-stats {
  margin-bottom: 0;
}

.restock-table {
  width: 100%;
  border-collapse: collapse;
}

.col-number {
  text-align: right;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

.place-order-btn {
  background: #059669;
  color: white;
  border: none;
  padding: 0.75rem 2rem;
  border-radius: 8px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #047857;
}

.place-order-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.success-card {
  border-left: 4px solid #059669;
  background: #f0fdf4;
}

.success-content {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
}

.success-icon {
  width: 2rem;
  height: 2rem;
  background: #059669;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  font-weight: 700;
  flex-shrink: 0;
}

.success-detail {
  color: #374151;
  font-size: 0.938rem;
  margin-top: 0.375rem;
}
</style>
