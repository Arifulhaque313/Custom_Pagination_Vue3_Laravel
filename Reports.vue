<script setup>
import mapper from '@/Mapper/PurchasesMapper.js'
import Table from '@/Table/Table.vue'
import { reactive, watch } from 'vue'
import { router,useForm,Link } from '@inertiajs/vue3'
import Label from '@/Components/Utilities/Form/Label.vue'
import SearchIcon from '@/Components/icons/SearchIcon.vue'
import ResetButton from '@/Components/Utilities/Button/ResetButton.vue'
import RestoreIcon from "@/Components/icons/RestoreIcon.vue"
import PrinterIcon from '@/Components/icons/PrinterIcon.vue'
import { ref, computed } from 'vue';
import EditIcon from '@/Components/icons/pencilicon.vue'
import ShowIcon from '@/Components/icons/EyeIcon.vue'
import { TrashIcon, DocumentTextIcon, CurrencyBangladeshiIcon  } from "@heroicons/vue/24/outline"
import can from '@/Helper/CheckPermission'
import Pagination from '@/Components/Utilities/Pagination.vue'


const bradecrumbName = 'Purchases Report'

const props = defineProps({
  purchases: {
    type: Object
  },
})

const form = useForm({
    from: null,
    to:null,
})

// current date
var currentDate = new Date();
var year = currentDate.getFullYear();
var month = String(currentDate.getMonth() + 1).padStart(2, '0');
var day = String(currentDate.getDate()).padStart(2, '0');
var formattedDate = day + '-' + month + '-' + year;
// currentDate finished

// Calculation
const totalPaid = computed(() => {
  const filteredNumbers = props.purchases.filter(item => item.paid_amount !== null && item.paid_amount !== undefined);
  return filteredNumbers.reduce((acc, item) => acc + parseFloat(item.paid_amount), 0);
});

const totalDue = computed(() => {
  const filteredNumbers = props.purchases.filter(item => item.due_amount !== null && item.due_amount !== undefined);
  return filteredNumbers.reduce((acc, item) => acc + parseFloat(item.due_amount), 0);
});

const totalAmount = computed(() => {
  const filteredNumbers = props.purchases.filter(item => item.grand_total !== null && item.grand_total !== undefined);
  return filteredNumbers.reduce((acc, item) => acc + parseFloat(item.grand_total), 0);
});


const submit = () => {
    form.get(route('purchases.report'), {
        preserveScroll: true,
        preserveState: true,
        onSuccess: (page) => {
            form.reset()
        }
    })
}

const reset = () => {
  router.visit('/purchase/report');
}


const currentPage = ref(props.purchases.current_page)
const paginationTotal = ref(props.purchases.total)
const pageSize = ref(props.purchases.per_page)

// update page number
const onUpdatePage = (page) => {
  let url = new window.URL(`${document.location}`);
            url.searchParams.set("page", page);

            history.pushState(null, null, url);
  // currentPage.value = page
  // router.visit();


  router.visit(url, {
    method: 'get',
    data: {},
    replace: false,
    preserveState: true,
    preserveScroll: true,
    only: [],
    headers: {},
    errorBag: null,
    forceFormData: false,
    onCancelToken: cancelToken => {},
    onCancel: () => {},
    onBefore: visit => {},
    onStart: visit => {},
    onProgress: progress => {},
    onSuccess: page => {},
    onError: errors => {},
    onFinish: visit => {},
  })
};



</script>

<template>
  <div class="">
    <h1 class="text-center text-[34px] text-gray-700 dark:text-white font-extrabold">Purchase Report</h1>
    <p class="text-center text-2xl text-gray-700 dark:text-gray-300 font-semibold">Today: {{ formattedDate }}</p>
  </div>

 

  <div>
    <table class="min-w-full my-4 border border-violet-600 rounded-sm px-8">
    <thead>
      <tr class="bg-violet-600 text-white !rounded-xl">
        <th class="w-8 pl-4 py-3 text-sm text-left">SL</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Date</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Invoice</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Supplier</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Issued By</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Created_at</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Paid</th>
        <th class="w-6 pl-4 py-3 text-sm text-left">Due</th>
        <th class="w-6 pl-4 py-3 text-sm text-left">Total</th>
        <th class="w-8 pl-4 py-3 text-sm text-left">Actions</th>
      </tr>

    </thead>
    <tbody class="bg-white dark:bg-slate-900 divide-y divide-gray-200 dark:divide-gray-700">
      <tr v-for="(item,index) in purchases.data" :key="index">
          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ index+1 }}</td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.date }}</td>
          <!-- <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.invoice_id }}</td> -->
          <td class="w-8 pl-4 py-3 text-sm text-violet-600 dark:text-gray-100"><Link :href="`/invoice/show/${item.invoice_id}`" class="hover:border-b hover:border-violet-600">{{ item.invoice_id }}</Link></td>

          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.supplier }}</td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.created_by }}</td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.created_at_formatted }}</td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.paid_amount }}</td>
          <td class="w-6 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.due_amount }}</td>
          <td class="w-6 pl-4 py-3 text-sm text-gray-500 dark:text-gray-100">{{ item.grand_total }}</td>
          <td class="w-[80px] pl-4 py-3 text-sm text-gray-500 dark:text-gray-100 flex justify-center items-center gap-x-1">
              <Link :href="`/trading/${item.id}/journal`"><DocumentTextIcon class="bg-purple-50 hover:bg-purple-100 rounded-full p-2 w-8 h-8 text-purple-600"/></Link>
              <Link :href="`/payment/${item.id}/history`"><CurrencyBangladeshiIcon class="bg-green-50 hover:bg-green-100 rounded-full p-2 w-8 h-8 text-green-600"/></Link>
              <Link href="#"><ShowIcon class="bg-sky-50 hover:bg-sky-100 rounded-full p-2 w-8 h-8 text-sky-600"/></Link>
              <Link v-if="can('app.accounts.update')" :href="`/purchase/${item.id}/edit`"><EditIcon class="bg-yellow-50 hover:bg-yellow-100 rounded-full p-2 w-8 h-8 text-yellow-600"/></Link>
              <Link v-if="can('app.accounts.update')" href="#"><TrashIcon class="bg-red-50 hover:bg-red-100 rounded-full p-2 w-8 h-8 text-red-600"/></Link>
          </td>
      </tr>

      <!-- <tr>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm text-gray-500"></td>
          <td class="w-8 pl-4 py-3 text-sm font-bold  ">
            <span class="text-white bg-green-600 px-4 py-1 rounded-lg">
            {{ totalPaid }} &#2547;
            </span>
          </td>
          <td class="w-8 pl-4 py-3 text-sm font-bold">
                <span class="text-white bg-red-600 px-4 py-1 rounded-lg">
                    {{ totalDue}} &#2547;
                </span>
            </td>
          <td class="w-8 pl-4 py-3 text-sm font-bold">
            <span class="text-white bg-violet-600 px-4 py-1 rounded-lg">
             {{ totalAmount }} &#2547;
            </span>
            </td>
      </tr> -->
    </tbody>
  </table>

  <div class="">
    <div v-if="purchases.data.length > 20">
      <Pagination  
        v-model="currentPage"
        @update:modelValue="onUpdatePage(currentPage)"
        :page-count="Math.ceil(paginationTotal / pageSize)"
        :margin-pages="2"
        :page-range="3"
        :currentPage = currentPage
        :pageSize = pageSize
        :paginationTotal = paginationTotal
        :first-last-button="true"
      />
    </div>
  </div>
  </div>

</template>


