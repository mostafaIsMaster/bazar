<template>
  <div class="q-pa-md bg-grey-2">

    <div class="row q-col-gutter-md">

      <!-- BAZAR CALCULATOR -->
      <div class="col-12 col-sm-6">
        <q-card
          clickable
          @click="go('/bazar-calculator')"
          class="calculator-card bg-green-1"
        >
          <q-card-section class="flex items-center">
            <q-icon name="shopping_cart" size="40px" color="green" />
            <div class="q-ml-md">
              <div class="text-h6">বাজার ক্যালকুলেটর</div>
              <div class="text-caption text-grey-7">
                লাভ ও বিক্রয় মূল্য হিসাব
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- MONTHLY CALCULATOR -->
      <div class="col-12 col-sm-6">
        <q-card
          clickable
          @click="go('/monthly-calculator')"
          class="calculator-card bg-blue-1"
        >
          <q-card-section class="flex items-center">
            <q-icon name="calendar_month" size="40px" color="blue" />
            <div class="q-ml-md">
              <div class="text-h6">মাসিক ক্যালকুলেটর</div>
              <div class="text-caption text-grey-7">
                মাসিক মোট হিসাব
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>

    </div>

  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

function go (path) {
  router.push(path)
}
</script>

<style scoped lang="scss">
.calculator-card {
  border-radius: 12px;
  transition: 0.2s ease;
  cursor: pointer;
}

.calculator-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.12);
}
</style>
