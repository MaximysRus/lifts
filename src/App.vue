<template>
  <div class = "wrapper">
    <div id = "shafts" class = "shafts">
      <LiftShaft 
        :floor="config.floorCount" 
        :lift="lift" 
        v-for = "lift in lifts" 
        :key="lift.id" 
        @changeLiftFloor="changeLiftFloor"
        @stopLift="stopLift"
        @setLiftDisable = "setLiftDisable"
      >
      </LiftShaft>
    </div>
    <div id = "buttonBox" class="buttonBox">
      <div class="floor" v-for = "floor in config.floorCount" :key="floor" :class="isActivated(getCallFloor(floor)) ? 'floor activated' :'floor'" >
        <input :id = "'button'+floor " type="radio" v-on:click="callLift(getCallFloor(floor))" name="floor"><label :for = "'button'+floor ">Этаж  {{getCallFloor(floor)}}</label>
      </div>
    </div>
  </div>
</template>

<script>
import LiftShaft from './components/LiftShaft.vue';
import config from './config.js';
import { getFloorDifference } from './helpers/counters.js';

export default {
  name: 'App',
  components: {
    LiftShaft
  },
  
  mounted() {
    this.lifts = this.getLifts();
    localStorage.setItem("config", JSON.stringify(this.config));
    window.ondestroy = () => localStorage.setItem('lifts', this.lifts)
  },

  data() {
    return {
      lifts: [],
      config,
    }
  },

  methods: {

    isActivated(floor) {
      return !!this.lifts.find((lift)=>lift.liftQueue.includes(floor));
    },

    refreshLocaleStore() {
      localStorage.setItem("lifts", JSON.stringify(this.lifts));
    },

    makeDefaultLift(id) {
      return {
        id,
        currentFloor: 1,
        liftQueue: [],
        disabled: false,
        stopped: false,
        direction: "",
      }
    },

    generateDefaultLifts() {
      return [...Array(this.config.shaftCount)].map((_, index)=>this.makeDefaultLift(index+1));
    },

    getLifts() {
      const configLocalStorge = JSON.parse(localStorage.getItem("config"));
      const liftsLocalStorage = JSON.parse(localStorage.getItem("lifts"));
      if(configLocalStorge) {
        return configLocalStorge.floorCount === this.config.floorCount && 
          configLocalStorge.shaftCount ===  this.config.shaftCount && liftsLocalStorage.length === this.config.shaftCount ? 
          liftsLocalStorage ?? this.generateDefaultLifts() : 
          this.generateDefaultLifts();
      }
      return this.generateDefaultLifts();
    },

    getCallFloor(floor) {
      return this.config.floorCount - floor + 1;
    },

    stopLift(currentLift) {
      const index = this.lifts.findIndex((lift)=> lift.id === currentLift.id);
      this.lifts[index] = {...this.lifts[index], stopped: true, disabled: false, liftQueue: this.lifts[index].liftQueue.slice(1)};
      this.refreshLocaleStore();
      setTimeout(()=>{
        this.lifts[index].stopped = false;
      }, 3000)
      
    },

    setLiftDisable(currentLift, disabled) {
      const index = this.lifts.findIndex((lift)=> lift.id===currentLift.id);
      this.lifts[index].disabled = disabled;
    },

    callLift: function(floor) {
      const causedLift = [...this.lifts].sort((firstLift, secondLift)=> {
        const firstFloorDifference = getFloorDifference(floor, firstLift);
        const secondFloorDifference = getFloorDifference(floor, secondLift);
        if(firstLift.disabled && !secondLift.disabled) {
          return 1;
        }
        if(!firstLift.disabled && secondLift.disabled) {
          return -1;
        }
        if(!firstFloorDifference){
          return 1;
        }
        if(!secondFloorDifference){
          return -1;
        }
        return firstFloorDifference - secondFloorDifference;
      }).find((lift)=>!lift.stopped)
      
      if (causedLift && !causedLift.liftQueue.includes(floor) && causedLift.currentFloor!=floor) {
        const index = this.lifts.findIndex((lift)=> lift.id===causedLift.id);
        this.lifts[index].liftQueue = [...this.lifts[index].liftQueue, floor];
      }
      this.refreshLocaleStore();
    },
    
    changeLiftFloor(currentLift, floor) {
      const index = this.lifts.findIndex((lift)=> lift.id===currentLift.id);
      this.lifts[index].direction =  this.lifts[index].currentFloor < floor ? "▲ " : "▼ ";
      this.lifts[index].currentFloor = floor;
    }
  }
}

</script>

<style lang="scss">
  .liftText{
    margin-top: 5%;
    margin-left: 33%;
    max-width: 30px;
    border-radius: 8%;
    display: grid;
    background-color: grey;
  }

  .shafts{
    display: inline-flex;
  }

  .floor{
    color: black;
    border: 3px solid aqua;
    border-radius: 10px;
    margin-top: 35px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    &.activated {
      border: 3px solid orange;
    }
  }

  .buttonBox{
    display: grid;
  }

  .wrapper{
    display: inline-flex;
  }
</style>
