<script>
import { ref, onMounted, watch } from "vue";
import axios from "@/config/axios/index.js";
export default {
  setup() {
    const gameStarted=ref(false)
    const disableGame=ref(false)
    const notEnough=ref(false)
    const dataIsFetched=ref(false)
    const balance=ref(0)
    const value=ref('?')


    onMounted(async()=>{
       const res=await axios.get('user')
       balance.value=res.data.balance
       dataIsFetched.value=true
    })

    watch(balance, (currentValue, oldValue) => {
      if(oldValue!=0){
        axios.post('update-balance', {
          balance: currentValue
        })
        .then((res)=>{
          console.log(res)
        })
      }
   });


    function play(){
      let min = 1;
      let max = 10;
      let random = Math.floor(Math.random() * (max - min + 1)) + min;
      value.value=random
      if(random==7){
        balance.value=2*(balance.value)
      }else{
        balance.value=balance.value-5
      }
      disableGame.value=true
    }

    function enableGame(){
      disableGame.value=false
      value.value='?'
      if(balance.value<5){
       notEnough.value=true
      }
    }

    function addBalance(){
      notEnough.value=false
      window.location.href=import.meta.env.VITE_API_BASE_URL+'create-checkout-session';

    }
   

    return {gameStarted, balance, value, play, disableGame, enableGame, notEnough, addBalance, dataIsFetched}
  },
}
</script>

<template>
  <main v-if="dataIsFetched" class="w-[100vw] h-[100vh] bg-gray-600 flex items-center justify-center">
     <div v-if="gameStarted"  class="absolute right-0 top-0 -translate-x-[7rem] translate-y-[2rem] text-[#fff] text-[2.4rem]">Balance: $<span>{{ balance }}</span></div>
     <div @click="addBalance" v-if="gameStarted"  class="absolute right-0 top-0 -translate-x-[7rem] translate-y-[6rem] text-[#fff] text-[1.6rem] bg-gray-900 px-[1rem] py-[0.6rem] rounded-[7px] cursor-pointer hover:bg-gray-800">Add balance</div>
      <div class="flex flex-col items-center justify-center mb-[5rem]">
      <div @click="gameStarted=true" :class="[gameStarted ? 'fadeaway' : '']" class="text-[#fff] text-[3rem] px-[3rem] py-[2rem] bg-[#002] cursor-pointer rounded-[9px] opacity-1">Start Game</div>
        <p v-if="notEnough" class="text-[3rem] font-[600] text-red-500 mb-[5rem] text-center">Not enough balance! Deposit to continue</p>
        <p @click="enableGame" :class="[gameStarted ? 'fadesymbols' : '']" class="opacity-0 text-[1.6rem] text-gray-300 mb-[2rem] px-[1rem] py-[0.6rem] text-center bg-gray-700 cursor-pointer rounded-[7px]">RELOAD</p>
      <div @click="play" :class="[gameStarted ? 'fadein' : '', balance < 5 ? 'notEnoughBalance' : '', disableGame ? 'noPointer' : '']" class="w-[10rem] h-[0rem] bg-[#693] opacity-0 pointer-events-none flex items-center justify-center rounded-[12px] cursor-pointer">
        <div>
          <p class="text-[7rem] opacity-0 pointer-events-none" :class="[gameStarted ? 'fadesymbols' : '']">{{ value }}</p>
        </div>
        <p :class="[gameStarted ? 'fadesymbols' : '']" class="opacity-0 pointer-events-none absolute bottom-[-5rem] left-1/2 -translate-x-1/2 text-[1.6rem] text-gray-300 w-[100%] text-center pointer-events-none">Click on to play!</p>
      </div>
      </div>
  </main>
</template>

<style scoped>
div{
  transition: 0.5s all ease-out;
}
.fadeaway{
  opacity: 0;
  pointer-events: none;
  }
.fadein{
  opacity: 1;
  pointer-events: all;
  height: 10rem;
  }
.fadesymbols{
  opacity: 1;
  }
  .notEnoughBalance{
    opacity: 0.5 !important;
    pointer-events: none !important;
  }
  .noPointer{
    pointer-events: none !important;
  }
</style>
