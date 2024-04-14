<template>
    <div class="flex items-center justify-center h-screen">
      <div class="text-[#40b783] flex items-center">
        <span class="text-xl tracking-wider">Loading</span>
        <div class="lds-dots mt-2">
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'LoadingScreen'
  }
  </script>
  
  <style scoped>
  .lds-dots {
    display: inline-block;
    position: relative;
    width: 80px;
    height: 10px;
  }
  .lds-dots div {
    position: absolute;
    top: 0;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #40b783;
    animation-timing-function: cubic-bezier(0, 1, 1, 0);
  }
  .lds-dots div:nth-child(1) {
    left: 8px;
    animation: lds-dots1 2s infinite;
  }
  .lds-dots div:nth-child(2) {
    left: 8px;
    background-color: #33485e;
    animation: lds-dots2 2s infinite;
  }
  .lds-dots div:nth-child(3) {
    left: 32px;
    animation: lds-dots2 2s infinite;
  }
  @keyframes lds-dots1 {
    0%,
    100% {
      transform: scale(0);
    }
    50% {
      transform: scale(1);
    }
  }
  @keyframes lds-dots2 {
    0%,
    100% {
      transform: translateX(0);
    }
    50% {
      transform: translateX(24px);
    }
  }
  </style>
  