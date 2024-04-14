<template>
    <header class="bg-white">
      <nav class="mx-auto flex max-w-7xl items-center justify-between p-6 lg:px-8" aria-label="Global">
        <div class="flex lg:flex-1">
          <a href="/" class="-m-1.5 p-1.5">
            <span class="sr-only">Your Company</span>
            <img class="h-8 w-auto" src="../assets/logo.png" alt="" />
          </a>
        </div>
        <div class="flex lg:hidden">
          <button type="button" class="-m-2.5 inline-flex items-center justify-center rounded-md p-2.5 text-gray-700" @click="mobileMenuOpen = true">
            <span class="sr-only">Open main menu</span>
            <Bars3Icon class="h-6 w-6" aria-hidden="true" />
          </button>
        </div>

        <div @click="pageTranition(false)" class="hidden lg:flex lg:flex-2 lg:justify-end relative">
            <div class="mx-3">
              <HomeIcon class="w-6 h-6 text-[#40b783]" />
            </div>
        </div>
        <div @click="pageTranition(true)" class="hidden lg:flex lg:flex-2 lg:justify-end relative">
            <div class="mx-3">
              <AdjustmentsVerticalIcon class="w-6 h-6 text-[#40b783]" />
            </div>
        </div>
        <div @click="goToCart" class="hidden lg:flex lg:flex-2 lg:justify-end relative">
            <div class="mx-3">
              <ShoppingBagIcon class="w-6 h-6 text-[#40b783]" />
            </div>
            <div class="absolute bottom-2 flex items-center justify-center w-5 h-5 bg-[#ff0000] rounded-full text-white">
                <span class="text-xs font-semibold">{{ props.count  }}</span>
            </div>
        </div>
      </nav>
      <Dialog as="div" class="lg:hidden" @close="mobileMenuOpen = false" :open="mobileMenuOpen">
        <div class="fixed inset-0 z-10" />
        <DialogPanel class="fixed inset-y-0 right-0 z-10 w-full overflow-y-auto bg-white px-6 py-6 sm:max-w-sm sm:ring-1 sm:ring-gray-900/10">
          <div class="flex items-center justify-between">
            <a href="#" class="-m-1.5 p-1.5">
              <span class="sr-only">Your Company</span>
              <img class="h-8 w-auto" src="https://tailwindui.com/img/logos/mark.svg?color=indigo&shade=600" alt="" />
            </a>
            <button type="button" class="-m-2.5 rounded-md p-2.5 text-gray-700" @click="mobileMenuOpen = false">
              <span class="sr-only">Close menu</span>
              <XMarkIcon class="h-6 w-6" aria-hidden="true" />
            </button>
          </div>
          <div class="mt-6 flow-root">
            <div class="-my-6 divide-y divide-gray-500/10">
              <div class="py-6">
                <a href="#" class="-mx-3 block rounded-lg px-3 py-2.5 text-base font-semibold leading-7 text-gray-900 hover:bg-gray-50" @click="goToCart">Log in</a>
              </div>
            </div>
          </div>
        </DialogPanel>
      </Dialog>
    </header>
  </template>
  
  <script setup>
  import { ref, defineEmits, defineProps } from 'vue'
  import { ShoppingBagIcon } from "@heroicons/vue/24/solid"
  import { HomeIcon } from "@heroicons/vue/24/solid"
  import { AdjustmentsVerticalIcon } from "@heroicons/vue/24/solid"
  
  import {
    Dialog,
    DialogPanel,
  } from '@headlessui/vue'
  import {
    Bars3Icon,
    XMarkIcon,
  } from '@heroicons/vue/24/outline'
  
  const mobileMenuOpen = ref(false)
  const props = defineProps({
    count: {
      type: Number,
      default: 0
    }
  });
  const emit = defineEmits(['go-to-cart', 'count', 'orderPageTransition']);

  const goToCart = () => {
    emit('go-to-cart', true);
  };

  const pageTranition = (status) => {
    emit('orderPageTransition', status)
  }

  </script>