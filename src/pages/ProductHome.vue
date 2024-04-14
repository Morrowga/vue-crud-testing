<template>
  <div class="bg-white" v-if="!showSkeleton">
    <div class="mx-auto max-w-2xl px-4 sm:px-6 lg:max-w-7xl lg:px-8">
      <h2 class="text-2xl font-bold text-gray-900 tracking-wide">Our Products</h2>

      <div class="mt-6 grid grid-cols-1 gap-x-6 gap-y-10 sm:grid-cols-2 lg:grid-cols-4 xl:gap-x-8">
        <div v-for="product in products" :key="product.id" class="group relative">
          <div class="aspect-h-1 aspect-w-1 w-full overflow-hidden rounded-lg bg-gray-200 lg:aspect-none lg:h-80 shadow-xl">
            <img :src="product.image" :alt="product.image"
            class="h-full w-full object-cover object-center lg:h-full lg:w-full">
            <div class="absolute bottom-12 mb-2 right-0 bg-white bg-opacity-75 px-2 py-1 rounded-tl-md text-sm font-bold">
                $ {{ product.price }}
            </div>
          </div>
          <div class="mt-4 flex justify-between">
              <h3 class="text-xl text-000 pt-1 capitalize font-bold">
                  {{product.name}}
              </h3>
              <button @click="addToCart(product)" :class="isAdded.some(item => item._id === product._id) ? 'bg-[#ff0000] p-2 w-100 rounded text-sm px-3' : 'bg-[#40b783] text-[#fff] p-2 w-100 rounded text-sm px-3'">
                  <div v-if="isAdded.some(item => item._id === product._id)">
                    <ArchiveBoxXMarkIcon class="w-6 h-6" />
                  </div>
                  <div v-else>
                      <ShoppingCartIcon class="w-6 h-6" />
                  </div>
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
import { onMounted, ref,defineEmits, defineProps } from 'vue';
import { Skeleton } from 'vue-loading-skeleton';
import { ArchiveBoxXMarkIcon, ShoppingCartIcon } from "@heroicons/vue/24/outline"
import "vue-loading-skeleton/dist/style.css"

const products = ref([]);
const showSkeleton = ref(true);
const skeletons = Array.from({ length: 4 });
const emit = defineEmits(['go-to-cart', 'count', 'updateIsAdded']);
const props = defineProps({
  count: {
    type: Number,
    default: 0 
  },
  isAdded: {
    type: Array,
    default: null
  }
});

const addToCart = (product) => {
  const updatedCart = [...props.isAdded];

  const index = updatedCart.findIndex(item => item._id === product._id);

  if (index !== -1) {
    updatedCart.splice(index, 1);
    decreaseCount();
  } else {
    const productWithQty = { ...product, qty: 1 };
    updatedCart.push(productWithQty);
    increaseCount();
  }

  emit('updateIsAdded', updatedCart)
};

const increaseCount = () => {
  let count = props.count
  emit('count', count + 1);
};

const decreaseCount = () => {
  let count = props.count;
  emit('count', count > 0 ? count - 1 : 0);
};



const getProducts = () => {
  axios.get('http://localhost:3000/products')
  .then(response => {
    products.value = response.data
    showSkeleton.value = false;
  })
  .catch(error => {
    console.error('Error fetching data:', error);
  });
}

onMounted(() => {
  getProducts();
});

</script>