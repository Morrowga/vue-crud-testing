<template>
    <TransitionRoot as="template" :show="props.showCart">
      <Dialog as="div" class="relative z-10" @close="closeCartModal(false)">
        <TransitionChild as="template" enter="ease-in-out duration-500" enter-from="opacity-0" enter-to="opacity-100" leave="ease-in-out duration-1000" leave-from="opacity-100" leave-to="opacity-0">
          <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" />
        </TransitionChild>
  
        <div class="fixed inset-0 overflow-hidden">
          <div class="absolute inset-0 overflow-hidden">
            <div class="pointer-events-none fixed inset-y-0 right-0 flex max-w-full pl-10">
              <TransitionChild as="template" enter="transform transition ease-in-out duration-500 sm:duration-700" enter-from="translate-x-full" enter-to="translate-x-0" leave="transform transition ease-in-out duration-500 sm:duration-700" leave-from="translate-x-0" leave-to="translate-x-full">
                <DialogPanel class="pointer-events-auto w-screen max-w-md">
                  <div class="flex h-full flex-col overflow-y-scroll bg-white shadow-xl">
                    <div class="flex-1 overflow-y-auto px-4 py-6 sm:px-6">
                      <div class="flex items-start justify-between">
                        <DialogTitle class="text-lg font-medium text-gray-900 tracking-wide font-bold">Your Shopping cart</DialogTitle>
                        <div class="ml-3 flex h-7 items-center">
                          <button type="button" class="relative -m-2 p-2 text-gray-400 hover:text-gray-500" @click="closeCartModal(false)">
                            <span class="absolute -inset-0.5" />
                            <span class="sr-only">Close panel</span>
                            <XMarkIcon class="h-6 w-6" aria-hidden="true" />
                          </button>
                        </div>
                      </div>
  
                      <div class="mt-8">
                        <div class="flow-root">
                          <ul role="list" class="-my-6 divide-y divide-gray-200">
                            <li v-for="product in props.isAdded" :key="product.id" class="flex py-6">
                              <div class="h-24 w-24 flex-shrink-0 overflow-hidden rounded-md border border-gray-200">
                                <img :src="product.image" :alt="product.image" class="h-full w-full object-cover object-center" />
                              </div>
  
                              <div class="ml-4 flex flex-1 flex-col">
                                <div>
                                  <div class="flex justify-between text-base font-medium text-gray-900">
                                    <h3>
                                      <a :href="product.href">{{ product.name }}</a>
                                    </h3>
                                    <p class="ml-4">$ {{ product.price }}</p>
                                  </div>
                                </div>
                                <p class="my-3 text-sm">Qty</p>
                                <div class="flex flex-1 items-end justify-between text-sm">
                                  <div class="flex items-center">
                                    <button
                                        @click="decreaseQuantity(product._id)"
                                        class="px-3 py-1 rounded-l border border-gray-300 bg-gray-100 hover:bg-gray-200"
                                    >
                                        -
                                    </button>
                                    <span class="px-3 py-1 border-t border-b border-gray-300">
                                        {{  product.qty  }}
                                    </span>
                                    <button
                                        @click="increaseQuantity(product._id)"
                                        class="px-3 py-1 rounded-r border border-gray-300 bg-gray-100 hover:bg-gray-200"
                                    >
                                        +
                                    </button>
                                  </div>
  
                                  <div class="flex">
                                    <button @click="addToCart(product)" type="button" class="font-medium text-red-600 hover:text-red-500">
                                        <TrashIcon class="w-6 h-6" />
                                    </button>
                                  </div>
                                </div>
                              </div>
                            </li>
                          </ul>
                        </div>
                      </div>
                    </div>
  
                    <div class="border-t border-gray-200 px-4 py-6 sm:px-6">
                      <div class="flex justify-between text-base font-medium text-gray-900">
                        <p>Total Quantity</p>
                        <p>{{ calculateTotalQuantity() }}</p>
                      </div>
                      <div class="flex justify-between text-base font-medium text-gray-900">
                        <p>Subtotal</p>
                        <p>$ {{ calculateTotalPrice() }}</p>
                      </div>
                      <div class="mt-6">
                        <button
                          @click="checkOut"
                          :disabled="postLoading"
                          class="w-full flex items-center justify-center rounded-md border border-transparent bg-[#40b783] px-6 py-3 text-base font-medium text-white shadow-sm hover:bg-[#33485e]"
                        >
                          <template v-if="postLoading">
                            <svg class="animate-spin h-5 w-5 mr-3 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V2.5a1.5 1.5 0 113 0V4a8 8 0 01-8 8z"></path>
                            </svg>
                          </template>
                          <template v-else>
                            Checkout
                          </template>
                        </button>
                      </div>
                      <div class="mt-6 flex justify-center text-center text-sm text-gray-500">
                        <p>
                          or {{ ' ' }}
                          <button type="button" class="font-medium text-[#40b783] hover:text-[#40b783]" @click="closeCartModal(false)">
                            Continue Shopping
                            <span aria-hidden="true"> &rarr;</span>
                          </button>
                        </p>
                      </div>
                    </div>
                  </div>
                </DialogPanel>
              </TransitionChild>
            </div>
          </div>
        </div>
      </Dialog>
    </TransitionRoot>
  </template>
  
  <script setup>
  import { defineProps, defineEmits,ref } from 'vue'
  import axios from 'axios';
  import Cookies from 'js-cookie';
  import { Dialog, DialogPanel, DialogTitle, TransitionChild, TransitionRoot } from '@headlessui/vue'
  import { XMarkIcon,TrashIcon } from '@heroicons/vue/24/outline'
  
  const props = defineProps({
    showCart: {
      type: Boolean,
      default: true
    },
    isAdded: {
      type: Array,
      default: null
    }
  });
  const postLoading = ref(false); 

  const emit = defineEmits(['updateCart', 'addToCart', 'increaseQuantity', 
  'decreaseQuantity', 'updateIsAddedIndex', 
  'updateIsAdded', 'updateCount', 'updateCheckSuccess']);

  const decreaseQuantity = (productId) => {
    const productIndex = props.isAdded.findIndex(product => product._id === productId);
    if (productIndex !== -1) {
      const updatedProduct = { ...props.isAdded[productIndex], qty: Math.max(0, props.isAdded[productIndex].qty - 1) };
      emit('updateIsAddedIndex', { index: productIndex, product: updatedProduct });
    }
  };

  const increaseQuantity = (productId) => {
    const productIndex = props.isAdded.findIndex(product => product._id === productId);
    if (productIndex !== -1) {
      const updatedProduct = { ...props.isAdded[productIndex], qty: props.isAdded[productIndex].qty + 1 };
      emit('updateIsAddedIndex', { index: productIndex, product: updatedProduct });
    }
  };

  const calculateTotalQuantity = () => {
  return props.isAdded.reduce((total, product) => {
    return total + product.qty;
  }, 0);
};

  const calculateTotalPrice = () => {
    return props.isAdded.reduce((total, product) => {
      return total + (product.qty * product.price);
    }, 0);
  };

  const checkOut = async () => {
  try {
    postLoading.value = true;
    const totalPrice = calculateTotalPrice(); 
    const totalQty = calculateTotalQuantity(); 
    const response = await axios.post('http://localhost:3000/orders', {
      products: props.isAdded.map(item => ({ product: item._id, qty: item.qty })),
      session_token: Cookies.get('session_token') ?? 'abcd',
      total_price: totalPrice,
      total_quantity: totalQty,
    }, {
      headers: {
        'Content-Type': 'application/json',
      }
    });

    if (response.status !== 200) {
      throw new Error('Checkout failed');
    } 
    emit('updateIsAdded', [])
    emit('updateCount', 0);
    emit('updateCheckSuccess', true);
    emit('closeCart', true);
  } catch (error) {
    console.error('Error during checkout:', error.message);
  } finally {
    postLoading.value = false;
  }
};


  const closeCartModal = (status) => {
    emit('updateCart', status);
  };
  </script>