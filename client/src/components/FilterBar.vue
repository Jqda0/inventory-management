<template>
  <div class="filters-panel">
    <div class="filters-header">
      <span class="filters-title">FILTERS</span>
      <button
        class="reset-btn"
        @click="resetFilters"
        :disabled="!hasActiveFilters"
        title="Reset all filters"
      >
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" width="13" height="13">
          <path fill-rule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z" clip-rule="evenodd" />
        </svg>
      </button>
    </div>

    <div class="filter-group">
      <label class="filter-label">{{ t('filters.timePeriod') }}</label>
      <select v-model="selectedPeriod" class="filter-select">
        <option value="all">{{ t('filters.allMonths') }}</option>
        <option value="2025-01">{{ t('months.january') }}</option>
        <option value="2025-02">{{ t('months.february') }}</option>
        <option value="2025-03">{{ t('months.march') }}</option>
        <option value="2025-04">{{ t('months.april') }}</option>
        <option value="2025-05">{{ t('months.may') }}</option>
        <option value="2025-06">{{ t('months.june') }}</option>
        <option value="2025-07">{{ t('months.july') }}</option>
        <option value="2025-08">{{ t('months.august') }}</option>
        <option value="2025-09">{{ t('months.september') }}</option>
        <option value="2025-10">{{ t('months.october') }}</option>
        <option value="2025-11">{{ t('months.november') }}</option>
        <option value="2025-12">{{ t('months.december') }}</option>
      </select>
    </div>

    <div class="filter-group">
      <label class="filter-label">{{ t('filters.location') }}</label>
      <select v-model="selectedLocation" class="filter-select">
        <option value="all">{{ t('filters.all') }}</option>
        <option value="San Francisco">{{ t('warehouses.sanFrancisco') }}</option>
        <option value="London">{{ t('warehouses.london') }}</option>
        <option value="Tokyo">{{ t('warehouses.tokyo') }}</option>
      </select>
    </div>

    <div class="filter-group">
      <label class="filter-label">{{ t('filters.category') }}</label>
      <select v-model="selectedCategory" class="filter-select">
        <option value="all">{{ t('filters.all') }}</option>
        <option value="circuit boards">{{ t('categories.circuitBoards') }}</option>
        <option value="sensors">{{ t('categories.sensors') }}</option>
        <option value="actuators">{{ t('categories.actuators') }}</option>
        <option value="controllers">{{ t('categories.controllers') }}</option>
        <option value="power supplies">{{ t('categories.powerSupplies') }}</option>
      </select>
    </div>

    <div class="filter-group">
      <label class="filter-label">{{ t('filters.orderStatus') }}</label>
      <select v-model="selectedStatus" class="filter-select">
        <option value="all">{{ t('filters.all') }}</option>
        <option value="delivered">{{ t('status.delivered') }}</option>
        <option value="shipped">{{ t('status.shipped') }}</option>
        <option value="processing">{{ t('status.processing') }}</option>
        <option value="backordered">{{ t('status.backordered') }}</option>
      </select>
    </div>
  </div>
</template>

<script>
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'FilterBar',
  setup() {
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    } = useFilters()

    const { t } = useI18n()

    return {
      t,
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    }
  }
}
</script>

<style scoped>
.filters-panel {
  padding: 0.5rem 0.75rem 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.875rem;
}

.filters-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.5rem 0.5rem 0.25rem;
}

.filters-title {
  font-size: 0.625rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: rgba(148, 163, 184, 0.5);
}

.reset-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0.25rem;
  background: transparent;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  color: rgba(148, 163, 184, 0.6);
  cursor: pointer;
  transition: all 0.15s ease;
}

.reset-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(255, 255, 255, 0.2);
  color: #e2e8f0;
}

.reset-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  padding: 0 0.25rem;
}

.filter-label {
  font-size: 0.688rem;
  font-weight: 600;
  color: rgba(148, 163, 184, 0.7);
  letter-spacing: 0.03em;
}

.filter-select {
  width: 100%;
  padding: 0.438rem 0.625rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 5px;
  font-size: 0.813rem;
  font-family: inherit;
  font-weight: 500;
  color: #cbd5e1;
  background: rgba(255, 255, 255, 0.05);
  cursor: pointer;
  transition: border-color 0.15s ease, background 0.15s ease;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 16 16' fill='none'%3E%3Cpath d='M4 6L8 10L12 6' stroke='%2394a3b8' stroke-width='2' stroke-linecap='round'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.5rem center;
  padding-right: 1.75rem;
}

.filter-select:hover {
  border-color: rgba(255, 255, 255, 0.2);
  background-color: rgba(255, 255, 255, 0.08);
}

.filter-select:focus {
  outline: none;
  border-color: rgba(37, 99, 235, 0.7);
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.2);
}

.filter-select option {
  background: #1e293b;
  color: #e2e8f0;
}
</style>
