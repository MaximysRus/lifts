<template lang="ru">
    <div class="liftShaft">
        <div :class="lift.stopped ? 'lift disabled' :'lift'" ref="lift">
            <div :class="lift.disabled ? 'textBlock activated': 'textBlock'">
                {{lift.direction + lift.currentFloor}}
            </div>
        </div>
        <div v-for="i in floor" :key="i" class="single-floor"></div>
    </div>
</template>
<script>

import { getFloorDifference } from '../helpers/counters.js';

export default {
    name: 'LiftShaft',
    props: ['floor', 'lift', 'changeLiftFloor', 'stopLift', 'setLiftDisable'],
    watch: { 
        'lift.liftQueue': function() { 
            this.reDraw();
        },
        'lift.stopped' : function() {
            this.reDraw();
        }
    },
    mounted() {
        this.startPosition(this.lift);
    },
    methods: {
        startPosition(lift) {
            this.$refs.lift.style.bottom = (lift.currentFloor-1)*100+"px";
            if (lift.stopped==true) {
                this.$emit('stopLift', lift);
            }
            if (lift.disabled) {
                lift.disabled = false;
            }
            if (lift.liftQueue.length) {
                this.reDraw();
            }  
        }, 
        reDraw: function() {
            if (this.lift.liftQueue.length && !this.lift.disabled && !this.lift.stopped) {
                this.$emit('setLiftDisable', this.lift, true );
                const floorDifference = getFloorDifference(this.lift.liftQueue[0], this.lift);
                this.$refs.lift.style.bottom = (this.lift.liftQueue[0]-1)*100+"px";
                this.$refs.lift.style.transitionDuration = floorDifference + "s";
                this.$emit('changeLiftFloor', this.lift, this.lift.liftQueue[0]);

                setTimeout(()=>{
                    this.$emit('stopLift', this.lift);
                }, floorDifference*1000);
            }
            
        }
    },

}


</script>
<style lang="scss">

.textBlock {
    margin: 5px 30px 5px 30px;
    display: none;
    background-color: rgb(134, 121, 103);
    color: aliceblue;
    border-radius: 15px;
    &.activated {
        display: flex;
        justify-content: center;
        align-items: center;
    }
}

.liftShaft {
    margin: 0px 10px 0px 0px;
    border-left: 3px solid grey;
    border-right: 3px solid grey;
    width: 100px;
    position: relative;
}

.single-floor{
    height: 99px;
    width: 100%;
    border-bottom: 1px solid grey;
    &:first-child{
        border-top: 1px solid grey;
    }
}

.lift {
    width: 100px;
    height: 100px;
    max-width: 100px;
    max-height: 100px;
    bottom: 0px;
    left: 0px;
    position: absolute;
    background-color: aqua;
    display: block;
    &.disabled {
        animation-name: blinker;
        animation-iteration-count: infinite;
        animation-timing-function: cubic-bezier(0.5,0,0,0.5);
        animation-duration: 0.5s;
    }
}
 
@keyframes blinker {
  from { opacity: 1.0; }
  to { opacity: 0.2; }
}
 
</style>