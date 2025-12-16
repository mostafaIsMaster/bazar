<template>
  <div padding class="q-pa-sm">
    <Glo-Header></Glo-Header>
    <!-- ITEM SELECT WITH IMAGES -->
    <q-select
      v-model="selectedItem"
      :options="bazarItems"
      label="বাজার আইটেম নির্বাচন করুন"
      option-value="value"
      option-label="label"
      emit-value
      map-options
      filled
      dense
      class="q-mb-sm"
    >
      <template v-slot:option="scope">
        <q-item clickable v-bind="scope.itemProps" dense>
          <q-item-section avatar>
            <q-img
              :src="scope.opt.image"
              style="width: 35px; height: 35px; object-fit: cover"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>{{ scope.opt.label }}</q-item-label>
          </q-item-section>
        </q-item>
      </template>

      <template v-slot:selected-item="scope">
        <div class="row items-center">
          <q-img
            :src="scope.opt.image"
            style="width: 25px; height: 25px; object-fit: cover"
            class="q-mr-xs"
          />
          <span class="text-caption">{{ scope.opt.label }}</span>
        </div>
      </template>
    </q-select>

    <!-- PRICE INPUT -->
    <q-input
      v-model.number="price"
      type="number"
      label="মোট দাম (টাকা)"
      filled
      dense
      class="q-mb-sm"
    />

    <!-- QUANTITY SELECTOR ROW -->
    <div class="row q-mb-sm">
      <!-- REGULAR QUANTITY (KG) -->
      <div class="col-6 q-pr-xs">
        <q-select
          v-model="quantity"
          :options="quantityOptions"
          label="কেজি"
          filled
          dense
          options-dense
        />
      </div>
      
      <!-- MON & PIECES OPTIONS -->
      <div class="col-6 q-pl-xs">
        <div class="row q-col-gutter-xs">
          <!-- MON CHECKBOX -->
          <div class="col-6">
            <q-checkbox
              v-model="isMonSelected"
              color="primary"
              dense
              class="full-height flex flex-center bg-grey-2 q-pa-xs rounded-borders"
            >
              <template v-slot:default>
                <div class="column items-center q-pa-xs">
                  <div class="text-bold text-caption">মণ</div>
                  <div class="text-caption text-grey">৪০ কেজি</div>
                </div>
              </template>
            </q-checkbox>
          </div>
          
          <!-- PIECES CHECKBOX -->
          <div class="col-6">
            <q-checkbox
              v-model="isPiecesSelected"
              color="green"
              dense
              class="full-height flex flex-center bg-grey-2 q-pa-xs rounded-borders"
            >
              <template v-slot:default>
                <div class="column items-center q-pa-xs">
                  <div class="text-bold text-caption">পিস</div>
                  <div class="text-caption text-grey">টুকরা</div>
                </div>
              </template>
            </q-checkbox>
          </div>
        </div>
      </div>
    </div>

    <!-- PIECES QUANTITY INPUT (ONLY IF PIECES SELECTED) -->
    <div v-if="isPiecesSelected" class="row q-mb-sm">
      <!-- <div class="col-12">
        <q-input
          v-model.number="piecesQuantity"
          type="number"
          label="পিস সংখ্যা"
          filled
          dense
        />
      </div> -->
            <div class="col-12">
        <q-select
          v-model="piecesQuantity"
          :options="quantityOptions"
          label="পিস সংখ্যা"
          filled
          dense
          options-dense
        />
      </div>
    </div>

    <!-- BUTTONS ROW -->
    <div class="row q-col-gutter-xs q-mb-sm">
      <div class="col-5">
        <q-btn
          label="যোগ করুন"
          color="primary"
          class="full-width"
          @click="addItem"
          :disable="!selectedItem || !price"
          dense
        />
      </div>
      <div class="col-3">
        <q-btn
          label="ডিলিট"
          color="red"
          class="full-width"
          @click="toggleDeleteMode"
          :disable="selectedItems.length === 0"
          dense
        />
      </div>
      <div class="col-4">
        <q-btn
          label="প্রেডিকশন"
          color="orange"
          class="full-width"
          @click="viewPredictions = true"
          :disable="selectedItems.length === 0"
          dense
        />
      </div>
    </div>

    <!-- SAVE DATA BUTTON -->
    <q-btn
      label="সেভ ডাটা"
      color="green"
      class="full-width q-mb-sm"
      @click="viewSavedData = true"
      dense
    />
<!-- Add this after the buttons row -->

    <!-- SELECTION INFO -->
    <div v-if="isMonSelected" class="bg-blue-1 text-caption q-pa-xs q-mb-sm rounded-borders text-center">
      ১ মণ নির্বাচিত (৪০ কেজি)
    </div>
    
    <div v-if="isPiecesSelected && piecesQuantity > 0" class="bg-green-1 text-caption q-pa-xs q-mb-sm rounded-borders text-center">
      {{ piecesQuantity }} পিস নির্বাচিত
    </div>

    <!-- DELETE MODE INFO -->
    <q-banner v-if="deleteMode" dense class="bg-red-1 text-red q-mb-sm">
      <div class="row items-center justify-between">
        <div>ডিলিট মোড চালু আছে - আইটেম সিলেক্ট করুন</div>
        <q-btn label="বাতিল" color="red" size="sm" @click="cancelDeleteMode" dense />
      </div>
    </q-banner>

    <!-- CURRENT ITEMS - SIMPLE DELETE -->
    <div class="q-mb-sm">
      <div 
        v-for="(item, index) in selectedItems" 
        :key="index" 
        class="q-mb-xs"
      >
        <q-card flat bordered class="bg-grey-1">
          <q-card-section class="q-pa-xs">
            <div class="row items-center no-wrap">
              <!-- DELETE CHECKBOX (ONLY IN DELETE MODE) -->
              <div v-if="deleteMode" class="col-1">
                <q-checkbox
                  v-model="selectedForDelete"
                  :val="index"
                  color="red"
                  dense
                  size="sm"
                />
              </div>
              
              <!-- LEFT: IMAGE -->
              <div :class="deleteMode ? 'col-2' : 'col-2'">
                <q-img
                  :src="item.image"
                  style="width: 40px; height: 40px; object-fit: cover"
                  class="rounded-borders"
                />
              </div>
              
              <!-- MIDDLE: ITEM DETAILS -->
              <div :class="deleteMode ? 'col-6 q-pl-xs' : 'col-7 q-pl-sm'">
                <div class="row items-center no-wrap">
                  <div class="col-12">
                    <div class="row items-center no-wrap">
                      <div class="text-bold text-body2">{{ item.label }}</div>
                      <q-space />
                      <div class="text-caption text-grey">
                        <span v-if="item.isPieces">{{ item.quantity }} পিস</span>
                        <span v-else-if="item.isMon">১ মণ (৪০ কেজি)</span>
                        <span v-else>{{ item.quantity }} কেজি</span>
                      </div>
                    </div>
                    
                    <div class="row items-center no-wrap q-mt-xs">
                      <div class="text-caption text-grey">
                        <span v-if="item.isPieces">প্রতি পিস: {{ item.perUnit }} টাকা</span>
                        <span v-else>প্রতি কেজি: {{ item.perUnit }} টাকা</span>
                      </div>
                      <q-space />
                      <div class="text-bold text-body1">{{ item.totalPrice }} টাকা</div>
                    </div>
                    
                    <!-- TYPE BADGE -->
                    <div class="text-caption q-mt-xs">
                      <q-badge 
                        :color="item.isMon ? 'blue' : item.isPieces ? 'green' : 'grey'" 
                        text-color="white" 
                        class="q-pa-xs"
                      >
                        <span v-if="item.isMon">মণে কেনা</span>
                        <span v-else-if="item.isPieces">পিস</span>
                        <span v-else>কেজি</span>
                      </q-badge>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- RIGHT: TIME & DELETE BUTTON -->
              <div :class="deleteMode ? 'col-3 text-right' : 'col-3 text-right'">
                <div v-if="!deleteMode" class="text-caption" :class="getTimeColor(item.time)">
                  {{ formatTimeAgo(item.time) }}
                </div>
                
                <!-- DELETE BUTTON (ALWAYS VISIBLE) -->
                <q-btn 
                  round 
                  dense 
                  flat 
                  color="red" 
                  icon="delete" 
                  size="sm"
                  @click="deleteItem(index)"
                  class="q-mt-xs"
                />
              </div>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>

    <!-- DELETE SELECTED BUTTON (ONLY IN DELETE MODE) -->
    <div v-if="deleteMode && selectedForDelete.length > 0" class="row q-col-gutter-xs q-mb-sm">
      <div class="col-12">
        <q-btn
          label="সিলেক্টেড ডিলিট করুন"
          color="red"
          class="full-width"
          @click="deleteSelectedItems"
          dense
          icon="delete"
        />
      </div>
    </div>

    <!-- SELECTED COUNT IN DELETE MODE -->
    <div v-if="deleteMode && selectedForDelete.length > 0" class="text-center text-red text-bold q-mb-sm">
      {{ selectedForDelete.length }} টি আইটেম সিলেক্টেড
    </div>

    <!-- TOTAL SUMMARY -->
    <q-card class="bg-grey-2 q-pa-sm">
      <div class="column q-gutter-xs">
        <div class="row justify-between">
          <div class="text-caption">মোট আইটেম:</div>
          <div class="text-bold">{{ selectedItems.length }} টি</div>
        </div>
        
        <div class="row justify-between" v-if="totalMonItems > 0">
          <div class="text-caption">মণে কেনা:</div>
          <div class="text-blue">{{ totalMonItems }} টি</div>
        </div>
        
        <div class="row justify-between" v-if="totalPiecesItems > 0">
          <div class="text-caption">পিস আইটেম:</div>
          <div class="text-green">{{ totalPiecesItems }} টি</div>
        </div>
        
        <div class="row justify-between">
          <div class="text-caption">সর্বমোট টাকা:</div>
          <div class="text-bold text-h6">{{ totalPrice }} টাকা</div>
        </div>
      </div>
    </q-card>

    <!-- PREDICTIONS DIALOG -->
    <q-dialog v-model="viewPredictions" maximized>
      <q-card>
        <q-toolbar class="bg-primary text-white">
          <q-toolbar-title class="text-caption">লাভ প্রেডিকশন</q-toolbar-title>
          <q-btn flat round dense icon="close" v-close-popup />
        </q-toolbar>
        
        <q-card-section class="q-pa-sm">
          <div v-if="selectedItems.length === 0" class="text-center q-pa-xl">
            <q-icon name="info" size="xl" class="q-mb-md" />
            <div>কোন আইটেম নেই প্রেডিকশন দেখানোর জন্য</div>
          </div>
          
          <div v-else>
            <!-- TOTAL SUMMARY -->
            <div class="row q-mb-md">
              <div class="col-12">
                <q-card class="bg-blue-1">
                  <q-card-section class="q-pa-sm">
                    <div class="text-bold text-center q-mb-xs">সব আইটেমের সামারি</div>
                    <div class="column q-gutter-xs">
                      <div class="row justify-between">
                        <div class="text-caption">মোট ক্রয় মূল্য:</div>
                        <div class="text-bold text-h6">{{ totalPrice }} টাকা</div>
                      </div>
                      <div class="row justify-between" v-if="totalKg > 0">
                        <div class="text-caption">মোট কেজি:</div>
                        <div class="text-bold">{{ totalKg }} কেজি</div>
                      </div>
                      <div class="row justify-between" v-if="totalPieces > 0">
                        <div class="text-caption">মোট পিস:</div>
                        <div class="text-bold">{{ totalPieces }} পিস</div>
                      </div>
                    </div>
                  </q-card-section>
                </q-card>
              </div>
            </div>
            
            <!-- PROFIT METHOD SELECTION -->
            <div class="row q-mb-md">
              <div class="col-12">
                <div class="text-bold q-mb-xs">লাভ হিসাবের পদ্ধতি:</div>
                <q-btn-toggle
                  v-model="selectedProfitMethod"
                  spread
                  :options="profitMethods"
                  class="q-mb-sm"
                  dense
                />
              </div>
            </div>
            
            <!-- PROFIT TARGET SELECTOR -->
            <div v-if="selectedProfitMethod === 'target'" class="row q-mb-md">
              <div class="col-12">
                <div class="text-bold q-mb-xs">লাভ টার্গেট সিলেক্ট করুন:</div>
                <div class="row q-col-gutter-xs">
                  <div class="col-4" v-for="profit in profitTargets" :key="profit">
                    <q-btn
                      :label="profit + ' টাকা'"
                      :color="selectedProfitTarget === profit ? 'primary' : 'grey'"
                      class="full-width"
                      dense
                      @click="selectedProfitTarget = profit"
                    />
                  </div>
                </div>
                
                <!-- CUSTOM PROFIT INPUT -->
                <div class="row q-mb-md">
                  <div class="col-12">
                    <q-input
                      v-model.number="customProfitTarget"
                      type="number"
                      label="কাস্টম লাভ টার্গেট (টাকা)"
                      filled
                      dense
                      class="q-mb-xs"
                    />
                    <q-btn
                      label="কাস্টম লাভ এপ্লাই করুন"
                      color="primary"
                      class="full-width"
                      dense
                      @click="applyCustomProfit"
                    />
                  </div>
                </div>
              </div>
            </div>
            
            <!-- UNIT PROFIT SETTINGS -->
            <div v-if="selectedProfitMethod === 'unit'" class="row q-mb-md">
              <div class="col-12">
                <div class="text-bold q-mb-xs">প্রতি ইউনিট লাভ:</div>
                <div class="row q-col-gutter-xs">
                  <div class="col-6">
                    <q-input
                      v-model.number="unitProfitKg"
                      type="number"
                      label="প্রতি কেজি লাভ (টাকা)"
                      filled
                      dense
                      @update:model-value="updateUnitProfit"
                    />
                  </div>
                  <div class="col-6">
                    <q-input
                      v-model.number="unitProfitPiece"
                      type="number"
                      label="প্রতি পিস লাভ (টাকা)"
                      filled
                      dense
                      @update:model-value="updateUnitProfit"
                    />
                  </div>
                </div>
                <div class="text-caption text-grey q-mt-xs">
                  প্রতিটি কেজি আইটেমে {{ unitProfitKg }} টাকা এবং প্রতিটি পিসে {{ unitProfitPiece }} টাকা লাভ
                </div>
              </div>
            </div>
            
            <!-- PERCENTAGE PROFIT SETTINGS -->
            <div v-if="selectedProfitMethod === 'percentage'" class="row q-mb-md">
              <div class="col-12">
                <div class="text-bold q-mb-xs">শতকরা লাভ (%):</div>
                <div class="row q-col-gutter-xs">
                  <div class="col-4" v-for="percentage in percentageOptions" :key="percentage">
                    <q-btn
                      :label="percentage + '%'"
                      :color="selectedPercentage === percentage ? 'primary' : 'grey'"
                      class="full-width"
                      dense
                      @click="selectedPercentage = percentage"
                    />
                  </div>
                </div>
                
                <!-- CUSTOM PERCENTAGE INPUT -->
                <div class="row q-mb-md">
                  <div class="col-12">
                    <q-input
                      v-model.number="customPercentage"
                      type="number"
                      label="কাস্টম শতকরা লাভ (%)"
                      filled
                      dense
                      class="q-mb-xs"
                    />
                    <q-btn
                      label="কাস্টম % এপ্লাই করুন"
                      color="primary"
                      class="full-width"
                      dense
                      @click="applyCustomPercentage"
                    />
                  </div>
                </div>
              </div>
            </div>
            
            <!-- TOTAL PROFIT SUMMARY -->
            <div class="row q-mb-md">
              <div class="col-12">
                <q-card class="bg-green-1">
                  <q-card-section class="q-pa-sm">
                    <div class="text-bold text-center q-mb-xs">মোট লাভের হিসাব</div>
                    <div class="column q-gutter-xs">
                      <div class="row justify-between">
                        <div class="text-caption">মোট ক্রয় মূল্য:</div>
                        <div class="text-bold">{{ totalPrice }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">মোট লাভ:</div>
                        <div class="text-bold text-green">{{ totalProfit }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">মোট বিক্রয় মূল্য:</div>
                        <div class="text-bold text-blue">{{ totalSellingPrice }} টাকা</div>
                      </div>
                    </div>
                  </q-card-section>
                </q-card>
              </div>
            </div>
            
            <!-- PROFIT DETAILS PER METHOD -->
            <div v-if="selectedProfitMethod === 'target'" class="row q-mb-md">
              <div class="col-12">
                <q-card class="bg-grey-1 q-pa-sm">
                  <div class="text-caption text-center">
                    <span v-if="selectedProfitTarget > 0">
                      {{ selectedProfitTarget }} টাকা লাভের জন্য, প্রতিটি আইটেমের লাভ ক্রয় মূল্যের অনুপাতে ভাগ করা হবে
                    </span>
                    <span v-else>
                      লাভ টার্গেট সিলেক্ট করুন
                    </span>
                  </div>
                </q-card>
              </div>
            </div>
            
            <div v-if="selectedProfitMethod === 'unit'" class="row q-mb-md">
              <div class="col-12">
                <q-card class="bg-grey-1 q-pa-sm">
                  <div class="text-caption text-center">
                    <span v-if="unitProfitKg > 0 || unitProfitPiece > 0">
                      প্রতি কেজিতে {{ unitProfitKg }} টাকা লাভ, প্রতি পিসে {{ unitProfitPiece }} টাকা লাভ
                    </span>
                    <span v-else>
                      প্রতি ইউনিট লাভ নির্ধারণ করুন
                    </span>
                  </div>
                </q-card>
              </div>
            </div>
            
            <div v-if="selectedProfitMethod === 'percentage'" class="row q-mb-md">
              <div class="col-12">
                <q-card class="bg-grey-1 q-pa-sm">
                  <div class="text-caption text-center">
                    <span v-if="selectedPercentage > 0">
                      প্রতিটি আইটেমে ক্রয় মূল্যের {{ selectedPercentage }}% লাভ
                    </span>
                    <span v-else>
                      শতকরা লাভ নির্ধারণ করুন
                    </span>
                  </div>
                </q-card>
              </div>
            </div>
            
            <!-- COLLAPSIBLE ITEM DETAILS -->
            <div class="row q-mb-sm">
              <div class="col-12">
                <q-btn
                  label="সব আইটেম দেখান/লুকান"
                  color="blue"
                  class="full-width"
                  dense
                  @click="toggleAllItems"
                  :icon="showAllItems ? 'expand_less' : 'expand_more'"
                />
              </div>
            </div>
            
            <!-- INDIVIDUAL ITEM PREDICTIONS (COLLAPSIBLE) -->
            <div v-for="(item, index) in selectedItems" :key="index" class="q-mb-sm">
              <q-expansion-item
                :label="getItemLabel(item)"
                :caption="getItemCaption(item)"
                expand-separator
                :default-opened="false"
                class="bg-grey-1"
              >
                <q-card>
                  <q-card-section class="q-pa-sm">
                    <div class="column q-gutter-sm">
                      <!-- CURRENT INFO -->
                      <div class="row justify-between">
                        <div class="text-caption">ক্রয় মূল্য:</div>
                        <div class="text-bold">{{ item.totalPrice }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">
                          <span v-if="item.isPieces">প্রতি পিস ক্রয়:</span>
                          <span v-else>প্রতি কেজি ক্রয়:</span>
                        </div>
                        <div class="text-bold">{{ item.perUnit }} টাকা</div>
                      </div>
                      
                      <!-- PREDICTION FOR SELECTED METHOD -->
                      <div class="row justify-between">
                        <div class="text-caption">মোট লাভ:</div>
                        <div class="text-bold text-green">{{ calculateItemProfit(item) }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">
                          <span v-if="item.isPieces">প্রতি পিস লাভ:</span>
                          <span v-else>প্রতি কেজি লাভ:</span>
                        </div>
                        <div class="text-bold">{{ calculateProfitPerUnit(item) }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">
                          <span v-if="item.isPieces">বিক্রয় মূল্য (প্রতি পিস):</span>
                          <span v-else>বিক্রয় মূল্য (প্রতি কেজি):</span>
                        </div>
                        <div class="text-bold text-blue">{{ calculateSellingPricePerUnit(item) }} টাকা</div>
                      </div>
                      <div class="row justify-between">
                        <div class="text-caption">মোট বিক্রয় মূল্য:</div>
                        <div class="text-bold">{{ calculateTotalSelling(item) }} টাকা</div>
                      </div>
                    </div>
                  </q-card-section>
                </q-card>
              </q-expansion-item>
            </div>

          </div>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- SAVED DATA DIALOG -->
    <q-dialog v-model="viewSavedData" maximized>
      <q-card>
        <q-toolbar class="bg-primary text-white">
          <q-toolbar-title class="text-caption">সেভ করা ডাটা</q-toolbar-title>
          <q-btn flat round dense icon="close" v-close-popup />
        </q-toolbar>
        
        <q-card-section class="q-pa-sm">
          <div v-if="savedItems.length === 0" class="text-center q-pa-xl">
            <q-icon name="info" size="xl" class="q-mb-md" />
            <div>কোন সেভ করা ডাটা নেই</div>
          </div>
          
          <div v-else>
            <!-- DELETE ALL BUTTON -->
            <div class="row q-mb-sm">
              <q-btn
                label="সব ডাটা ডিলিট করুন"
                color="red"
                @click="deleteAllSavedData"
                class="full-width"
                dense
              />
            </div>

            <!-- GROUP BY 5 MINUTE INTERVALS -->
            <div v-for="(group, groupIndex) in groupedSavedItems" :key="groupIndex" class="q-mb-md">
              <!-- GROUP HEADER -->
              <div class="text-bold text-caption q-mb-sm bg-blue-1 q-pa-xs rounded-borders">
                {{ formatGroupTime(group.time) }}
              </div>
              
              <!-- GROUP ITEMS -->
              <div>
                <div 
                  v-for="(item, itemIndex) in group.items" 
                  :key="getSavedItemKey(item, itemIndex)" 
                  class="q-mb-xs"
                >
                  <q-card flat bordered class="bg-grey-1">
                    <q-card-section class="q-pa-xs">
                      <div class="row items-center no-wrap">
                        <!-- IMAGE -->
                        <div class="col-3">
                          <q-img
                            :src="item.image"
                            style="width: 35px; height: 35px; object-fit: cover"
                            class="rounded-borders"
                          />
                        </div>
                        
                        <!-- DETAILS -->
                        <div class="col-8 q-pl-sm">
                          <div class="row items-center no-wrap">
                            <div class="text-bold text-body2">{{ item.label }}</div>
                            <q-space />
                            <div class="text-caption text-grey">
                              <span v-if="item.isPieces">{{ item.quantity }} পিস</span>
                              <span v-else-if="item.isMon">১ মণ (৪০ কেজি)</span>
                              <span v-else>{{ item.quantity }} কেজি</span>
                            </div>
                          </div>
                          
                          <div class="row items-center no-wrap q-mt-xs">
                            <div class="text-caption text-grey">
                              <span v-if="item.isPieces">প্রতি পিস: {{ item.perUnit }} টাকা</span>
                              <span v-else>প্রতি কেজি: {{ item.perUnit }} টাকা</span>
                            </div>
                            <q-space />
                            <div class="text-bold">{{ item.totalPrice }} টাকা</div>
                          </div>
                          
                          <!-- TYPE BADGE -->
                          <div class="text-caption q-mt-xs">
                            <q-badge 
                              :color="item.isMon ? 'blue' : item.isPieces ? 'green' : 'grey'" 
                              text-color="white" 
                              class="q-pa-xs"
                            >
                              <span v-if="item.isMon">মণে কেনা</span>
                              <span v-else-if="item.isPieces">পিস</span>
                              <span v-else>কেজি</span>
                            </q-badge>
                          </div>
                        </div>
                        
                        <!-- TIME -->
                        <div class="col-1 text-right">
                          <div class="text-caption text-grey">
                            {{ formatTimeOnly(item.time) }}
                          </div>
                        </div>
                      </div>
                    </q-card-section>
                  </q-card>
                </div>
              </div>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
    <div class="row q-col-gutter-xs q-mb-sm">
  <div class="col-6 q-pr-xs">
    <q-btn
      label="প্রিন্ট (সরকারি)"
      color="primary"
      class="full-width"
      @click="printOfficialFormat"
      :disable="selectedItems.length === 0"
      dense
      icon="print"
    />
  </div>
  <div class="col-6 q-pl-xs">
    <q-btn
      label="প্রিন্ট (সাধারণ)"
      color="blue-grey"
      class="full-width"
      @click="printSimpleFormat"
      :disable="selectedItems.length === 0"
      dense
      icon="description"
    />
  </div>
</div>
<!-- PRINT PREVIEW DIALOG -->
<q-dialog v-model="showPrintPreview" maximized>
  <q-card>
    <q-toolbar class="bg-primary text-white">
      <q-toolbar-title class="text-caption">প্রিন্ট প্রিভিউ</q-toolbar-title>
      <q-btn flat round dense icon="close" v-close-popup />
    </q-toolbar>
    
    <q-card-section class="q-pa-sm">
      <!-- Print Controls -->
      <div class="row q-mb-md q-pa-sm bg-grey-3 rounded-borders">
        <div class="col-12">
          <div class="row items-center justify-between">
            <div class="text-bold">প্রিন্ট সেটিংস</div>
            <q-btn 
              label="প্রিন্ট করুন" 
              color="primary" 
              @click="executePrint" 
              icon="print"
              dense
            />
          </div>
          <div class="row q-mt-sm">
            <div class="col-6">
              <q-select
                v-model="printFormat"
                :options="printFormats"
                label="প্রিন্ট ফরম্যাট"
                filled
                dense
              />
            </div>
            <div class="col-6">
              <q-checkbox
                v-model="printWithDate"
                label="তারিখ যোগ করুন"
                dense
              />
            </div>
          </div>
        </div>
      </div>
      
      <!-- Print Preview Content -->
      <div id="printContent" class="print-content">
        <!-- Official Format -->
        <div v-if="printFormat === 'official'" class="official-format">
          <div class="text-center q-mb-md">
            <h3 style="margin: 0">বাজার খরচ বিবরণী</h3>
            <div v-if="printWithDate" style="border-bottom: 1px solid #ccc; padding-bottom: 8px; margin-bottom: 16px">
              তারিখ: {{ currentDate }}
            </div>
          </div>
          
          <table style="width: 100%; border-collapse: collapse; font-family: 'Siyam Rupali', 'SolaimanLipi', Arial, sans-serif;">
            <thead>
              <tr style="background-color: #f0f0f0;">
                <th style="border: 1px solid #000; padding: 8px; text-align: center; width: 40px">ক্র. নং</th>
                <th style="border: 1px solid #000; padding: 8px; text-align: left">আইটেমের নাম</th>
                <th style="border: 1px solid #000; padding: 8px; text-align: center">পরিমাণ</th>
                <th style="border: 1px solid #000; padding: 8px; text-align: center">একক মূল্য</th>
                <th style="border: 1px solid #000; padding: 8px; text-align: center">মোট মূল্য</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in selectedItems" :key="index">
                <td style="border: 1px solid #000; padding: 8px; text-align: center">{{ index + 1 }}</td>
                <td style="border: 1px solid #000; padding: 8px">{{ item.label }}</td>
                <td style="border: 1px solid #000; padding: 8px; text-align: center">
                  <span v-if="item.isPieces">{{ item.quantity }} পিস</span>
                  <span v-else-if="item.isMon">১ মণ (৪০ কেজি)</span>
                  <span v-else>{{ item.quantity }} কেজি</span>
                </td>
                <td style="border: 1px solid #000; padding: 8px; text-align: center">{{ item.perUnit }} টাকা</td>
                <td style="border: 1px solid #000; padding: 8px; text-align: center; font-weight: bold">{{ item.totalPrice }} টাকা</td>
              </tr>
            </tbody>
            <tfoot>
              <tr style="background-color: #f0f0f0; font-weight: bold;">
                <td colspan="3" style="border: 1px solid #000; padding: 8px; text-align: right">সর্বমোট:</td>
                <td style="border: 1px solid #000; padding: 8px; text-align: center">-</td>
                <td style="border: 1px solid #000; padding: 8px; text-align: center; font-size: 1.1em">{{ totalPrice }} টাকা</td>
              </tr>
            </tfoot>
          </table>
          
          <div class="q-mt-lg" style="font-family: 'Siyam Rupali', 'SolaimanLipi', Arial, sans-serif;">
            <div style="border-top: 2px dashed #000; padding-top: 16px; margin-top: 16px">
              <div class="row">
                <div class="col-6">
                  <div style="margin-top: 40px">
                    <div style="border-top: 1px solid #000; width: 80%; padding-top: 4px">
                      প্রস্তুতকারকের স্বাক্ষর
                    </div>
                  </div>
                </div>
                <div class="col-6 text-right">
                  <div style="margin-top: 40px">
                    <div style="border-top: 1px solid #000; width: 80%; margin-left: auto; padding-top: 4px">
                      প্রাপকের স্বাক্ষর
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- Simple Format -->
        <div v-else class="simple-format">
          <div class="text-center q-mb-md">
            <h2 style="margin: 0; color: #1976d2">বাজার হিসাব</h2>
            <div v-if="printWithDate" style="color: #666; margin-top: 8px">
              তারিখ: {{ currentDate }}
            </div>
          </div>
          
          <div v-for="(item, index) in selectedItems" :key="index" style="margin-bottom: 12px; padding-bottom: 12px; border-bottom: 1px dashed #ddd;">
            <div style="display: flex; justify-content: space-between; align-items: center;">
              <div>
                <span style="font-weight: bold; font-size: 1.1em">{{ item.label }}</span>
                <span style="margin-left: 12px; background-color: #e3f2fd; padding: 2px 8px; border-radius: 12px; font-size: 0.9em">
                  <span v-if="item.isPieces">{{ item.quantity }} পিস</span>
                  <span v-else-if="item.isMon">১ মণ</span>
                  <span v-else>{{ item.quantity }} কেজি</span>
                </span>
              </div>
              <div style="font-weight: bold; font-size: 1.1em">{{ item.totalPrice }} টাকা</div>
            </div>
            <div style="color: #666; margin-top: 4px; font-size: 0.9em">
              প্রতি ইউনিট: {{ item.perUnit }} টাকা
              <span style="margin-left: 12px">
                <q-badge :color="item.isMon ? 'blue' : item.isPieces ? 'green' : 'grey'" style="font-size: 0.8em">
                  <span v-if="item.isMon">মণ</span>
                  <span v-else-if="item.isPieces">পিস</span>
                  <span v-else>কেজি</span>
                </q-badge>
              </span>
            </div>
          </div>
          
          <div style="margin-top: 24px; padding-top: 16px; border-top: 2px solid #1976d2">
            <div style="display: flex; justify-content: space-between; font-weight: bold; font-size: 1.2em">
              <span>সর্বমোট:</span>
              <span>{{ totalPrice }} টাকা</span>
            </div>
            <div style="display: flex; justify-content: space-between; margin-top: 8px; color: #666">
              <span>মোট আইটেম:</span>
              <span>{{ selectedItems.length }} টি</span>
            </div>
            <div v-if="totalMonItems > 0" style="display: flex; justify-content: space-between; margin-top: 4px; color: #1976d2">
              <span>মণে কেনা:</span>
              <span>{{ totalMonItems }} টি</span>
            </div>
            <div v-if="totalPiecesItems > 0" style="display: flex; justify-content: space-between; margin-top: 4px; color: #4caf50">
              <span>পিস আইটেম:</span>
              <span>{{ totalPiecesItems }} টি</span>
            </div>
          </div>
          
          <div style="margin-top: 32px; text-align: center; color: #999; font-size: 0.9em">
            <div>Bazar Calculator App - Developed by Abdul Kader</div>
            <div>প্রিন্টের সময়: {{ currentTime }}</div>
          </div>
        </div>
      </div>
    </q-card-section>
  </q-card>
</q-dialog>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import GloHeader from './Glo-Header.vue'

import { 
  QSelect, QInput, QBtn, QImg, QCard, QCardSection, 
  QCheckbox, QDialog, QToolbar, QToolbarTitle, QBtnToggle,
  QIcon, date, QSpace, QBadge, QBanner, QExpansionItem
} from 'quasar'

// IMPORT IMAGES
import fulkopiImg from 'src/assets/fullcofi.png'
import peyajImg from 'src/assets/পেঁয়াজ.png'

// ITEMS LIST
const bazarItems = [
  { label: 'ফুলকপি', value: 'fulkopi', image: fulkopiImg },
  { label: 'পেঁয়াজ', value: 'peyaj', image: peyajImg }
]

// QUANTITY OPTIONS (1-20 kg)
const quantityOptions = Array.from({ length: 20 }, (_, i) => i + 1)

// STATE
const selectedItem = ref(null)
const price = ref(null)
const quantity = ref(5)
const piecesQuantity = ref(1)
const isMonSelected = ref(false)
const isPiecesSelected = ref(false)
const selectedItems = ref([])
const savedItems = ref([])
const viewSavedData = ref(false)
const viewPredictions = ref(false)
const deleteMode = ref(false)
const selectedForDelete = ref([])

// PROFIT METHOD STATE
const profitMethods = [
  { label: 'লাভ টার্গেট', value: 'target' },
  { label: 'প্রতি ইউনিট সমান লাভ', value: 'unit' },
  { label: '% লাভ', value: 'percentage' }
]
const selectedProfitMethod = ref('target') // Default method

// PROFIT TARGET STATE (Method 1)
const profitTargets = [100, 300, 500, 800, 1000]
const selectedProfitTarget = ref(500)
const customProfitTarget = ref(null)

// UNIT PROFIT STATE (Method 2)
const unitProfitKg = ref(10) // Default 10 taka per kg
const unitProfitPiece = ref(2) // Default 2 taka per piece

// PERCENTAGE PROFIT STATE (Method 3)
const percentageOptions = [10, 15, 20, 25, 30]
const selectedPercentage = ref(20)
const customPercentage = ref(null)

// UI STATE
const showAllItems = ref(false)

// LOAD SAVED DATA ON MOUNT
onMounted(() => {
  const saved = localStorage.getItem('bazarData')
  if (saved) {
    savedItems.value = JSON.parse(saved)
    selectedItems.value = [...savedItems.value.slice(0, 10)]
  }
})

// WATCH FOR CHECKBOX CHANGES
watch(isPiecesSelected, (newVal) => {
  if (newVal) {
    isMonSelected.value = false
  }
})

watch(isMonSelected, (newVal) => {
  if (newVal) {
    isPiecesSelected.value = false
    quantity.value = 40
  } else if (!isPiecesSelected.value) {
    quantity.value = 5
  }
})

// CREATE UNIQUE KEY FOR SAVED ITEMS
function getSavedItemKey(item, index) {
  return `${item.time}-${item.label}-${index}`
}

// ADD ITEM FUNCTION
function addItem () {
  if (!selectedItem.value || !price.value) return

  const item = bazarItems.find(i => i.value === selectedItem.value)
  const currentTime = new Date()
  
  let actualQuantity, perUnit
  
  if (isPiecesSelected.value) {
    actualQuantity = piecesQuantity.value || 1
    perUnit = (Number(price.value) / actualQuantity).toFixed(2)
  } else if (isMonSelected.value) {
    actualQuantity = 40
    perUnit = (Number(price.value) / actualQuantity).toFixed(2)
  } else {
    actualQuantity = quantity.value
    perUnit = (Number(price.value) / actualQuantity).toFixed(2)
  }

  const newItem = {
    label: item.label,
    image: item.image,
    quantity: actualQuantity,
    isMon: isMonSelected.value,
    isPieces: isPiecesSelected.value,
    totalPrice: Number(price.value),
    perUnit: perUnit,
    time: currentTime.toISOString()
  }

  selectedItems.value.unshift(newItem)
  savedItems.value.unshift(newItem)
  localStorage.setItem('bazarData', JSON.stringify(savedItems.value))

  // RESET INPUTS
  price.value = null
  isMonSelected.value = false
  isPiecesSelected.value = false
  quantity.value = 5
  piecesQuantity.value = 1
  selectedItem.value = null
}

// TOGGLE DELETE MODE
function toggleDeleteMode() {
  deleteMode.value = !deleteMode.value
  if (!deleteMode.value) {
    selectedForDelete.value = []
  }
}

// CANCEL DELETE MODE
function cancelDeleteMode() {
  deleteMode.value = false
  selectedForDelete.value = []
}

// DELETE SINGLE ITEM
function deleteItem(index) {
  const itemToDelete = selectedItems.value[index]
  selectedItems.value.splice(index, 1)
  
  const savedIndex = savedItems.value.findIndex(item => 
    item.time === itemToDelete.time && 
    item.label === itemToDelete.label &&
    item.totalPrice === itemToDelete.totalPrice
  )
  
  if (savedIndex !== -1) {
    savedItems.value.splice(savedIndex, 1)
    localStorage.setItem('bazarData', JSON.stringify(savedItems.value))
  }
}

// DELETE SELECTED ITEMS (BULK DELETE)
function deleteSelectedItems() {
  if (selectedForDelete.value.length === 0) return
  
  const sortedIndices = [...selectedForDelete.value].sort((a, b) => b - a)
  const deletedItems = []
  
  sortedIndices.forEach(index => {
    if (selectedItems.value[index]) {
      deletedItems.push(selectedItems.value[index])
    }
  })
  
  sortedIndices.forEach(index => {
    selectedItems.value.splice(index, 1)
  })
  
  deletedItems.forEach(itemToDelete => {
    const savedIndex = savedItems.value.findIndex(item => 
      item.time === itemToDelete.time && 
      item.label === itemToDelete.label &&
      item.totalPrice === itemToDelete.totalPrice
    )
    
    if (savedIndex !== -1) {
      savedItems.value.splice(savedIndex, 1)
    }
  })
  
  localStorage.setItem('bazarData', JSON.stringify(savedItems.value))
  deleteMode.value = false
  selectedForDelete.value = []
}

// DELETE ALL SAVED DATA
function deleteAllSavedData() {
  savedItems.value = []
  selectedItems.value = []
  localStorage.removeItem('bazarData')
  viewSavedData.value = false
}

// PROFIT CALCULATION METHODS

// Method 1: Target Profit (proportional to item value)
function calculateItemProfitTarget(item) {
  if (totalPrice.value === 0 || selectedProfitTarget.value <= 0) return 0
  const itemPercentage = item.totalPrice / totalPrice.value
  const totalProfit = selectedProfitTarget.value * itemPercentage
  return parseFloat(totalProfit.toFixed(2))
}

// Method 2: Unit Profit
function calculateItemProfitUnit(item) {
  if (item.isPieces) {
    return unitProfitPiece.value * item.quantity
  } else {
    return unitProfitKg.value * item.quantity
  }
}

// Method 3: Percentage Profit
function calculateItemProfitPercentage(item) {
  return (item.totalPrice * selectedPercentage.value) / 100
}

// Main profit calculation function
function calculateItemProfit(item) {
  switch (selectedProfitMethod.value) {
    case 'target':
      return calculateItemProfitTarget(item)
    case 'unit':
      return calculateItemProfitUnit(item)
    case 'percentage':
      return calculateItemProfitPercentage(item)
    default:
      return 0
  }
}

// Calculate profit per unit (kg or piece)
function calculateProfitPerUnit(item) {
  const itemProfit = calculateItemProfit(item)
  return (itemProfit / item.quantity).toFixed(2)
}

// Calculate selling price per unit
function calculateSellingPricePerUnit(item) {
  const profitPerUnit = parseFloat(calculateProfitPerUnit(item))
  const sellingPrice = parseFloat(item.perUnit) + profitPerUnit
  return sellingPrice.toFixed(2)
}

// Calculate total selling price
function calculateTotalSelling(item) {
  const sellingPricePerUnit = parseFloat(calculateSellingPricePerUnit(item))
  return (sellingPricePerUnit * item.quantity).toFixed(2)
}

// Get item label for expansion item
function getItemLabel(item) {
  if (item.isPieces) {
    return `${item.label} - ${item.quantity} পিস (ক্রয়: ${item.totalPrice} টাকা)`
  } else if (item.isMon) {
    return `${item.label} - ১ মণ (৪০ কেজি) (ক্রয়: ${item.totalPrice} টাকা)`
  } else {
    return `${item.label} - ${item.quantity} কেজি (ক্রয়: ${item.totalPrice} টাকা)`
  }
}

// Get item caption for expansion item
function getItemCaption(item) {
  const sellingPrice = calculateSellingPricePerUnit(item)
  if (item.isPieces) {
    return `প্রতি কেজি ক্রয়: ${item.perUnit} টাকা | বিক্রয় করবেন: প্রতি পিস ${sellingPrice} টাকা`
  } else {
    return `প্রতি কেজি ক্রয়: ${item.perUnit} টাকা | বিক্রয় করবেন: প্রতি কেজি ${sellingPrice} টাকা`
  }
}

// Update unit profit when kg or piece value changes
function updateUnitProfit() {
  // This ensures UI updates when unit profit values change
}

// Apply custom profit
function applyCustomProfit() {
  if (customProfitTarget.value && customProfitTarget.value > 0) {
    selectedProfitTarget.value = customProfitTarget.value
    customProfitTarget.value = null
  }
}

// Apply custom percentage
function applyCustomPercentage() {
  if (customPercentage.value && customPercentage.value > 0) {
    selectedPercentage.value = customPercentage.value
    customPercentage.value = null
  }
}

// Toggle all items
function toggleAllItems() {
  showAllItems.value = !showAllItems.value
}

// Calculate total profit
const totalProfit = computed(() => {
  if (selectedItems.value.length === 0) return 0
  
  let total = 0
  selectedItems.value.forEach(item => {
    total += calculateItemProfit(item)
  })
  return parseFloat(total.toFixed(2))
})

// Calculate total selling price
const totalSellingPrice = computed(() => {
  return parseFloat(totalPrice.value + totalProfit.value).toFixed(2)
})

// FORMAT TIME AGO IN BANGLA
function formatTimeAgo(time) {
  if (!time) return 'এখনই'
  
  const now = new Date()
  const itemTime = new Date(time)
  const diffMinutes = Math.floor((now - itemTime) / (1000 * 60))
  
  if (diffMinutes < 1) return 'এখনই'
  if (diffMinutes === 1) return '১ মিনিট'
  if (diffMinutes < 60) return `${diffMinutes} মিনিট`
  
  const diffHours = Math.floor(diffMinutes / 60)
  if (diffHours === 1) return '১ ঘন্টা'
  if (diffHours < 24) return `${diffHours} ঘন্টা`
  
  const diffDays = Math.floor(diffHours / 24)
  if (diffDays === 1) return '১ দিন'
  return `${diffDays} দিন`
}

// FORMAT TIME ONLY (for saved data)
function formatTimeOnly(time) {
  return date.formatDate(time, 'h:mm')
}

// GET TIME COLOR BASED ON RECENCY
function getTimeColor(time) {
  if (!time) return 'text-grey'
  
  const now = new Date()
  const itemTime = new Date(time)
  const diffMinutes = Math.floor((now - itemTime) / (1000 * 60))
  
  if (diffMinutes < 2) return 'text-green'
  if (diffMinutes < 5) return 'text-blue'
  if (diffMinutes < 10) return 'text-orange'
  return 'text-grey'
}

// GROUP SAVED ITEMS BY 5-MINUTE INTERVALS
const groupedSavedItems = computed(() => {
  const groups = []
  
  savedItems.value.forEach((item, index) => {
    const itemTime = new Date(item.time)
    const roundedMinutes = Math.floor(itemTime.getMinutes() / 5) * 5
    const groupTime = new Date(itemTime)
    groupTime.setMinutes(roundedMinutes, 0, 0)
    
    const groupKey = groupTime.toISOString()
    
    let group = groups.find(g => g.key === groupKey)
    if (!group) {
      group = { key: groupKey, time: groupTime, items: [] }
      groups.push(group)
    }
    
    group.items.push({ ...item, originalIndex: index })
  })
  
  return groups.sort((a, b) => new Date(b.time) - new Date(a.time))
})

// FORMAT GROUP TIME
function formatGroupTime(time) {
  const dateObj = new Date(time)
  const formattedTime = date.formatDate(dateObj, 'h:mm A')
  
  const now = new Date()
  const diffMinutes = Math.floor((now - dateObj) / (1000 * 60))
  
  if (diffMinutes <= 5) return `${formattedTime} (৫ মিনিট আগে)`
  if (diffMinutes <= 10) return `${formattedTime} (১০ মিনিট আগে)`
  
  return `${formattedTime} (${Math.floor(diffMinutes/5)*5} মিনিট আগে)`
}

// COMPUTED: TOTAL PRICE
const totalPrice = computed(() =>
  selectedItems.value.reduce((acc, item) => acc + Number(item.totalPrice), 0)
)

// COMPUTED: TOTAL MON ITEMS
const totalMonItems = computed(() =>
  selectedItems.value.filter(item => item.isMon).length
)

// COMPUTED: TOTAL PIECES ITEMS
const totalPiecesItems = computed(() =>
  selectedItems.value.filter(item => item.isPieces).length
)

// COMPUTED: TOTAL KG
const totalKg = computed(() => {
  const regularKg = selectedItems.value
    .filter(item => !item.isPieces && !item.isMon)
    .reduce((acc, item) => acc + item.quantity, 0)
  
  const monKg = selectedItems.value
    .filter(item => item.isMon)
    .length * 40
  
  return regularKg + monKg
})

// COMPUTED: TOTAL PIECES
const totalPieces = computed(() => {
  return selectedItems.value
    .filter(item => item.isPieces)
    .reduce((acc, item) => acc + item.quantity, 0)
})
// Add these to your state variables
const showPrintPreview = ref(false)
const printFormat = ref('official') // 'official' or 'simple'
const printWithDate = ref(true)
const printFormats = [
  { label: 'সরকারি ফরম্যাট', value: 'official' },
  { label: 'সাধারণ ফরম্যাট', value: 'simple' }
]

// Add these computed properties
const currentDate = computed(() => {
  return date.formatDate(new Date(), 'DD MMMM YYYY')
})

const currentTime = computed(() => {
  return date.formatDate(new Date(), 'h:mm A')
})

// Add these methods to your existing methods
function printOfficialFormat() {
  printFormat.value = 'official'
  showPrintPreview.value = true
}

function printSimpleFormat() {
  printFormat.value = 'simple'
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
  doc.write('<title>বাজার খরচ বিবরণী</title>')
  doc.write('<meta charset="UTF-8">')
  doc.write('<meta name="viewport" content="width=device-width, initial-scale=1.0">')
  
  // Add styles
  if (printFormat.value === 'official') {
    doc.write('<style>')
    doc.write('@media print {')
    doc.write('  body { font-family: "Siyam Rupali", "SolaimanLipi", Arial, sans-serif; }')
    doc.write('  table { width: 100%; border-collapse: collapse; }')
    doc.write('  th, td { border: 1px solid #000; padding: 8px; }')
    doc.write('  th { background-color: #f0f0f0; text-align: center; }')
    doc.write('  tfoot { font-weight: bold; background-color: #f0f0f0; }')
    doc.write('  h3 { text-align: center; margin-bottom: 16px; }')
    doc.write('}')
    doc.write('</style>')
  } else {
    doc.write('<style>')
    doc.write('@media print {')
    doc.write('  body { font-family: "Kalpurush", Arial, sans-serif; }')
    doc.write('  .item-row { margin-bottom: 12px; padding-bottom: 8px; border-bottom: 1px dashed #ddd; }')
    doc.write('  .item-title { font-weight: bold; font-size: 1.1em; }')
    doc.write('  .total-section { margin-top: 24px; padding-top: 16px; border-top: 2px solid #1976d2; }')
    doc.write('}')
    doc.write('</style>')
  }
  
  doc.write('</head>')
  doc.write('<body>')
  doc.write(printContent.innerHTML)
  
  // Add print script - NOTE: We split the script tag to avoid Vue parsing issues
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
.rounded-borders {
  border-radius: 6px;
}

.q-card {
  border-radius: 8px;
}

.q-page {
  padding: 8px;
}

.q-field--dense .q-field__control {
  min-height: 40px;
}

.q-btn--dense {
  min-height: 40px;
}

.selected-for-delete {
  background-color: #ffebee !important;
  border: 2px solid #f44336 !important;
}
/* Add these print styles */
@media print {
  .print-content {
    font-family: 'Kalpurush', 'Siyam Rupali', 'SolaimanLipi', Arial, sans-serif;
    direction: ltr;
  }
  
  .no-print {
    display: none !important;
  }
  
  .official-format table {
    page-break-inside: avoid;
  }
  
  .simple-format .item-row {
    page-break-inside: avoid;
  }
}

/* Add font imports at the top of your main HTML file or in CSS */
@import url('https://fonts.maateen.me/kalpurush/font.css');
@import url('https://fonts.maateen.me/solaiman-lipi/font.css');

.print-content {
  font-family: 'Kalpurush', 'SolaimanLipi', Arial, sans-serif;
}
@media (max-width: 600px) {
  .q-card-section {
    padding: 6px;
  }
  
  .text-body2 {
    font-size: 0.9rem;
  }
  
  .text-body1 {
    font-size: 1rem;
  }
  
  .text-caption {
    font-size: 0.75rem;
  }
}
</style>