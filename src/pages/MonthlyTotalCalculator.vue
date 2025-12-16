<template>
  <div class="q-pa-sm">
     <Glo-Header></Glo-Header>
    
    <!-- Today's Entry Section -->
    <q-card class="q-mb-sm">
      <q-card-section class="bg-primary text-white q-pa-sm">
        <div class="text-subtitle1">Today | <span class="text-caption">{{ todayDate }}</span></div>
        <div ></div>
      </q-card-section>
      
      <q-card-section class="q-pa-sm">
        <div class="row q-col-gutter-sm">
          <div class="col-6">
            <q-input 
              v-model="todayEntry.kena"
              label="Kena"
              type="number"
              filled
              dense
              @update:model-value="calculateTodayProfit"
            />
          </div>
          <div class="col-6">
            <q-input 
              v-model="todayEntry.becha"
              label="Becha"
              type="number"
              filled
              dense
              @update:model-value="calculateTodayProfit"
            />
          </div>
        </div>
        
        <!-- Today's Profit -->
        <div class="row">
          <!-- <div class="col-8">
            <div class="text-caption">Today's Profit:</div>
            <div 
              class="text-h6"
              :class="{
                'text-positive': todayEntry.profit > 0,
                'text-negative': todayEntry.profit < 0,
                'text-grey': todayEntry.profit === 0
              }"
            >
              {{ todayEntry.profit }} TK
            </div>
          </div> -->
<div class="q-pa-md">
  <q-btn 
    color="primary" 
    label="Save" 
    @click="saveTodayEntry"
    :disabled="!canSaveToday"
    class="full-width"
  />
</div>

        </div>
        
        <!-- Last 3 Days Data -->
        <div class="q-mt-sm" v-if="lastThreeDays.length > 0">
          <div class="text-caption q-mb-xs">Last 3 Days:</div>
          <div class="row">
            <div 
              v-for="day in lastThreeDays" 
              :key="day.date"
              class="col-4 q-px-xs"
            >
              <q-card flat class="bg-grey-2 q-pa-xs">
                <div class="text-center text-caption">{{ day.date }}</div>
                <div class="text-center">
                  <span class="text-red">{{ day.kena }}→</span>
                  <span class="text-blue">{{ day.becha }}</span>
                </div>
                <div 
                  class="text-center text-bold"
                  :class="{
                    'text-positive': day.profit > 0,
                    'text-negative': day.profit < 0
                  }"
                >
                  {{ day.profit }}
                </div>
              </q-card>
            </div>
          </div>
        </div>
      </q-card-section>
    </q-card>
    
    <!-- Monthly Table -->
    <q-card>
      <q-card-section class="q-pa-sm">
        <div class="text-subtitle1 q-mb-xs">
          {{ currentMonthName }} {{ currentYear }}
          <q-btn 
            round 
            dense 
            flat 
            icon="chevron_left" 
            size="sm"
            @click="previousMonth"
            class="q-ml-xs"
          />
          <q-btn 
            round 
            dense 
            flat 
            icon="chevron_right" 
            size="sm"
            @click="nextMonth"
            class="q-ml-xs"
          />
        </div>
        
        <!-- Table -->
        <div class="table-container">
          <div class="table-header row text-bold bg-grey-3 q-pa-xs">
            <div class="col-3">Date</div>
            <div class="col-3">Kena</div>
            <div class="col-3">Becha</div>
            <div class="col-3">Profit</div>
          </div>
          
          <div 
            v-for="day in daysInMonth" 
            :key="day"
            class="table-row row q-pa-xs border-bottom"
            :class="{
              'bg-blue-1': isToday(day)
            }"
          >
            <div class="col-3">
              {{ day }}
              <q-icon 
                v-if="isToday(day)" 
                name="circle" 
                size="8px" 
                class="q-ml-xs text-primary"
              />
            </div>
            
            <div class="col-3">
              <template v-if="hasTransaction(day)">
                {{ getTransaction(day).kena || 0 }}
              </template>
              <template v-else>
                <q-btn 
                  round 
                  dense 
                  icon="add" 
                  size="xs"
                  @click="editDay(day)"
                  class="bg-grey-4"
                />
              </template>
            </div>
            
            <div class="col-3">
              <template v-if="hasTransaction(day)">
                {{ getTransaction(day).becha || 0 }}
              </template>
            </div>
            
            <div class="col-3">
              <template v-if="hasTransaction(day)">
                <span 
                  :class="{
                    'text-positive': getProfit(day) > 0,
                    'text-negative': getProfit(day) < 0,
                    'text-bold': getProfit(day) !== 0
                  }"
                >
                  {{ getProfit(day) }}
                </span>
              </template>
            </div>
          </div>
        </div>
      </q-card-section>
    </q-card>
    
    <!-- Summary -->
    <q-card class="q-mt-sm">
      <q-card-section class="q-pa-sm">
        <div class="text-subtitle1 q-mb-xs">Summary</div>
        
        <div class="row">
          <div class="col-4 text-center">
            <div class="text-caption">Kena</div>
            <div class="text-h6 text-red">{{ totalKena }}</div>
          </div>
          <div class="col-4 text-center">
            <div class="text-caption">Becha</div>
            <div class="text-h6 text-blue">{{ totalBecha }}</div>
          </div>
          <div class="col-4 text-center">
            <div class="text-caption">Profit</div>
            <div 
              class="text-h5"
              :class="{
                'text-positive': totalProfit > 0,
                'text-negative': totalProfit < 0
              }"
            >
              {{ totalProfit }}
            </div>
          </div>
        </div>
      </q-card-section>
    </q-card>
    
    <!-- Edit Dialog -->
    <q-dialog v-model="editDialog" position="bottom">
      <q-card style="width: 100%">
        <q-card-section class="q-pa-sm">
          <div class="text-subtitle1">
            Day {{ editingDay }}
          </div>
        </q-card-section>
        
        <q-card-section class="q-pt-none q-pa-sm">
          <q-input 
            v-model="editingData.kena"
            label="Kena"
            type="number"
            filled
            dense
            class="q-mb-xs"
          />
          <q-input 
            v-model="editingData.becha"
            label="Becha"
            type="number"
            filled
            dense
            class="q-mb-xs"
          />
          
          <div class="q-mt-xs text-caption">
            Profit: <span :class="{
              'text-positive': editingProfit > 0,
              'text-negative': editingProfit < 0
            }">{{ editingProfit }} TK</span>
          </div>
        </q-card-section>
        
        <q-card-actions align="right" class="q-pa-sm">
          <q-btn flat label="Cancel" color="primary" v-close-popup dense />
          <q-btn 
            label="Save" 
            color="primary" 
            @click="saveEditedDay"
            v-close-popup
            dense
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <!-- Add this after the Summary section -->
<q-card class="q-mt-sm">
  <q-card-section class="q-pa-sm">
    <div class="text-subtitle1 q-mb-xs">Print Report</div>
    <div class="row q-col-gutter-xs">
      <div class="col-6">
        <q-btn
          label="Print Today"
          color="primary"
          class="full-width"
          @click="printTodayReport"
          :disable="!hasTransaction(todayDay)"
          dense
          icon="print"
        />
      </div>
      <div class="col-6">
        <q-btn
          label="Print Month"
          color="blue-grey"
          class="full-width"
          @click="printMonthReport"
          :disable="totalKena === 0"
          dense
          icon="description"
        />
      </div>
    </div>
    
    <!-- Print options (shown when viewing month) -->
    <div class="q-mt-sm" v-if="printMonthMode">
      <div class="row q-col-gutter-xs">
        <div class="col-6">
          <q-checkbox
            v-model="printWithDetails"
            label="With Details"
            dense
          />
        </div>
        <div class="col-6">
          <q-checkbox
            v-model="printWithSummary"
            label="With Summary"
            dense
            :disable="!printWithDetails"
          />
        </div>
      </div>
    </div>
  </q-card-section>
</q-card>
<!-- Print Preview Dialog -->
<q-dialog v-model="showPrintPreview" maximized>
  <q-card>
    <q-toolbar class="bg-primary text-white">
      <q-toolbar-title class="text-caption">Print Preview</q-toolbar-title>
      <q-btn flat round dense icon="close" v-close-popup />
    </q-toolbar>
    
    <q-card-section class="q-pa-sm">
      <!-- Print Controls -->
      <div class="row q-mb-md q-pa-sm bg-grey-3 rounded-borders">
        <div class="col-12">
          <div class="row items-center justify-between">
            <div class="text-bold">Print Settings</div>
            <q-btn 
              label="Print Now" 
              color="primary" 
              @click="executePrint" 
              icon="print"
              dense
            />
          </div>
        </div>
      </div>
      
      <!-- Print Preview Content -->
      <div id="printContent" class="print-content">
        <!-- Today's Report -->
        <div v-if="printType === 'today'" class="today-report-format">
          <div class="text-center q-mb-md">
            <h2 style="margin: 0; color: #1976d2">Daily Profit Report</h2>
            <div style="color: #666; margin-top: 8px; border-bottom: 1px solid #ccc; padding-bottom: 8px">
              তারিখ: {{ todayDate }}
            </div>
          </div>
          
          <div style="margin: 20px 0; padding: 15px; background-color: #f5f5f5; border-radius: 8px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 10px;">
              <div>
                <div style="font-weight: bold; color: #d32f2f">ক্রয় মূল্য (Kena):</div>
                <div style="font-size: 1.2em; font-weight: bold; color: #d32f2f">{{ todayEntry.kena || 0 }} TK</div>
              </div>
              <div>
                <div style="font-weight: bold; color: #1976d2">বিক্রয় মূল্য (Becha):</div>
                <div style="font-size: 1.2em; font-weight: bold; color: #1976d2">{{ todayEntry.becha || 0 }} TK</div>
              </div>
            </div>
            
            <div style="text-align: center; margin-top: 15px; padding-top: 15px; border-top: 2px dashed #ccc;">
              <div style="font-weight: bold; color: #666; margin-bottom: 5px">মোট লাভ (Profit):</div>
              <div 
                style="font-size: 1.5em; font-weight: bold;"
                :style="{ color: todayEntry.profit > 0 ? '#4caf50' : todayEntry.profit < 0 ? '#f44336' : '#666' }"
              >
                {{ todayEntry.profit || 0 }} TK
              </div>
            </div>
          </div>
          
          <!-- Status Badge -->
          <div style="text-align: center; margin-top: 20px;">
            <div 
              style="display: inline-block; padding: 8px 16px; border-radius: 20px; font-weight: bold;"
              :style="{ 
                backgroundColor: todayEntry.profit > 0 ? '#e8f5e8' : todayEntry.profit < 0 ? '#ffebee' : '#f5f5f5',
                color: todayEntry.profit > 0 ? '#2e7d32' : todayEntry.profit < 0 ? '#c62828' : '#666'
              }"
            >
              <span v-if="todayEntry.profit > 0">✓ লাভ হয়েছে</span>
              <span v-else-if="todayEntry.profit < 0">✗ ক্ষতি হয়েছে</span>
              <span v-else>● সমান আছে</span>
            </div>
          </div>
          
          <div style="margin-top: 30px; text-align: center; color: #999; font-size: 0.9em">
            <div>Kena-Becha Profit Tracker</div>
            <div>প্রিন্টের সময়: {{ currentTime }}</div>
          </div>
        </div>
        
        <!-- Monthly Report -->
        <div v-else class="monthly-report-format">
          <div class="text-center q-mb-md">
            <h2 style="margin: 0; color: #1976d2">Monthly Profit Report</h2>
            <div style="color: #666; margin-top: 8px; border-bottom: 1px solid #ccc; padding-bottom: 8px">
              মাস: {{ currentMonthName }} {{ currentYear }}
            </div>
          </div>
          
          <!-- Monthly Summary -->
          <div style="margin: 20px 0; padding: 15px; background-color: #f5f5f5; border-radius: 8px;">
            <div style="display: flex; justify-content: space-around; text-align: center;">
              <div>
                <div style="font-weight: bold; color: #d32f2f">মোট ক্রয়</div>
                <div style="font-size: 1.2em; font-weight: bold; color: #d32f2f">{{ totalKena }} TK</div>
              </div>
              <div>
                <div style="font-weight: bold; color: #1976d2">মোট বিক্রয়</div>
                <div style="font-size: 1.2em; font-weight: bold; color: #1976d2">{{ totalBecha }} TK</div>
              </div>
              <div>
                <div style="font-weight: bold; color: '#4caf50'">মোট লাভ</div>
                <div 
                  style="font-size: 1.2em; font-weight: bold;"
                  :style="{ color: totalProfit > 0 ? '#4caf50' : totalProfit < 0 ? '#f44336' : '#666' }"
                >
                  {{ totalProfit }} TK
                </div>
              </div>
            </div>
          </div>
          
          <!-- Monthly Details Table -->
          <div v-if="printWithDetails" style="margin-top: 20px;">
            <div style="font-weight: bold; margin-bottom: 10px; color: #555">Daily Transactions:</div>
            
            <table style="width: 100%; border-collapse: collapse; font-family: 'Kalpurush', Arial, sans-serif;">
              <thead>
                <tr style="background-color: #f0f0f0;">
                  <th style="border: 1px solid #ddd; padding: 8px; text-align: center">তারিখ</th>
                  <th style="border: 1px solid #ddd; padding: 8px; text-align: center">ক্রয় (Kena)</th>
                  <th style="border: 1px solid #ddd; padding: 8px; text-align: center">বিক্রয় (Becha)</th>
                  <th style="border: 1px solid #ddd; padding: 8px; text-align: center">লাভ (Profit)</th>
                </tr>
              </thead>
              <tbody>
<tr 
  v-for="day in filteredDaysInMonth" 
  :key="day"
>
                  <td style="border: 1px solid #ddd; padding: 8px; text-align: center">
                    {{ day }} {{ currentMonthName }}
                    <span v-if="isToday(day)" style="color: #1976d2; font-weight: bold">(Today)</span>
                  </td>
                  <td style="border: 1px solid #ddd; padding: 8px; text-align: center; color: #d32f2f">
                    {{ getTransaction(day).kena || 0 }}
                  </td>
                  <td style="border: 1px solid #ddd; padding: 8px; text-align: center; color: #1976d2">
                    {{ getTransaction(day).becha || 0 }}
                  </td>
                  <td 
                    style="border: 1px solid #ddd; padding: 8px; text-align: center; font-weight: bold"
                    :style="{ color: getProfit(day) > 0 ? '#4caf50' : getProfit(day) < 0 ? '#f44336' : '#666' }"
                  >
                    {{ getProfit(day) }}
                  </td>
                </tr>
              </tbody>
            </table>
            
            <!-- Summary if enabled -->
            <div v-if="printWithSummary && printWithDetails" style="margin-top: 20px; padding: 15px; background-color: #e8f5e8; border-radius: 8px;">
              <div style="font-weight: bold; margin-bottom: 10px; color: #2e7d32">Monthly Summary:</div>
              <div style="display: flex; justify-content: space-between;">
                <div>
                  <div style="font-weight: bold; color: #666">মোট দিন:</div>
                  <div>{{ Object.keys(transactions).length }} দিন</div>
                </div>
                <div>
                  <div style="font-weight: bold; color: #666">লাভের দিন:</div>
                  <div style="color: #4caf50">{{ profitableDays }} দিন</div>
                </div>
                <div>
                  <div style="font-weight: bold; color: #666">ক্ষতির দিন:</div>
                  <div style="color: #f44336">{{ lossDays }} দিন</div>
                </div>
                <div>
                  <div style="font-weight: bold; color: #666">গড় লাভ:</div>
                  <div>{{ averageProfit }} TK/দিন</div>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Profit Analysis -->
          <div style="margin-top: 20px; padding: 15px; background-color: #e3f2fd; border-radius: 8px;">
            <div style="font-weight: bold; margin-bottom: 10px; color: #1976d2">Profit Analysis:</div>
            <div style="display: flex; align-items: center; justify-content: space-between;">
              <div style="flex: 1; text-align: center;">
                <div style="font-size: 2em; font-weight: bold; color: #1976d2">
                  <span v-if="totalProfit > 0">+</span>{{ totalProfit }} TK
                </div>
                <div style="color: #666; margin-top: 5px">Total Profit</div>
              </div>
              <div style="flex: 1; text-align: center;">
                <div style="font-size: 1.5em; font-weight: bold; color: #4caf50">{{ profitPercentage }}%</div>
                <div style="color: #666; margin-top: 5px">Profit Margin</div>
              </div>
              <div style="flex: 1; text-align: center;">
                <div 
                  style="font-size: 1.5em; font-weight: bold;"
                  :style="{ color: totalProfit > 0 ? '#4caf50' : totalProfit < 0 ? '#f44336' : '#666' }"
                >
                  <span v-if="totalProfit > 0">লাভ</span>
                  <span v-else-if="totalProfit < 0">ক্ষতি</span>
                  <span v-else>সমতা</span>
                </div>
                <div style="color: #666; margin-top: 5px">Status</div>
              </div>
            </div>
          </div>
          
          <div style="margin-top: 30px; text-align: center; color: #999; font-size: 0.9em">
            <div>Kena-Becha Profit Tracker - {{ currentMonthName }} {{ currentYear }}</div>
            <div>প্রিন্টের সময়: {{ currentTime }}</div>
            <div>মোট রেকর্ড: {{ Object.keys(transactions).length }} দিন</div>
          </div>
        </div>
      </div>
    </q-card-section>
  </q-card>
</q-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import GloHeader from './Glo-Header.vue'
const months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']

// Current date
const currentDate = new Date()
const currentMonth = ref(currentDate.getMonth())
const currentYear = ref(currentDate.getFullYear())
const todayDay = currentDate.getDate()

// Today's entry
const todayEntry = ref({
})

// All transactions
const transactions = ref({})

// Edit dialog
const editDialog = ref(false)
const editingDay = ref(0)
const editingData = ref({ kena: 0, becha: 0 })

// Computed
const currentMonthName = computed(() => {
  return months[currentMonth.value]
})

const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
})

const todayDate = computed(() => {
  return `${todayDay} ${months[currentDate.getMonth()]} ${currentDate.getFullYear()}`
})

const canSaveToday = computed(() => {
  return todayEntry.value.kena || todayEntry.value.becha
})

const editingProfit = computed(() => {
  const kena = parseFloat(editingData.value.kena) || 0
  const becha = parseFloat(editingData.value.becha) || 0
  return becha - kena
})

// Last 3 Days Data
const lastThreeDays = computed(() => {
  const days = []
  const today = new Date()
  
  for (let i = 1; i <= 3; i++) {
    const date = new Date(today)
    date.setDate(today.getDate() - i)
    
    const key = `${date.getFullYear()}-${date.getMonth()}-${date.getDate()}`
    const transaction = transactions.value[key]
    
    if (transaction) {
      days.unshift({
        date: `${date.getDate()} ${months[date.getMonth()].substring(0,3)}`,
        kena: transaction.kena || 0,
        becha: transaction.becha || 0,
        profit: transaction.profit || 0
      })
    }
  }
  
  return days
})

// Totals
const totalKena = computed(() => {
  return Object.values(transactions.value).reduce((sum, t) => sum + (parseFloat(t.kena) || 0), 0)
})

const totalBecha = computed(() => {
  return Object.values(transactions.value).reduce((sum, t) => sum + (parseFloat(t.becha) || 0), 0)
})

const totalProfit = computed(() => {
  return totalBecha.value - totalKena.value
})

// Functions
function calculateTodayProfit() {
  const kena = parseFloat(todayEntry.value.kena) || 0
  const becha = parseFloat(todayEntry.value.becha) || 0
  todayEntry.value.profit = becha - kena
}

function saveTodayEntry() {
  const key = getDateKey(todayDay)
  transactions.value[key] = {
    kena: parseFloat(todayEntry.value.kena) || 0,
    becha: parseFloat(todayEntry.value.becha) || 0,
    profit: todayEntry.value.profit
  }
  
  // Reset with default values
  todayEntry.value.kena = 400
  todayEntry.value.becha = 500
  calculateTodayProfit()
}

function getDateKey(day) {
  return `${currentYear.value}-${currentMonth.value}-${day}`
}

function isToday(day) {
  return day === todayDay && 
         currentMonth.value === currentDate.getMonth() && 
         currentYear.value === currentDate.getFullYear()
}

function hasTransaction(day) {
  const key = getDateKey(day)
  return transactions.value[key] !== undefined
}

function getTransaction(day) {
  const key = getDateKey(day)
  return transactions.value[key] || { kena: 0, becha: 0, profit: 0 }
}

function getProfit(day) {
  const transaction = getTransaction(day)
  return transaction.profit || 0
}

function editDay(day) {
  editingDay.value = day
  editingData.value = { ...getTransaction(day) }
  editDialog.value = true
}

function saveEditedDay() {
  const key = getDateKey(editingDay.value)
  transactions.value[key] = {
    kena: parseFloat(editingData.value.kena) || 0,
    becha: parseFloat(editingData.value.becha) || 0,
    profit: editingProfit.value
  }
}

function previousMonth() {
  if (currentMonth.value === 0) {
    currentMonth.value = 11
    currentYear.value--
  } else {
    currentMonth.value--
  }
}

function nextMonth() {
  if (currentMonth.value === 11) {
    currentMonth.value = 0
    currentYear.value++
  } else {
    currentMonth.value++
  }
}

// Storage
function saveToLocalStorage() {
  localStorage.setItem('kenaBechaSimple', JSON.stringify(transactions.value))
}

function loadFromLocalStorage() {
  const saved = localStorage.getItem('kenaBechaSimple')
  if (saved) {
    transactions.value = JSON.parse(saved)
  }
}

// Initialize
onMounted(() => {
  loadFromLocalStorage()
  calculateTodayProfit()
})

// Auto-save when transactions change
watch(transactions, () => {
  saveToLocalStorage()
}, { deep: true })

// Add these state variables
const showPrintPreview = ref(false)
const printType = ref('today') // 'today' or 'month'
const printMonthMode = ref(false)
const printWithDetails = ref(true)
const printWithSummary = ref(true)

// Add these computed properties
const profitableDays = computed(() => {
  return Object.values(transactions.value).filter(t => (t.profit || 0) > 0).length
})

const lossDays = computed(() => {
  return Object.values(transactions.value).filter(t => (t.profit || 0) < 0).length
})
// Add this computed property after daysInMonth computed property
const filteredDaysInMonth = computed(() => {
  const days = []
  for (let day = 1; day <= daysInMonth.value; day++) {
    if (hasTransaction(day)) {
      days.push(day)
    }
  }
  return days
})
const averageProfit = computed(() => {
  const totalDays = Object.keys(transactions.value).length
  if (totalDays === 0) return 0
  return Math.round(totalProfit.value / totalDays)
})

const profitPercentage = computed(() => {
  if (totalKena.value === 0) return 0
  return Math.round((totalProfit.value / totalKena.value) * 100)
})

const currentTime = computed(() => {
  const now = new Date()
  return `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`
})

// Add these methods
function printTodayReport() {
  printType.value = 'today'
  printMonthMode.value = false
  showPrintPreview.value = true
}

function printMonthReport() {
  printType.value = 'month'
  printMonthMode.value = true
  showPrintPreview.value = true
}

function executePrint() {
  const printContent = document.getElementById('printContent')
  
  // Create a new window for printing
  const printWindow = window.open('', '_blank')
  
  // Build the print document
  const doc = printWindow.document
  
  // Create the HTML structure
  doc.open()
  doc.write('<!DOCTYPE html>')
  doc.write('<html lang="bn">')
  doc.write('<head>')
  doc.write('<title>Kena-Becha Profit Report</title>')
  doc.write('<meta charset="UTF-8">')
  doc.write('<meta name="viewport" content="width=device-width, initial-scale=1.0">')
  doc.write('<style>')
  doc.write('@media print {')
  doc.write('  body { font-family: "Kalpurush", "Siyam Rupali", Arial, sans-serif; direction: ltr; }')
  doc.write('  table { width: 100%; border-collapse: collapse; font-size: 12px; }')
  doc.write('  th, td { border: 1px solid #ddd; padding: 6px; text-align: center; }')
  doc.write('  th { background-color: #f0f0f0; }')
  doc.write('  h2 { text-align: center; margin-bottom: 16px; }')
  doc.write('  .summary-box { background-color: #f5f5f5; padding: 15px; border-radius: 8px; margin: 15px 0; }')
  doc.write('  .footer { margin-top: 30px; text-align: center; color: #999; font-size: 0.9em; }')
  doc.write('}')
  doc.write('</style>')
  doc.write('</head>')
  doc.write('<body>')
  doc.write(printContent.innerHTML)
  
  // Add print script - Split to avoid Vue parsing issues
  doc.write('<scr' + 'ipt>')
  doc.write('window.onload = function() {')
  doc.write('  window.print();')
  doc.write('  setTimeout(function() {')
  doc.write('    window.close();')
  doc.write('  }, 1000);')
  doc.write('}')
  doc.write('</scr' + 'ipt>')
  
  doc.write('</body>')
  doc.write('</html>')
  doc.close()
}
</script>

<style scoped>
.table-container {
  max-height: 50vh;
  overflow-y: auto;
}

.table-row {
  min-height: 36px;
  align-items: center;
}

.border-bottom {
  border-bottom: 1px solid #e0e0e0;
}

/* Hide scrollbar for Chrome, Safari and Opera */
.table-container::-webkit-scrollbar {
  display: none;
}

/* Hide scrollbar for IE, Edge and Firefox */
.table-container {
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}
/* Add these print styles */
@media print {
  .print-content {
    font-family: 'Kalpurush', 'Siyam Rupali', Arial, sans-serif;
    direction: ltr;
  }
  
  .no-print {
    display: none !important;
  }
  
  .today-report-format .summary-box {
    page-break-inside: avoid;
  }
  
  .monthly-report-format table {
    page-break-inside: avoid;
  }
}

/* Add font imports if not already present */
@import url('https://fonts.maateen.me/kalpurush/font.css');

.print-content {
  font-family: 'Kalpurush', Arial, sans-serif;
}

/* Style adjustments for print preview */
.table-container {
  max-height: 40vh !important;
}
/* Mobile optimization */
@media (max-width: 600px) {
  .table-row {
    min-height: 32px;
    font-size: 0.85rem;
  }
  
  .col-3 {
    padding: 2px;
  }
  
  .text-h6 {
    font-size: 1.1rem;
  }
  
  .text-h5 {
    font-size: 1.2rem;
  }
}
</style>