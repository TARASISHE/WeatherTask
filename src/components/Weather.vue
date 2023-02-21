<template>
<h1 class="text-3xl text-center mb-10 p-4">
    Weather App
</h1>
<div class="relative">
    <input type="text"
    class="search-bar"
    placeholder="Search..."
    v-model="query"
    @input="findCity()">
    <ul class="absolute top-[60px] left-5 z-10 w-1/3 border border-[#B2B1CF] xs:w-1/2 " 
        v-if="searchCityPanel">
            <li
              class="p-3 text-semibold text-lg cursor-pointer hover:bg-[#98D2EB]"
              v-for="city in cities" 
              :key="city.id" 
              @click="appendCity(city)"
            >
                   {{ city.name }}
            </li>  
    </ul>
</div>
<Spinner v-if="spinner"/>
<div v-else>
    <div class="flex items-center justify-center">
        <select 
        v-if="sortedWeatherItemList.length" 
        v-model="selectedData" 
        class="mt-10 border w-36 rounded border-[#49392C]">
            <option  
            v-for="(day, index) in days" 
            :key="day.id" 
            :value="index">
                    {{ day }}
            </option>
        </select>
    </div>
    <div  class="flex flex-col items-center justify-center">
    <div v-if="sortedWeatherItemList.length" 
         class="flex mx-3 mt-6">
        <div 
        @click="sortBy('city')" 
        class="p-2 flex items-center justify-center w-52 h-16 border-y-2 border-l-2 border-black font-semibold md:w-36 sm:w-36 xs:w-24">
              City
        </div>
        <div 
        @click="sortBy('min')" 
        class="p-2 flex items-center justify-center w-52  h-16 border-2 border-black  font-semibold md:w-36 sm:w-36 xs:w-24">
        MinTemp
        </div>
        <div 
        @click="sortBy('max')" 
        class="p-2 flex items-center justify-center w-52  h-16 border-y-2 border-r-2 border-black font-semibold md:w-36 sm:w-36 xs:w-24">
        MaxTemp
        </div>
    </div>
    <TransitionGroup 
    name="list" 
    tag="div">  
    <div  
        v-for="oneCity in sortedWeatherItemList" 
        :key="oneCity.id" 
        class=" mx-3  flex text-base">
            <div 
            class=" relative flex flex-row items-center justify-center w-52 h-16 border-b-2 border-l-2 border-black text-base md:w-36 sm:w-36 xs:w-24">
                {{ oneCity.city }} 
                <img 
                @click="deleteCityFromTable(oneCity)" 
                class="absolute top-2 left-2 w-3 h-3" 
                src="../assets/img/delete.png" alt="">
            </div>
            <div 
            class="flex flex-row items-center justify-center w-52 h-16 border-b-2 border-l-2 border-black text-base md:w-36 sm:w-36 xs:w-24">
                {{ oneCity.min[selectedData] }} 
            </div>
            <div 
            class="flex flex-row items-center justify-center w-52 h-16 border-l-2 border-b-2 border-r-2 border-black text-base md:w-36 sm:w-36 xs:w-24">
                {{ oneCity.max[selectedData] }}   
            </div>
    </div>
    </TransitionGroup>
    </div>
</div>



</template>


<script setup>
//imports
import Spinner from '../components/Spinner.vue'
import { onMounted, ref, computed } from 'vue'
//imports

// date for change the day of forecast
const selectedData = ref(0)
// date for change the day of forecast

// start find city logic
const spinner = ref(false)
const searchCityPanel = ref(false)

const findCity = async () =>{
 try{
   if(query.value === ''){
    spinner.value = false
   } else {
    spinner.value =true
   }
    searchCityPanel.value = true
    const response = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${query.value}`);
    const data = await response.json();
    cities.value = data.results
   }catch(err){
    console.log(err)
  } 
}
// end find city logic


// start append city in table
const query = ref('')
const cities = ref({})
const weatherItemList = ref([])
const latitude = ref(0)
const longitude = ref(0)

const appendCity = (city)=> {
  searchCityPanel.value = true


  query.value = city;
  weatherItemList.value.push({
     id: city.id,
     city: city.name,
     min:[],
     max:[]
  })


  latitude.value = city.latitude;
  longitude.value = city.latitude;
  fetchWeather() 

  spinner.value = false
  searchCityPanel.value = false
  query.value = '';
  localStorage.setItem('weatherItemList', JSON.stringify(weatherItemList.value))
}
// end append city in table



// start fetch weather and get day 
const days = ref([])
const weather = ref({})
const fetchWeather = async () =>{
 try{ 
    const response = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&hourly=temperature_2m&daily=temperature_2m_max,temperature_2m_min&current_weather=true&timezone=Europe%2FBerlin`);
    const data = await response.json();
    weather.value = data
    days.value = weather.value.daily.time
    localStorage.setItem('days', JSON.stringify(days.value))


    weatherItemList.value[weatherItemList.value.length - 1].min = weather.value.daily.temperature_2m_min
    weatherItemList.value[weatherItemList.value.length - 1].max = weather.value.daily.temperature_2m_max
    localStorage.setItem('weatherItemList', JSON.stringify(weatherItemList.value))


  }catch(err){
    console.log(err)
  }
}
// end fetch weather and get day 

// start delete the item from table
const deleteCityFromTable = (ind ) => {
    let index = weatherItemList.value.findIndex((el) => el === ind)
    weatherItemList.value.splice(index , 1)
    localStorage.setItem('weatherItemList', JSON.stringify(weatherItemList.value))
}
// end delete the item from table


// start sort the table
const sortedColumn = ref('city')
const sortAsc = ref(1)

const sortedWeatherItemList = computed(()=>{
     return weatherItemList.value.sort((a,b)=>{
        if (a[sortedColumn.value] > b[sortedColumn.value]) return 1 * sortAsc.value;
        if (a[sortedColumn.value] < b[sortedColumn.value]) return -1 * sortAsc.value;
     })
      
})

const sortBy = (arg) =>{
    sortedColumn.value = arg
    if(sortedColumn.value === arg){
      sortAsc.value *= -1
    } else{
      sortAsc.value = 1
      sortedColumn.value = arg
    }
}

//end sort the table


// get a data on first render
onMounted(()=>{
    const data = localStorage.getItem('weatherItemList');
        if(data){
           weatherItemList.value = JSON.parse(data);
        }

    const dayData = localStorage.getItem('days');
        if(dayData){
           days.value = JSON.parse(dayData);
        }
})
// get a data on first render

</script>


<style scoped> 
.search-bar {
  display: block;
  width: 90%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 16px 0px 16px 0px;
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
  margin-left: 20px;
}
.search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.7s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.list-leave-active {
  position: absolute;
}

@media  screen and (max-width: 470px){
  .spinner {
  position: relative;
  top: -55px;
  left: 88%;
  margin: 0 0 0 -25px;
}
}
</style>