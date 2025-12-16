<template>
  <div class="q-pa-md bg-grey-2">

    <div class="row items-center q-gutter-md">

      <!-- Bazar Calculator Link -->
      <router-link
        to="/bazar-calculator"
        class="nav-link bg-green-1"
      >
        <q-icon name="shopping_cart" size="20px" class="q-mr-sm" />
        বাজার ক্যালকুলেটর
      </router-link>

      <!-- Monthly Calculator Link -->
      <router-link
        to="/monthly-calculator"
        class="nav-link bg-blue-1"
      >
        <q-icon name="calendar_month" size="20px" class="q-mr-sm" />
        মাসিক ক্যালকুলেটর
      </router-link>

    </div>

  </div>
</template>

<script setup>
</script>

<style scoped>
.nav-link {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  border-radius: 6px;
  text-decoration: none;
  color: #000;
  font-weight: 500;
}
</style>
