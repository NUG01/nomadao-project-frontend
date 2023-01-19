<script>
import { ref, onMounted, watch } from "vue";
import axios from "@/config/axios/index.js";
import { Form } from 'vee-validate';

export default {
  components:{Form},
  setup() {
    const gameStarted=ref(false)
    const disableGame=ref(false)
    const notEnough=ref(false)
    const dataIsFetched=ref(false)
    const balanceFormActivated=ref(false)
    const withdrawFormActivated=ref(false)
    const unableWithdraw=ref(false)
    const rules=ref(false)
    const balance=ref(0)
    const bet=ref(5)
    const amount=ref('')
    const amountWithdraw=ref('')
    const value=ref('?')
    const numberRegex=/^[0-9]{1,}$/



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
      notEnough.value=false
      let min = 1;
      let max = 10;
      let random = Math.floor(Math.random() * (max - min + 1)) + min;
      if(random==7 && (balance.value-bet.value)>=0){
        value.value=random
        balance.value=(bet.value*7)+(balance.value)
      }else if((balance.value-bet.value)>=0){
        value.value=random
        balance.value=balance.value-bet.value
      }else{
        notEnough.value=true
      }
      disableGame.value=true
    }

    function enableGame(){
      disableGame.value=false
      notEnough.value=false
      value.value='?'
      if(balance.value<bet.value){
       notEnough.value=true
      }
    }

 

   async function addBalance(){
     notEnough.value=false
    if(!(amount.value).match(numberRegex)){
     return
    }
    const res= await axios.post('create-checkout-session', {
      amount: amount.value
      })
      window.location.href=res.data;
    }



   async function withdraw(){
    unableWithdraw.value=false
    if(!(amountWithdraw.value).match(numberRegex) || amountWithdraw.value>balance.value){
      unableWithdraw.value=true
      return
    }
    const res= await axios.post('withdraw', {
      amount: amountWithdraw.value
      })
      balance.value=res.data
      withdrawFormActivated.value=false
    }


    function changeBetValue(ev){
      bet.value=Number(ev.target.innerText)
    }
   

    return {
            gameStarted, 
            balance,
            value, 
            play, 
            disableGame,
            enableGame, 
            notEnough, 
            addBalance, 
            dataIsFetched, 
            amount,
            balanceFormActivated,
            withdrawFormActivated,
            amountWithdraw,
            withdraw,
            bet,
            changeBetValue,
            unableWithdraw,
            rules
       }
  },
}
</script>

<template>
  <main v-if="dataIsFetched" class="w-[100vw] h-[100vh] bg-gray-600 flex items-center justify-center">
    <div v-if="rules" class="w-[100vw] h-[100vh] absolute top-0 left-0 flex items-center justify-center z-30">
    <div @click="rules=false" class="absolute top-0 left-0 w-[100vw] h-[100vh] bg-[#000] opacity-[0.5] z-40"></div>
      <div class="w-[50%] bg-gray-300 border-[2px] border-[#000] border-solid py-[2rem] px-[4rem] rounded-[5px] z-50">
        <ul class="flex flex-col justify-center text-[1.6rem]">
          <li>◾ You choose how much to bet from the given options ❕</li>
          <li>◾ If your number is 7, you will get bet multiplied by 7 ❕</li>
          <li>◾ If your number is not 7, you will lose your bet ❕</li>
          <li>◾ Your Game = Your Risk ❕</li>
        </ul>
      </div>
    </div>
    <div v-if="gameStarted" class="flex items-center justify-center absolute top-0 left-0 translate-x-[20%] translate-y-1/2 gap-[1rem]">
    <div @click="changeBetValue($event)" class="text-[3rem] rounded-[5px] cursor-pointer p-[1rem]" :class="[bet===5 ? 'bg-gray-500 text-[#fff]' : 'bg-gray-200 text-[#000]']" :value="5">5</div>
    <div @click="changeBetValue($event)" class="text-[3rem] rounded-[5px] cursor-pointer p-[1rem]" :class="[bet===10 ? 'bg-gray-500 text-[#fff]' : 'bg-gray-200 text-[#000]']" :value="10">10</div>
    <div @click="changeBetValue($event)" class="text-[3rem] rounded-[5px] cursor-pointer p-[1rem]" :class="[bet===20 ? 'bg-gray-500 text-[#fff]' : 'bg-gray-200 text-[#000]']" :value="20">20</div>
    <div @click="changeBetValue($event)" class="text-[3rem] rounded-[5px] cursor-pointer p-[1rem]" :class="[bet===50 ? 'bg-gray-500 text-[#fff]' : 'bg-gray-200 text-[#000]']" :value="50">50</div>
    <div @click="changeBetValue($event)" class="text-[3rem] rounded-[5px] cursor-pointer p-[1rem]" :class="[bet===100 ? 'bg-gray-500 text-[#fff]' : 'bg-gray-200 text-[#000]']" :value="100">100</div>
    </div>
<div v-if="balanceFormActivated" class="absolute top-0 left-0 w-[100vw] h-[100vh]">
  <div @click="balanceFormActivated=false" class="absolute top-0 left-0 w-[100vw] h-[100vh] bg-[#000] opacity-[0.5] z-30"></div>
<Form @submit="addBalance" class="absolute bg-gray-500 top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50 w-[30%] py-[5rem] px-[7rem] rounded-[3px] flex flex-col items-center justify-center">
  <input v-model="amount" type="text" name="amount" id="amount" class="w-[100%] h-[6.3rem] text-[2.4rem] text-[#636363] border-[#cdcdcd] focus:border-[#a7a7a7] focus:border-[1.5px] border-[1px] border-solid rounded-[3px] bg-[#f6f7f7c1] pl-[1.6rem]" />
<button type="submit" class="flex items-center justify-center bg-[#0095f6] text-[#fff] text-[2.4rem] py-[1rem] font-[500] hover:bg-[#0074cc] px-[3rem] rounded-[5px] mt-[2rem]">Deposit</button>
</Form>
</div>
<div v-if="withdrawFormActivated" class="absolute top-0 left-0 w-[100vw] h-[100vh]">
  <div @click="withdrawFormActivated=false" class="absolute top-0 left-0 w-[100vw] h-[100vh] bg-[#000] opacity-[0.5] z-30"></div>
<Form @submit="withdraw" class="absolute bg-gray-500 top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50 w-[30%] py-[5rem] px-[7rem] rounded-[3px] flex flex-col items-center justify-center">
  <input v-model="amountWithdraw" type="text" name="amount" id="amount" class="w-[100%] h-[6.3rem] text-[2.4rem] text-[#636363] border-[#cdcdcd] focus:border-[#a7a7a7] focus:border-[1.5px] border-[1px] border-solid rounded-[3px] bg-[#f6f7f7c1] pl-[1.6rem]" />
<button type="submit" class="flex items-center justify-center bg-[#0095f6] text-[#fff] text-[2.4rem] py-[1rem] font-[500] hover:bg-[#0074cc] px-[3rem] rounded-[5px] mt-[2rem]">Withdraw</button>
<div @click="unableWithdraw=false" v-if="unableWithdraw" class="text-red-700 text-[1.6rem] text-center mt-[2rem] px-[1rem] py-[0.6rem] bg-[#fff] rounded-[10px] font-500 cursor-pointer">Not enough balance!</div>
</Form>
</div>
     <div v-if="gameStarted"  class="absolute right-0 top-0 -translate-x-[7rem] translate-y-[2rem] text-[#fff] text-[2.4rem]">Balance: $<span>{{ balance }}</span></div>
     <div @click="balanceFormActivated=true" v-if="gameStarted"  class="absolute right-0 top-0 -translate-x-[7rem] translate-y-[6rem] text-[#fff] text-[1.6rem] bg-gray-900 px-[1rem] py-[0.6rem] rounded-[7px] cursor-pointer hover:bg-gray-800">Add balance</div>
     <div @click="withdrawFormActivated=true" v-if="gameStarted"  class="absolute right-0 top-0 -translate-x-[7rem] translate-y-[10.5rem] text-[#fff] text-[1.6rem] bg-gray-500 px-[1rem] py-[0.6rem] rounded-[7px] cursor-pointer hover:bg-gray-400">Withdraw</div>
      <div class="flex flex-col items-center justify-center mb-[3rem]">
      <div @click="gameStarted=true" :class="[gameStarted ? 'fadeaway' : '']" class="text-[#fff] text-[3rem] px-[3rem] py-[2rem] bg-[#002] cursor-pointer rounded-[9px] opacity-1">Start Game</div>
      <div @click="rules=true" :class="[gameStarted ? 'fadeaway' : '']" class="text-[#000] text-[2rem] px-[2rem] py-[1rem] bg-gray-400 cursor-pointer rounded-[9px] opacity-1 mt-[2rem] font-500">Rules</div>
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
