<script setup>
import {ref,reactive, computed} from 'vue'

//在 <script setup> 中，任何以 v 开头的驼峰式命名的变量都可以被用作一个自定义指令。
const vClickOutside = {
  mounted: (el) => {
    document.addEventListener('click',(e) => handleFn(e,el),false)
  }
}
const handleFn = (e,el) => {
    e.stopPropagation();
    const box = document.querySelector('.box')
    if(!box){
      return
    }
    if(!el.contains(e.target) && !box.contains(e.target)){
      visible.value = false
    }
}
const getDateString = function(date){
  const yyyy = date.getFullYear()
  const mm = date.getMonth()+1
  const dd = date.getDate()
  return yyyy + '-' + (mm<10?'0'+mm:mm) + '-' + (dd<10?'0'+dd:dd)
}
const prevDate = function(date, amount = 1) {
  return new Date(date.getFullYear(), date.getMonth(), date.getDate() - amount);
};
// 获取指定月份有多少天
const getDayCountOfMonth = function(year, month) {
  if (month === 3 || month === 5 || month === 8 || month === 10) {
    return 30;
  }
  if (month === 1) {
    if (year % 4 === 0 && year % 100 !== 0 || year % 400 === 0) {
      return 29;
    } else {
      return 28;
    }
  }
  return 31;
};
// 获取指定月份的第一天
const getFirstDayOfMonth = function(date){
  return new Date(date.getFullYear(), date.getMonth(), 1)
}
// 获取指定月份第一天是星期几
const getDay = function(date){
  const newDate =  new Date(date.getFullYear(), date.getMonth(), 1);
  return newDate.getDay()
}

const cDate = ref(getDateString(new Date()))
const weeks = reactive(['日','一','二','三','四','五','六'])
const dateInCurrentMonth = function(date){
  //return date.getMonth()===new Date(cDate.value).getMonth()
  return date.getMonth()===month.value
}
const dateIsCurrentDate = function(date){
  return date.getDate()===new Date().getDate()
}
const selectedDate = ref(getDateString(new Date()))
const dateIsSelected = function(date){
  return getDateString(date) === selectedDate.value
}
const holidayList = ['2023-04-30','2023-05-01','2023-05-02','2023-05-03','2023-05-04','2023-09-10','2023-09-11','2023-09-12',]
const dateIsHoliday = function(date){
  return holidayList.includes(getDateString(date))
}
const chooseDate = function(e,date){
  e.stopPropagation();
  cDate.value = getDateString(date)
  selectedDate.value = getDateString(date)
  visible.value =false
}
const visible = ref(true)
const year = ref(new Date().getFullYear())
const month = ref(new Date().getMonth())
const changeYear = function(type){
  if(type==='add'){
    if(currentView.value==='year'){
      year.value+=10
    }else{
      year.value++
    }
  }else{
    if(currentView.value==='year'){
      year.value-=10
    }else{
      year.value--
    }
  }
}
const changeMonth = function(type){
  if(type==='add'){
    if(month.value>=11){
      year.value++
      month.value = 0
    }else{
      month.value++
    }
  }else{
    if(month.value<1){
      year.value--
      month.value = 11
    }else{
      month.value--
    }
  }
}
const list = computed(() =>{
  // 获取指定年月
  const date = new Date(year.value,month.value,new Date(cDate.value).getDate())
  //获取月份第一天是星期几，那么日期面板中前面就需要填充多少天
  const week = getDay(date)
  const dayCountOfMonth = getDayCountOfMonth(date.getFullYear(),date.getMonth())
  const arr = []
  for(let i=week;i>0;i--){
    arr.push(prevDate(getFirstDayOfMonth(date),i))
  }
  for(let i=1;i<=dayCountOfMonth;i++){
    const newDate = new Date(date.getFullYear(),date.getMonth(),i)
    arr.push(newDate)
  }
  for(let i=1;i<=42-week-dayCountOfMonth;i++){
    const newDate = new Date(date.getFullYear(),date.getMonth()+1,i)
    arr.push(newDate)
  }
  return arr
})

const currentView = ref('date')
const yearLabel = computed(() =>{
  if(currentView.value==='year'){
    const startYear = Math.floor(year.value/10) * 10
    return startYear + ' 年 - ' + (startYear+9) + ' 年 '
  }
  return year.value + ' 年 '
})
const yearData = computed(() =>{
  const startYear = Math.floor(year.value/10) * 10
  let arr = [startYear]
  while (arr.length<10) {
    arr.push(startYear+arr.length)
  }
  return arr
})
const monthData = reactive(['一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'])

const selectYear = function(){
  currentView.value = 'year'
}
const selectMonth = function(){
  currentView.value = 'month'
}

const chooseMonth = function(val){
  const index = monthData.indexOf(val)
  month.value = index
  currentView.value = 'date'
}

const chooseYear = function(val){
  year.value = val
  currentView.value = 'month'
}

</script>

<template>
  <div class="inputBox" v-click-outside>
    <span class="prefix">☺</span>
    <input v-model="cDate" type="text" class="input" @focus="visible = true;currentView = 'date'" />
  </div>
  <div class="box" v-if="visible">
    <div class="header">
      <span class="arrow" @click="changeYear('reduce')">«</span>
      <span class="arrow" @click="changeMonth('reduce')" v-show="currentView === 'date'">‹</span>
      <div>
        <span class="headerLabel" @click="selectYear">{{yearLabel}}</span>
        <span class="headerLabel" @click="selectMonth" v-show="currentView === 'date'">{{(month+1) + ' 月'}}</span>
      </div>
      <span class="arrow" @click="changeMonth('add')" v-show="currentView === 'date'">›</span>
      <span class="arrow" @click="changeYear('add')">»</span>
    </div>
    <div class="week">
      <div v-for="item in weeks" :key="item">{{item}}</div>
    </div>
    <div class="yearBody" v-show="currentView === 'year'">
      <div v-for="item in yearData" :key="item" @click="chooseYear(item)">{{item}}</div>
    </div>
    <div class="monthBody" v-show="currentView === 'month'">
      <div v-for="item in monthData" :key="item" @click="chooseMonth(item)">{{item}}</div>
    </div>
    <div class="dateBody" v-show="currentView === 'date'">
      <div v-for="i in 6" class="dateRow">  
        <div v-for="j in 7" class="dateCell">
          <span
            class="number"
            :class="[
              {'notCurrentMonth':!dateInCurrentMonth(list[i>1?(i-1)*7+j-1:j-1])},
              {'currentDate':dateIsCurrentDate(list[i>1?(i-1)*7+j-1:j-1])},
              {'selected':dateIsSelected(list[i>1?(i-1)*7+j-1:j-1])},
              {'isHoliday':dateIsHoliday(list[i>1?(i-1)*7+j-1:j-1])},
            ]"
            @click="chooseDate($event,list[i>1?(i-1)*7+j-1:j-1])"
          >
            {{list[i>1?(i-1)*7+j-1:j-1].getDate()}}
          </span>
        </div>
      </div>
    </div>
    <div class="beforeEle"></div>
  </div>
</template>

<style scoped>
  .inputBox{
    width: 220px;
    position: relative;
    margin-left:auto;
    margin-right:auto;
    .prefix{
      position: absolute;
      left: 5px;
      top: 0;
      color: #c0c4cc;
      height: 100%;
      text-align: center;
      width: 25px;
      line-height: 34px;
      font-size: 26px;
      color:#409eff;
    }
    .input{
      font-size: 14px;
      background-color: #fff;
      background-image: none;
      border-radius: 4px;
      border: 1px solid #dcdfe6;
      box-sizing: border-box;
      color: #606266;
      display: inline-block;
      height: 40px;
      line-height: 40px;
      outline: none;
      padding: 0 30px;
      transition: border-color .2s cubic-bezier(.645,.045,.355,1);
      width: 100%;
    }
  }
  .box{
    position: relative;
    margin-left:auto;
    margin-right:auto;
    margin-top: 12px;
    border-radius: 4px;
    border: 1px solid #e4e7ed;
    box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
    box-sizing: border-box;
    padding: 10px;
    width:322px;
    user-select: none;
    color: #606266;
    font-family: Helvetica Neue,Helvetica,PingFang SC,Hiragino Sans GB,Microsoft YaHei,SimSun,sans-serif;
    .header{
      display:flex;
      justify-content:space-between;
      font-size: 16px;
      font-weight: 500;
      margin-bottom:15px;
      .arrow{
        cursor: pointer;
        line-height: 20px;
        font-size: 24px;
        padding: 0 10px;
      }
      .headerLabel{
        cursor: pointer;
        &:hover{
          color:#409eff;
        }
      }
    }
    .week{
      display:flex;
      justify-content:space-around;
      padding:12px 0;
      font-size: 14px;
      border-bottom: 1px solid #ebeef5;
    }
    .dateBody{
      font-size: 14px;
      .dateRow{
        display: flex;
        .dateCell{
          width:13%;
          aspect-ratio: 1;
          line-height: 2.4;
          text-align: center;
          padding:4px 4px;
          cursor: pointer;
          position: relative;
          .number{
            width: 24px;
            height: 24px;
            display: block;
            line-height: 24px;
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%,-50%);
            border-radius: 50%;
            &:hover{
              color:#fff;
              background-color: #409eff;
            }
          }
          .number.notCurrentMonth{
            color:#c0c4cc;
          }
          .number.currentDate{
            color:#409eff;
          }
          .number.isHoliday{
            background: rgba(235,51,51,.05);
            color: #eb3333;
            position: relative;
            &::before{
              content: '休';
              position: absolute;
              top:-6px;
              right:-8px;
              color: #fff;
              background: #eb3333;
              width: 16px;
              height:14px;
              line-height: 1.2;
              font-size: 12px;
              border-radius: 6px;
            }
          }
          .number.selected{
            color: #fff;
            background-color: #409eff;
          }
        }
      }
    }
    .yearBody,.monthBody{
      display: flex;
      flex-wrap: wrap;
      & > div{
        width:calc(100% / 4);
        height:48px;
        font-size: 12px;
        text-align: center;
        line-height: 48px;
        cursor: pointer;
        &:hover{
          color:#409eff;
        }
      }
    }
    .beforeEle{
      position: absolute;
      display: block;
      width: 0;
      height: 0;
      top: -6px;
      left: 35px;
      border-color: transparent;
      border-style: solid;
      border-width: 6px;
      border-top-width: 0;
      border-bottom-color: #fff;
    }
  }
</style>
