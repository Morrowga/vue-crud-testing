<template>
  <template v-if="!isLoading">
    <HeaderComponent @go-to-cart="showCart = true" :count="count" @orderPageTransition="orderPageTransition"  />
    <template v-if="!isOrderPage">
      <ProductHome :isAdded="isAdded" @updateIsAdded="updateIsAdded" :count="count" @count="updateCount" />
    </template>
    <template v-else>
      <OrderPage :isCancel="isCancel" @updateReloadingOrder="updateReloadingOrder" :isReloadOrderPage="isReloadOrderPage" @updateCancelStatus="updateCancelStatus" @updateConfirmDialogStatus="updateConfirmDialogStatus" @updateOrderDetail="updateOrderDetail" @updateDetailCart="updateDetailCart"/>
      <OrderDetailComponent :orderDetail="orderDetail" :showDetailCart="showDetailCart" @updateConfirmDialogStatus="updateConfirmDialogStatus" @updateDetailCart="updateDetailCart" />
      <ConfirmDialog @updateCancelStatus="updateCancelStatus" @updateConfirmDialogStatus="updateConfirmDialogStatus" :isConfirm="isConfirm" />
    </template> 
    <CartComponent :showCart="showCart" @updateCount="updateCount" @updateCheckSuccess="updateCheckSuccess" @updateIsAdded="updateIsAdded" @updateIsAddedIndex="updateIsAddedIndex" :isAdded="isAdded" @updateCart="updateCart" />
    <SuccessDialog :isOrderPage="isOrderPage" @orderPageTransition="orderPageTransition" :checkSuccess="checkSuccess" @updateCheckSuccess="updateCheckSuccess" />
    <WarningDialog :isOpenWarningDialog="isOpenWarningDialog" @updateWarningDialog="updateWarningDialog"/>
  </template>
  <template v-else>
    <LoadingScreenComponent />
  </template>
</template>


<script setup>
  import { ref, onMounted } from 'vue';
  import LoadingScreenComponent from './components/LoadingScreenComponent.vue'
  import ProductHome from './pages/ProductHome.vue'
  import OrderPage from './pages/OrderPage.vue'
  import HeaderComponent from './components/HeaderComponent.vue'
  import CartComponent from './components/CartComponent.vue'
  import SuccessDialog from './components/SuccessDialogComponent.vue'
  import ConfirmDialog from './components/ConfirmDialogComponent.vue'
  import WarningDialog from './components/WarningDialogComponent.vue'
  import OrderDetailComponent from './components/OrderDetailComponent.vue'
  import { v4 as uuidv4 } from 'uuid';
  import Cookies from 'js-cookie';
  import axios from 'axios';

  const showCart = ref(false);
  const showDetailCart = ref(false);
  const orderDetail = ref([]);
  const isCancel = ref(false);
  const count = ref(0);
  const isConfirm = ref(false);
  const isOrderPage = ref(false);
  const checkSuccess = ref(false);
  const isOpenWarningDialog = ref(false);
  const isAdded = ref([]);
  const isLoading = ref(true);
  const isReloadOrderPage = ref(false);
  

  const updateCount = (newCount) => {
    count.value = newCount;
  };

  const updateCancelStatus = (status) => {
      isCancel.value = status;
  } 

  const updateReloadingOrder = (status) => {
    isReloadOrderPage.value = status
  }

  const updateCart = (status) => {
    showCart.value = status
  }

  const updateConfirmDialogStatus = (status) => {
    isConfirm.value = status
  }

  const updateOrderDetail = (order) => {
    orderDetail.value = order
  }   

  const updateDetailCart = (status) => {
    showDetailCart.value = status
  }

  const orderPageTransition = (status) => {
    isOrderPage.value = status
    isReloadOrderPage.value = status;
    console.log(isReloadOrderPage.value);
  }

  const updateCheckSuccess = (status) => {
    checkSuccess.value = status;
  };

  const updateIsAdded = (updatedCart) => {
    isAdded.value = updatedCart;
  };

  const updateIsAddedIndex= ({ index, product }) => {
    isAdded.value.splice(index, 1, product); // Update isAdded array
    console.log(isAdded.value)
  };

  const updateWarningDialog = (status) => {
    isOpenWarningDialog.value = status
    Cookies.remove('session_token');
    Cookies.remove('expiration_time');
    window.location.reload()
  }

  const generateSessionToken = () => {
    const sessionToken = Cookies.get('session_token');
    if (!sessionToken) {
      const newSessionToken = uuidv4();
      const expirationDate = new Date();
      expirationDate.setTime(expirationDate.getTime() + (1 * 60 * 60 * 1000));
      const expirationTime = expirationDate.toUTCString(); 
      Cookies.set('session_token', newSessionToken); 
      Cookies.set('expiration_time', expirationTime);
    }
  };

  const isSessionTokenExpired = () => {
    const expirationTime = Cookies.get('expiration_time');
    if (!expirationTime) {
      return true;
    }
    return new Date(expirationTime) < new Date();
  };

  const deleteAllOrders = async (sessionToken) => {
    try {
      const response = await axios.delete('http://localhost:3000/delete-all-orders?session_token=' + sessionToken);
      console.log('Orders deleted successfully:', response.data);
    } catch (error) {
      console.error('Error deleting orders:', error.message);
    }
  };

  const handlingInterval = () => {
    const sessionToken = Cookies.get('session_token');
    if (sessionToken) {
      if (isSessionTokenExpired(sessionToken)) {
        deleteAllOrders(sessionToken);
        isOpenWarningDialog.value = true;
      }
    } else {
      generateSessionToken();
    }
  };

  onMounted(() => {

    handlingInterval();
    setInterval(handlingInterval, 60000);

    const delay = setTimeout(() => {
      isLoading.value = false;
    }, 2000); 
    return () => clearTimeout(delay);
  });

</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
