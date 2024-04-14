<template>
    <div class="bg-white" v-if="!showSkeleton">
      <div class="mx-auto max-w-2xl px-4 sm:px-6 lg:max-w-7xl lg:px-8">
        <h2 class="text-2xl font-bold text-gray-900 tracking-wide">Your Order</h2>
  
        <div class="mt-6 grid grid-cols-1 gap-x-6 gap-y-10 sm:grid-cols-2 lg:grid-cols-4 xl:gap-x-8">
          <div v-for="order in orders" :key="order.id" class="group relative">
            <div class="aspect-h-1 aspect-w-1 w-full overflow-hidden rounded-lg bg-gray-200 lg:aspect-none lg:h-80 shadow-xl">
              <img src="../assets/productbg.jpg" alt="../assets/productbg.jpg"
              class="h-full w-full object-cover object-center lg:h-full lg:w-full">
            </div>
            <div class='absolute top-5 left-0 right-0 px-4 py-2 text-center'>
                <p class="text-lg font-semibold text-gray-800">
                    Total Products
                </p>
                <p class="font-semibold text-gray-800 my-1">
                    <span class="text-big text-[#40b783]">{{order.products.length}}</span>
                </p>
                <div class='flex justify-between'>
                    <div>
                        <p class="text-lg font-semibold text-gray-800">
                            Total Qty
                        </p>
                        <p class="font-semibold text-gray-800">
                            <span class="text-big text-[#40b783]">{{order.total_quantity}}</span>
                        </p>
                    </div>
                    <div>
                        <p class="text-lg font-semibold text-gray-800">
                            Total Price
                        </p>
                        <p class="text-sm font-semibold text-gray-800 my-3">
                            <span class="text-big text-[#40b783]">$ {{order.total_price}}</span>
                        </p>
                    </div>
                </div>

            </div>
            <div class='absolute bottom-custom right-0 px-4 py-2 flex justify-between'>
                <CalendarIcon class="w-4 h-4 mt-1 mx-1 text-gray-400"/>
                <span class='text-gray-400'>
                    {{formatDate(order.createdAt)}}
                </span>
            </div>
            <div class="absolute bottom-0 left-0 right-0 bg-white bg-opacity-75 px-4 py-2">
                <button @click="handleOrderDetail(true, order)" class="w-full mt-2 px-4 py-2 bg-[#40b783] text-white rounded-md hover:bg-[#33485e] focus:outline-none focus:bg-[#33485e]">
                    View Details
                </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <div class="bg-white">
          <div class="mx-auto max-w-2xl px-4 py-16 sm:px-6 sm:py-24 lg:max-w-7xl lg:px-8">
              <div class="mt-6 grid grid-cols-1 gap-x-6 gap-y-10 sm:grid-cols-2 lg:grid-cols-4 xl:gap-x-8">
                  <div  v-for="(item, index) in skeletons" :key="index">
                    <div class="group relative">
                        <div class="aspect-h-1 aspect-w-1 w-full overflow-hidden rounded-md bg-gray-200 lg:aspect-none group-hover:opacity-75 lg:h-80 relative">
                            <Skeleton class="absolute inset-0 w-full h-full" count={1} /> 
                        </div>
                        <div class="mt-4 flex justify-between">
                            <div>
                                <h3 class="text-sm text-gray-700">
                                    <a href='/#'>
                                        <span aria-hidden="true" class="absolute inset-0" />
                                    </a>
                                </h3>
                            </div>
                        </div>
                    </div>
                  </div>
              </div>
          </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import axios from 'axios';
  import { onMounted, ref, defineEmits, defineProps,watch } from 'vue';
  import { Skeleton } from 'vue-loading-skeleton';
  import { CalendarIcon } from '@heroicons/vue/24/outline'
  import "vue-loading-skeleton/dist/style.css"
  import Cookies from 'js-cookie';
  
  const orders = ref([]);
  const showSkeleton = ref(true);
  const skeletons = Array.from({ length: 4 });
  const emit = defineEmits(['updateDetailCart', 'updateOrderDetail', 'updateCancelStatus', 'updateConfirmDialogStatus']);
  const existOrderDetail = ref(null);

  const props = defineProps({
    isCancel: {
      type: Boolean,
      default: false
    },
    isReloadOrderPage:{
        type: Boolean,
        default: false
    }
  });

const cancelOrder = async () => {
    try {
        const response = await axios.delete('http://localhost:3000/orders/' + existOrderDetail.value);
        if(response.data.message === 'Order deleted successfully'){
            console.log('Orders deleted successfully:', response.data);
            emit('updateDetailCart', false);
            emit('updateCancelStatus', false);
            emit('updateConfirmDialogStatus', false);
            getOrders();
        }
    } catch (error) {
        console.error('Error deleting orders:', error.message);
    }
}

const getOrders = () => {
    axios.get('http://localhost:3000/orders?session_token=' + Cookies.get('session_token'))
    .then(response => {
        orders.value = response.data
        showSkeleton.value = false;
        emit('updateReloadingOrder', false)
    })
    .catch(error => {
        console.error('Error fetching data:', error);
    });
}

const handleOrderDetail = (status, order) => {
    existOrderDetail.value = order._id;
    emit('updateOrderDetail', order)
    emit('updateDetailCart', status)
}


function formatDate(dateString) {
    const date = new Date(dateString);
    const options = { year: 'numeric', month: 'short', day: 'numeric' };
    const formattedDate = date.toLocaleDateString(undefined, options); // Format date as "Mon, Jan 1, 2023"
    const hours = date.getHours();
    const minutes = date.getMinutes();
    const ampm = hours >= 12 ? 'PM' : 'AM';
    const formattedHours = hours % 12 || 12; // Convert 0 to 12 for midnight
    const formattedMinutes = minutes.toString().padStart(2, '0'); // Add leading zero if necessary
    return `${formattedDate} ${formattedHours}:${formattedMinutes} ${ampm}`;
}

watch(() => props.isCancel, (newValue) => {
  if (newValue && newValue == true) {
    cancelOrder();
  }
});

watch(() => props.isReloadOrderPage, (newValue) => {
console.log(newValue);
  if (newValue && newValue == true) {
    showSkeleton.value = true;
    getOrders();
  }
});
  
onMounted(() => {
    getOrders();
});
  
  </script>