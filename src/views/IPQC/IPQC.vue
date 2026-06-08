<template>
  <div class="min-h-screen bg-gray-50 p-4 sm:p-8">
    <div class="max-w-7xl mx-auto">
      <!-- 標題與新增按鈕 -->
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-8 gap-4">
        <div>
          <h1 class="text-3xl font-extrabold text-gray-900">IPQC 檢驗系統</h1>
          <p class="text-gray-500 mt-1">管理生產線品質檢驗紀錄</p>
        </div>
        <button
          @click="openModal()"
          class="inline-flex items-center px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-semibold rounded-lg shadow-sm transition-colors"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
          </svg>
          新增檢驗紀錄
        </button>
      </div>

      <!-- 列表區域 -->
      <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
        <div v-if="loading" class="p-12 text-center">
          <div class="animate-spin inline-block w-8 h-8 border-4 border-indigo-500 border-t-transparent rounded-full mb-4"></div>
          <p class="text-gray-500">資料載入中...</p>
        </div>

        <div v-else-if="items.length === 0" class="p-12 text-center">
          <div class="text-gray-300 mb-4">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
            </svg>
          </div>
          <p class="text-gray-500 text-lg">目前尚無檢驗紀錄</p>
          <button @click="openModal()" class="mt-4 text-indigo-600 hover:text-indigo-700 font-medium">點此新增第一筆資料</button>
        </div>

        <div v-else class="overflow-x-auto">
          <table class="w-full text-left border-collapse">
            <thead class="bg-gray-50 border-b border-gray-100">
              <tr>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">檢驗時間</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">工單編號</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">品號/品名/規格</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">數量</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">缺點現象</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600">巡檢員</th>
                <th class="px-6 py-4 text-sm font-semibold text-gray-600 text-right">操作</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-50">
              <tr v-for="item in items" :key="item.id" class="hover:bg-gray-50 transition-colors">
                <td class="px-6 py-4 text-sm text-gray-600">
                  <div>{{ item.date }}</div>
                  <div class="text-xs text-gray-400">{{ item.time }}</div>
                </td>
                <td class="px-6 py-4 text-sm text-gray-600">
                  <div class="font-medium text-gray-900 whitespace-nowrap">{{ item.order_number }}</div>
                </td>
                <td class="px-6 py-4 text-sm text-gray-600">
                  <div class="font-medium text-gray-900 whitespace-nowrap">{{ item.product_number }}</div>
                  <div class="text-xs whitespace-nowrap">{{ item.product_name }}</div>
                  <div class="text-xs text-gray-500 whitespace-nowrap">{{ item.spec }}</div>
                </td>
                <td class="px-6 py-4 text-sm text-gray-900">{{ item.quantity }}</td>
                <td class="px-6 py-4">
                  <div 
                    class="text-sm font-medium text-gray-900" 
                    :title="item.defect_status && item.defect_status.length > 10 ? item.defect_status : ''"
                  >
                    {{ item.defect_status && item.defect_status.length > 10 
                       ? item.defect_status.substring(0, 10) + '...' 
                       : item.defect_status || '無' }}
                  </div>
                  <div class="text-xs text-gray-500">{{ item.defect_classification }}</div>
                </td>
                <td class="px-6 py-4 text-sm text-gray-600 whitespace-nowrap">
                  {{ item.inspector }}
                </td>
                <td class="px-6 py-4 text-right whitespace-nowrap">
                  <button 
                    @click="openModal(item)" 
                    class="text-indigo-600 hover:text-indigo-900 p-1 rounded-md hover:bg-indigo-50 transition-colors"
                    title="編輯"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" /></svg>
                  </button>
                  <button 
                    @click="deleteItem(item.id)" 
                    class="text-red-600 hover:text-red-900 p-1 rounded-md hover:bg-red-50 transition-colors ml-2"
                    title="刪除"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    
    <!-- 表單彈窗 -->
    <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-gray-900/50 backdrop-blur-sm">
      <div class="bg-white rounded-2xl shadow-xl w-full max-w-2xl overflow-hidden">
        <div class="px-6 py-4 border-b border-gray-100 flex justify-between items-center">
          <h3 class="text-xl font-bold text-gray-900">{{ editingId ? '編輯紀錄' : '新增檢驗紀錄' }}</h3>
          <button @click="showModal = false" class="text-gray-400 hover:text-gray-600">
            <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
          </div>
        <div class="p-6 space-y-4 max-h-[70vh] overflow-y-auto">
          <!-- 第一組：時間資訊 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">日期</label>
              <input v-model="form.date" type="date" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">時間</label>
              <input v-model="form.time" type="time" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" />
            </div>
          </div>

          <!-- 第三組：工單與人員 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1 flex items-center justify-between">
                <span>工單編號</span>
                <button 
                  @click="openOrderModal" 
                  type="button" 
                  class="text-indigo-600 hover:text-indigo-800 p-0.5" 
                  title="快速新增工單">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                </button>
              </label>
              <div class="relative">
                <input 
                  v-model="form.order_number" 
                  type="text" 
                  class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" 
                  placeholder="請輸入工單編號" 
                />
                <!-- 工單比對提示 -->
                <div v-if="isSearchingOrder" class="absolute right-3 top-2.5">
                  <div class="animate-spin h-4 w-4 border-2 border-indigo-500 border-t-transparent rounded-full"></div>
                </div>
              </div>
              <p v-if="isSearchingOrder" class="mt-1 text-xs text-indigo-500">
                正在比對工單與規格資料...
              </p>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">生產人員</label>
              <select 
                v-model="form.operator" 
                class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
              >
                <option value="">請選擇生產人員</option>
                <option v-for="op in operators" :key="op.name" :value="op.name">
                  {{ op.name }}
                </option>
              </select>
            </div>
          </div>

          <!-- 第四組：品號與版次 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1 flex items-center justify-between">
                <span>品號</span>
                <button 
                  @click="openSpecModal" 
                  type="button" 
                  class="text-indigo-600 hover:text-indigo-800 p-0.5" 
                  title="快速新增規格">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                </button>
              </label>
              <input 
                v-model="form.product_number" 
                type="text" 
                :class="[
                  'w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500',
                  { 'text-blue-600 font-bold': autoMatchedFields.product_number && form.product_number === matchedData.product_number }
                ]"
                placeholder="品號" 
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">版次 (Draw Ver)</label>
              <input 
                v-model="form.draw_ver" 
                type="text" 
                :class="[
                  'w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500',
                  { 'text-blue-600 font-bold': autoMatchedFields.draw_ver && form.draw_ver === matchedData.version }
                ]"
                placeholder="版次" 
              />
            </div>
          </div>

          <!-- 第五組：品名與規格 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">品名</label>
              <input 
                v-model="form.product_name" 
                type="text" 
                :class="[
                  'w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500',
                  { 'text-blue-600 font-bold': autoMatchedFields.product_name && form.product_name === matchedData.product_name }
                ]"
                placeholder="產品名稱" 
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">規格 (Spec)</label>
              <input 
                v-model="form.spec" 
                type="text" 
                :class="[
                  'w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500',
                  { 'text-blue-600 font-bold': autoMatchedFields.spec && form.spec === matchedData.spec }
                ]"
                placeholder="規格" 
              />
            </div>
          </div>

          <!-- 第五組：數量與巡檢人員 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">數量</label>
              <input 
                v-model.number="form.quantity" 
                type="number" 
                :class="[
                  'w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500',
                  { 'text-blue-600 font-bold': autoMatchedFields.quantity && form.quantity === matchedData.quantity }
                ]"
                placeholder="數量" 
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">巡檢人員</label>
              <input v-model="form.inspector" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="巡檢員姓名" />
            </div>
          </div>

          <!-- 第六組：缺點資訊 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">缺點分類</label>
              <select 
                v-model="form.defect_classification" 
                class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
              >
                <option value="">請選擇缺點分類</option>
                <option v-for="defect in defectTypes" :key="defect.defect_type" :value="defect.defect_type">
                  {{ defect.defect_type }}
                </option>
              </select>
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">缺點現象</label>
              <input v-model="form.defect_status" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="例如：刮傷" />
            </div>
          </div>

          <!-- 第七組：處理措施 -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">處理措施</label>
            <textarea v-model="form.handling_measures" rows="2" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入處理方式..."></textarea>
          </div>

          <!-- 第八組：備註 -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">備註 (Remark)</label>
            <textarea v-model="form.remark" rows="2" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="其他說明..."></textarea>
          </div>
        </div>
        <div class="px-6 py-4 bg-gray-50 flex justify-end gap-3">
          <button @click="showModal = false" class="px-4 py-2 text-sm font-medium text-gray-700 hover:text-gray-900">取消</button>
          <button 
            @click="saveItem" 
            :disabled="saving"
            class="px-4 py-2 bg-indigo-600 text-white text-sm font-medium rounded-lg hover:bg-indigo-700 disabled:opacity-50"
          >
            {{ saving ? '儲存中...' : '確認送出' }}
          </button>
        </div>
      </div>
    </div>

    <!-- 快速新增工單彈窗 -->
    <div v-if="showOrderModal" class="fixed inset-0 z-[60] flex items-center justify-center p-4 bg-gray-900/60 backdrop-blur-sm">
      <div class="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden">
        <div class="px-6 py-4 border-b border-gray-100 flex justify-between items-center bg-indigo-50">
          <h3 class="text-lg font-bold text-indigo-900">新增工單資料</h3>
          <button @click="showOrderModal = false" class="text-gray-400 hover:text-gray-600">
            <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
        <div class="p-6 space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">工單編號</label>
            <input v-model="orderForm.order_number" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入新工單號碼" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">品號</label>
            <input v-model="orderForm.product_number" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入產品品號" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">品名</label>
            <input v-model="orderForm.product_name" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入產品名稱" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">工單數量</label>
            <input v-model.number="orderForm.quantity" type="number" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入計畫生產數量" />
          </div>
        </div>
        <div class="px-6 py-4 bg-gray-50 flex justify-end gap-3">
          <button 
            @click="showOrderModal = false" 
            class="px-4 py-2 text-sm font-medium text-gray-700 hover:text-gray-900"
          >
            取消
          </button>
          <button 
            @click="saveOrder" 
            :disabled="isSavingOrder"
            class="px-4 py-2 bg-indigo-600 text-white text-sm font-medium rounded-lg hover:bg-indigo-700 disabled:opacity-50"
          >
            {{ isSavingOrder ? '儲存中...' : '確認新增' }}
          </button>
        </div>
      </div>
    </div>

    <!-- 快速新增規格彈窗 -->
    <div v-if="showSpecModal" class="fixed inset-0 z-[60] flex items-center justify-center p-4 bg-gray-900/60 backdrop-blur-sm">
      <div class="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden">
        <div class="px-6 py-4 border-b border-gray-100 flex justify-between items-center bg-indigo-50">
          <h3 class="text-lg font-bold text-indigo-900">新增規格資料</h3>
          <button @click="showSpecModal = false" class="text-gray-400 hover:text-gray-600">
            <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
        <div class="p-6 space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">品號</label>
            <input v-model="specForm.product_number" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入產品品號" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">品名</label>
            <input v-model="specForm.product_name" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入產品名稱" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">規格</label>
            <input v-model="specForm.spec" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入產品規格" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">版次</label>
            <input v-model="specForm.version" type="text" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入版次" />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">檢驗工時</label>
            <input v-model.number="specForm.inspection_hours" type="number" step="0.1" class="w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500" placeholder="輸入檢驗工時" />
          </div>
        </div>
        <div class="px-6 py-4 bg-gray-50 flex justify-end gap-3">
          <button 
            @click="showSpecModal = false" 
            class="px-4 py-2 text-sm font-medium text-gray-700 hover:text-gray-900"
          >
            取消
          </button>
          <button 
            @click="saveSpec" 
            :disabled="isSavingSpec"
            class="px-4 py-2 bg-indigo-600 text-white text-sm font-medium rounded-lg hover:bg-indigo-700 disabled:opacity-50"
          >
            {{ isSavingSpec ? '儲存中...' : '確認新增' }}
          </button>
        </div>
      </div>
    </div>

  </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, watch } from 'vue'
import { supabase } from '@/supabase'
import { useAuthStore } from '@/stores/auth'
import { useToast } from 'vue-toastification'

const authStore = useAuthStore()
const toast = useToast()

const items = ref([])
const loading = ref(true)
const saving = ref(false)
const defectTypes = ref([]) // 新增：缺點分類清單
const operators = ref([])
const showModal = ref(false)
const editingId = ref(null)

// 工單新增相關狀態
const showOrderModal = ref(false)
const isSavingOrder = ref(false)
const orderForm = reactive({
  order_number: '',
  product_number: '',
  product_name: '',
  quantity: null
})

// Spec 新增相關狀態
const showSpecModal = ref(false)
const isSavingSpec = ref(false)
const specForm = reactive({
  product_number: '',
  product_name: '',
  spec: '',
  version: '',
  inspection_hours: null
})

// 工單比對相關狀態
const isSearchingOrder = ref(false)
const matchedData = reactive({
  product_number: '',
  product_name: '',
  quantity: null,
  spec: '',
  version: ''
})

// 追蹤哪些欄位是自動帶入的
const autoMatchedFields = reactive({
  product_number: false,
  product_name: false,
  quantity: false,
  spec: false,
  draw_ver: false
})

const form = reactive({
  date: new Date().toISOString().split('T')[0],
  time: new Date().toLocaleTimeString('it-IT', { hour: '2-digit', minute: '2-digit' }),
  order_number: '',
  operator: '',
  draw_ver: '',
  product_number: '',
  product_name: '',
  spec: '',
  quantity: 1,
  inspector: authStore.user?.user_metadata?.full_name || authStore.user?.email || '',
  defect_classification: '',
  defect_status: '',
  handling_measures: '',
  remark: '',
})

// 監聽工單編號輸入，自動比對 order_list
let searchTimer = null
watch(() => form.order_number, (newVal) => {
  // 清除舊的比對結果與狀態
  Object.keys(matchedData).forEach(key => matchedData[key] = key === 'quantity' ? null : '')
  Object.keys(autoMatchedFields).forEach(key => autoMatchedFields[key] = false)
  
  if (!newVal || newVal.length < 3) return // 長度太短不搜尋

  // 防抖處理
  clearTimeout(searchTimer)
  searchTimer = setTimeout(async () => {
    isSearchingOrder.value = true
    try {
      const { data, error } = await supabase
        .from('order_list')
        .select('product_number, product_name, quantity')
        .eq('order_number', newVal)
        .maybeSingle()

      if (error) throw error

      if (data) {
        // 填入工單相關資料
        const updateFields = (fields) => {
          if (fields.product_number) {
            matchedData.product_number = fields.product_number
            form.product_number = fields.product_number
            autoMatchedFields.product_number = true
          }
          if (fields.product_name) {
            matchedData.product_name = fields.product_name
            form.product_name = fields.product_name
            autoMatchedFields.product_name = true
          }
          if (fields.quantity !== undefined) {
            matchedData.quantity = fields.quantity
            form.quantity = fields.quantity
            autoMatchedFields.quantity = true
          }
        }

        updateFields(data)

        // 額外邏輯：根據工單編號去 spec_list 找品號 (如 user 要求)
        const { data: specByOrder } = await supabase
          .from('spec_list')
          .select('product_number')
          .eq('order_number', newVal)
          .maybeSingle()
        
        const finalProductNumber = data.product_number || specByOrder?.product_number

        // 接著根據品號去 match spec_list 找出規格與版次
        if (finalProductNumber) {
          const { data: specData, error: specError } = await supabase
            .from('spec_list')
            .select('spec, version')
            .eq('product_number', finalProductNumber)
            .maybeSingle()
          
          if (!specError && specData) {
            matchedData.spec = specData.spec
            matchedData.version = specData.version
            form.spec = specData.spec
            form.draw_ver = specData.version
            autoMatchedFields.spec = true
            autoMatchedFields.draw_ver = true
          }
        }
      } else {
        // 若工單編號沒有完全符合，顯示預設值
        form.product_number = '未設定'
        form.product_name = '未設定'
        form.draw_ver = '未設定'
        form.spec = '未設定'
        form.quantity = 0
      }
    } catch (error) {
      console.error('Order matching error:', error)
    } finally {
      isSearchingOrder.value = false
    }
  }, 500)
})

// 抓取資料
const fetchItems = async () => {
  loading.value = true
  try {
    const { data, error } = await supabase
      .from('ipqc_list')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    items.value = data
  } catch (error) {
    toast.error('無法抓取資料: ' + error.message)
  } finally {
    loading.value = false
  }
}

// 抓取生產人員清單
const fetchOperators = async () => {
  try {
    const { data, error } = await supabase
      .from('operator_list')
      .select('name')
      .order('name', { ascending: true })
    
    if (error) throw error
    operators.value = data || []
  } catch (error) {
    console.error('無法抓取人員名單:', error.message)
  }
}

// 抓取缺點分類清單
const fetchDefectTypes = async () => {
  try {
    const { data, error } = await supabase
      .from('defect_list')
      .select('defect_type')
      .order('defect_type', { ascending: true })
    
    if (error) throw error
    defectTypes.value = data || []
  } catch (error) {
    console.error('無法抓取缺點分類名單:', error.message)
  }
}

/**
 * 開啟新增工單彈窗
 */
const openOrderModal = () => {
  orderForm.order_number = ''
  orderForm.product_number = ''
  orderForm.product_name = ''
  orderForm.quantity = null
  showOrderModal.value = true
}

/**
 * 儲存新工單到 order_list
 */
const saveOrder = async () => {
  if (!orderForm.order_number || !orderForm.product_number) {
    toast.warning('工單編號與品號為必填')
    return
  }

  isSavingOrder.value = true
  try {
    const { error } = await supabase
      .from('order_list')
      .insert([orderForm])
    
    if (error) throw error
    
    toast.success('工單已成功新增')
    // 自動帶入到目前的檢驗表單
    form.order_number = orderForm.order_number
    showOrderModal.value = false
  } catch (error) {
    toast.error('新增工單失敗: ' + error.message)
  } finally {
    isSavingOrder.value = false
  }
}

/**
 * 開啟新增規格彈窗
 */
const openSpecModal = () => {
  specForm.product_number = form.product_number
  specForm.product_name = form.product_name
  specForm.spec = ''
  specForm.version = ''
  specForm.inspection_hours = null
  showSpecModal.value = true
}

/**
 * 儲存新規格到 spec_list
 */
const saveSpec = async () => {
  if (!specForm.product_number) {
    toast.warning('品號為必填')
    return
  }

  isSavingSpec.value = true
  try {
    const { error } = await supabase
      .from('spec_list')
      .insert([specForm])
    
    if (error) throw error
    
    toast.success('規格已成功新增')
    // 自動帶入到目前的檢驗表單
    form.product_number = specForm.product_number
    form.product_name = specForm.product_name
    form.spec = specForm.spec
    form.draw_ver = specForm.version
    showSpecModal.value = false
  } catch (error) {
    toast.error('新增規格失敗: ' + error.message)
  } finally {
    isSavingSpec.value = false
  }
}

// 開啟彈窗 (新增或編輯)
const openModal = (item = null) => {
  // 重置自動比對狀態
  Object.keys(matchedData).forEach(key => {
    matchedData[key] = key === 'quantity' ? null : ''
  })
  Object.keys(autoMatchedFields).forEach(key => autoMatchedFields[key] = false)

  if (item) {
    editingId.value = item.id
    form.date = item.date || ''
    form.time = item.time || ''
    form.order_number = item.order_number || ''
    form.operator = item.operator || ''
    form.draw_ver = item.draw_ver || ''
    form.product_number = item.product_number || ''
    form.product_name = item.product_name || ''
    form.spec = item.spec || ''
    form.quantity = item.quantity || 1
    form.inspector = item.inspector || ''
    form.defect_classification = item.defect_classification || ''
    form.defect_status = item.defect_status || ''
    form.handling_measures = item.handling_measures || ''
    form.remark = item.remark || ''
  } else {
    editingId.value = null
    form.date = new Date().toISOString().split('T')[0]
    form.time = new Date().toLocaleTimeString('it-IT', { hour: '2-digit', minute: '2-digit' })
    form.order_number = ''
    form.operator = ''
    form.draw_ver = ''
    form.product_number = ''
    form.product_name = ''
    form.spec = ''
    form.quantity = 1
    form.inspector = authStore.user?.user_metadata?.full_name || authStore.user?.email || ''
    form.defect_classification = ''
    // form.defect_classification = defectTypes.value.length > 0 ? defectTypes.value[0].defect_type : '' // 或者設定第一個選項為預設
    form.defect_status = ''
    form.handling_measures = ''
    form.remark = ''
  }
  showModal.value = true
}

// 儲存資料 (Create / Update)
const saveItem = async () => {
  if (!form.product_name) {
    toast.warning('請填寫產品名稱')
    return
  }

  saving.value = true
  try {
    if (editingId.value) {
      // 更新
      const { error } = await supabase
        .from('ipqc_list')
        .update({
          date: form.date,
          time: form.time,
          order_number: form.order_number,
          operator: form.operator,
          draw_ver: form.draw_ver,
          product_number: form.product_number,
          product_name: form.product_name,
          spec: form.spec,
          quantity: form.quantity,
          inspector: form.inspector,
          defect_classification: form.defect_classification,
          defect_status: form.defect_status,
          handling_measures: form.handling_measures,
          remark: form.remark,
        })
        .eq('id', editingId.value)
      if (error) throw error
      toast.success('更新成功')
    } else {
      // 新增
      const { error } = await supabase
        .from('ipqc_list')
        .insert([{ ...form }])
      if (error) throw error
      toast.success('新增成功')
    }
    showModal.value = false
    fetchItems()
  } catch (error) {
    toast.error('儲存失敗: ' + error.message)
  } finally {
    saving.value = false
  }
}

// 刪除資料 (Delete)
const deleteItem = async (id) => {
  if (!confirm('您確定要刪除此筆檢驗紀錄嗎？')) return

  try {
    const { error } = await supabase
      .from('ipqc_list')
      .delete()
      .eq('id', id)
    if (error) throw error
    toast.success('資料已刪除')
    fetchItems()
  } catch (error) {
    toast.error('刪除失敗: ' + error.message)
  }
}

onMounted(() => {
  fetchItems()
  fetchOperators()
  fetchDefectTypes() // 新增：載入缺點分類
})
</script>