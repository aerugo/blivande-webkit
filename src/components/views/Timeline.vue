<template>
  <div class="timeline_container">
  <div class="timeline"  id="events_container">
    <div class="filters w-full" v-if="filtered()">
      <div class="filter type" v-if="type" @click="type = null">
        <p class="key">type</p><p>{{type.label}}</p>
      </div>
      <div class="filter location" v-if="location" @click="location = null">
        <p class="key">location</p><p>{{location}}</p>
      </div>
      <div class="filter search" v-if="search" @click="search = ''">
        <p class="key">search</p><p>{{search}}</p>
      </div>
    </div>
    <transition-group name="list" tag="div" class="events_list">
    <div v-for="(item, index) in filteredItems" :key='index' class="day" :class="{active: isActive(item.event.start) }">
      <h4 v-if="newDate(index, item.event.start)"  :id="'day-' + dateId(item.event.start)">{{item.event.start | formatDate}}</h4>
      <div class="event">
        <div class="time">{{item.event.start | formatTime}}</div>

        <div class="info">
          <div class="title">
            <a :style="'border-bottom: 1px solid ' + eventColor(item.event_type)" :href="item.url" target="_blank">{{item.title}}</a>
            <div class="options">
              <a class="icon info" :href="item.url" target="_blank"></a>
                <a class="icon twitter" :href="'https://twitter.com/home?status=Check out this upcoming event: ' + item.title + ' - ' + item.url + ' #edgeryders'" target="_blank"></a>
                <a class="icon facebook" :href="'https://www.facebook.com/sharer/sharer.php?u=' + item.url" target="_blank"></a>
                <a class="icon email" :href="'mailto:?subject=Check out this upcoming event by Edgeryders&body=' + item.title + ' - ' +  item.url" target="_blank"></a>
            </div>
          </div>
          <div class="description">
            <div class="excerpt">{{item.cooked}}</div>
          </div>
        </div>

      </div>
    </div>
  </transition-group>


  </div>
<!--   <div class="sidebar">
   
  
 

  </div> -->
</div>
</template>

<script>
import moment from "moment";
import { bus } from '@/main';
import VueScrollTo from 'vue-scrollto';

export default {
  name: 'Timeline',
  data () {
    return {
      dates: [],
      events: [],
      search: "",
      location: null,
      date: null,
      type: null,
      locations: [],
      types: [],
      options: {
        container: '#events_container',
        duration: 500,
         easing: "ease"
      },
      selected: null,
      selectedDate: null,
      select: {
        date: null,
        location: null,
        type: null
      }
    }
  },
  methods: {
    selectDate(){
      this.location = null;
      this.type = null;
    },
    clear(key){
      this[key] = null;
    },
    mapEvents(){
      const events = this.data.reduce((c, v) => {
        let date = this.getDate(v.event.start);
        c[date] = c[date] || [];       //Initiate if key does not exist
        c[date].push(v);                //Push the value
        return c;
      }, {});
      this.events.push(events);

      var dates = this.mapData('date');
      this.dates = this.filterDuplicates(dates);

      this.locations = this.mapData('location');
      this.types = this.mapData('event', 'event_type');
    },
    mapInfo(tag){
      let array = [];
      if (this.date || this.type) {
        array = this.filteredItems.map(obj => 
          obj[tag]
        );
      } else {
        array = this.items.map(obj => 
          obj[tag]
        );
      }
      array = this.filterDuplicates(array);
      return array;
    },
    selectType(tag){
      this.type = tag;
    },
    mapTypes(){
      let array = [];
      if (this.select.date || this.select.type) {
        array = this.items.map(obj => 
          obj.event_type
        );
      } else {
        array = this.items.map(obj => 
          obj.event_type
        );
      }
      array = this.filterDuplicates(array);
      return array;
    },
    newDate(index, date){
      var prevIndex = this.filteredItems[index-1];
      if (index == 0 || moment(prevIndex.event.start).format("YYYY-MM-DD") !== moment(date).format("YYYY-MM-DD")) {
      return true;
      } else {
        return false
      }
    },
    mapData(key, key2){
      let array;
      array = this.items.map(obj => 
        obj[key]
      );
      if (key2) {
        array = this.items.map(obj => 
          obj[key2]
        );
      }
      return array.filter(Boolean);
    },
    filterDuplicates(array){
      let unique = [...new Set(array)];
      return unique.filter(Boolean);
    },
    getDate: function(value) {
      return moment(value).format("dddd, MMMM Do");
    },
    dateId: function(value) {
      return moment(value).format("YYYYMMDD");
    },
    eventColor(type){
      var color = this.config[type];
      if (color) {
        return color;
      } else {
        return null
      }
    },
    isActive(date) {
      if (this.selectedDate == moment(date).format("YYYYMMDD")) {
        return true
      } else {
        return false
      }
    },
    filtered() {
      if (this.type !== null || this.search !== "" || this.location) {
        return true
      } else {
        return false
      }
    }
  },
  created() {
    this.mapEvents();
    bus.$on('setDate', (data) => {
      this.selectedDate = data;
      var scrollId = '#day-' + data;
      VueScrollTo.scrollTo(scrollId, 500, this.options)
    })
    bus.$on('filterDate', (data) => {
      this.date = data;
      this.type = null;
      this.location = null;
    })
    bus.$on('filterSearch', (data) => {
      this.search = data;
      this.type = null;
      this.date = null;
    })
    bus.$on('filterLocation', (data) => {
      this.location = data;
      this.date = null;
    })
    bus.$on('filterType', (data) => {
      this.type = data;
      window.console.log(data);
    })
  },
  filters: {
    formatTime: function(value) {
      return moment(value).format("HH:mm");
    },
    formatDate: function(value) {
      return moment(value).format("dddd, MMMM Do");
    }
  },
  computed: {
    filteredItems() {

      let filtered = this.items;

      if (this.date != null) {
           filtered = filtered.filter(e => e.date == this.date)
      }

      if (this.search != "") {
           filtered = filtered.filter(e => e.title.toLowerCase().includes(this.search.toLowerCase()))
      }

      if (this.location != null) {
           filtered = filtered.filter(e => e.location == this.location)
      }

      if (this.type != null) {
           filtered = filtered.filter(e => e.event_type == this.type.label)
      }

      return filtered;
    }
  },
  props: ["data", "config", "items"]
}
</script>

<style lang="scss" scoped>
.filters {
  @apply w-full flex items-start border-b pb-4;
  width: auto;
  height: 60px;
  p {
    @apply flex items-center px-3;
    height: 100%;
    font-size: 14px;
  }
  .filter {
    height: 30px;
    width: auto;
    @apply flex bg-white mr-2 my-4 border;
    overflow: hidden;
    border-radius: 4px;
    &:hover {
      cursor: pointer;
      opacity: 0.7;
    }
    p.key {
      @apply bg-black font-bold;
      background: rgba(0,0,0,0.03);
      text-transform: uppercase;
      font-size: 9px;
    }
    p {
      font-size: 11px;
      @apply font-bold;
    }
  }
}
.timeline_container {
  @apply flex;
  max-width: 65%;
  justify-content: flex-start;
  align-items: flex-start;
}
.timeline {
  flex-basis: 100%;
  height: 500px;
  overflow: scroll;
}
.list-enter-active, .list-leave-active {
  transition: all .3s ease;
}
.list-enter, .list-leave-to /* .list-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(200px);
}
.day {
  transition: padding .3s ease;
  h4 {
    @apply mt-4 mb-3 font-bold;
  }
  &.active {
    background: #fafafa;
    padding-left: 2%;
    width: 100%;
    h4 {
      @apply pt-4 mt-4 mb-3 ml-2;
      font-size: 1.2rem;
    }
    &.last {
      @apply pb-6;
    }
  }
}

.event {
  @apply flex flex-row;
  width: 100%;
  .info {
    @apply flex w-full flex-col border-l;
    position: relative;
    transition: border .4s ease;
    &:hover {
      border-left: 1px solid #000;
      background: #fafafa;
      cursor: pointer;
      .title {
        padding-bottom: 4px;
        transition: padding 0.2s ease;
      }
      .description {
        max-height: 500px;
        transition: max-height 1s ease;
        @apply pb-2;
      }
    }
  }
  .title a {
    @apply pb-1 mb-0;
  }
  .description {
    @apply pl-4;
    max-height: 0px;
    font-size: 11px;
    
    transition: max-height 0s ease;
    overflow: hidden;
  }
  
  .time {
    @apply pr-3 py-4 text-xs flex flex-col justify-center items-center;
    width: 60px !important;
    flex-shrink: 0;
  }
  .title {
    @apply py-4 pt-2 pl-4 pb-2 flex items-center;
    transition: padding 0s ease;
    &:hover {
      .options {
        opacity: 1;
        transform: translateX(0px);
      }
    }
    .options {
      width: 120px !important;
      opacity: 0;
      transition: all .5s ease;
      transform: translateX(-3px);
      min-width: 120px;
      height: 23px;
      float: left;
      margin-left: 15px;
      display: inline-block;
      .icon {
        width: 23px;
        height: 23px !important;
        display: inline-block;
        border-radius: 4px;
        background: #000;
        margin-right: 5px;
        border: .5px solid #efefef;
        background-color: #fff !important;
        &:hover {
          border: .5px solid #000;
        }
        &.info {
          background: url("data:image/svg+xml,%3Csvg width='17' height='48' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M17 6c0 1.7-.6 3-1.7 4.2a5.6 5.6 0 01-4.1 1.8c-1.7 0-3-.6-4.2-1.8A5.8 5.8 0 015.3 6c0-1.6.6-3 1.7-4.2A5.6 5.6 0 0111.2 0c1.6 0 3 .6 4 1.8A5.9 5.9 0 0117 6zm-.6 39.5l-4 1.9c-1.1.4-2.3.6-3.7.6-2 0-3.7-.6-4.8-1.9a6.6 6.6 0 01-1.7-4.7 20.2 20.2 0 01.5-4.8L5 27.3a51.1 51.1 0 00.6-4.7c0-1.1-.2-2-.6-2.4-.4-.5-1.1-.7-2.3-.7a5 5 0 00-1.7.3c-.6.2-.9-2.3-.9-2.3l4-1.8c1.3-.5 2.5-.7 3.7-.7 2 0 3.7.6 4.8 1.8a6.7 6.7 0 011.7 4.8 32.2 32.2 0 01-.6 4.9l-2.1 9.2a29.2 29.2 0 00-.7 4.7c0 1.2.2 2 .7 2.5.4.4 1.2.6 2.3.6.5 0 1.1 0 1.8-.3.6-.2.8 2.3.8 2.3z' fill='%23000' fill-rule='nonzero'/%3E%3C/svg%3E") no-repeat center 65% #fff;
          background-size: 17% !important;
        }
        &.email {
          background: url("data:image/svg+xml,%3Csvg width='320' height='256' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M114 134C23 68 24 68 0 49V24C0 11 11 0 24 0h272c13 0 24 11 24 24v25c-24 19-23 19-114 85-11 8-32 26-46 26-15 0-35-18-46-26zm206-44v142c0 13-11 24-24 24H24c-13 0-24-11-24-24V90l95 70c14 10 38 32 65 32s51-22 65-32l95-70z' fill='%23000' fill-rule='nonzero'/%3E%3C/svg%3E") no-repeat center 51% #fff;
          background-size: 50% !important;
        }
        &.twitter {
          background: url("data:image/svg+xml,%3Csvg aria-hidden='true' data-prefix='fab' data-icon='twitter' class='svg-inline--fa fa-twitter fa-w-16' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 512 512'%3E%3Cpath fill='currentColor' d='M459 152l1 13c0 139-106 299-299 299-59 0-115-17-161-47a217 217 0 00156-44c-47-1-85-31-98-72l19 1c10 0 19-1 28-3-48-10-84-52-84-103v-2c14 8 30 13 47 14A105 105 0 0136 67c51 64 129 106 216 110-2-8-2-16-2-24a105 105 0 01181-72c24-4 47-13 67-25-8 24-25 45-46 58 21-3 41-8 60-17-14 21-32 40-53 55z'/%3E%3C/svg%3E") no-repeat center 51% #fff;
          background-size: 50% !important;
        }
        &.facebook {
          background: url("data:image/svg+xml,%3Csvg aria-hidden='true' data-prefix='fab' data-icon='facebook-f' class='svg-inline--fa fa-facebook-f fa-w-10' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 320 512'%3E%3Cpath fill='currentColor' d='M279 288l14-93h-89v-60c0-25 13-50 53-50h40V6s-37-6-72-6c-73 0-121 44-121 125v70H23v93h81v224h100V288z'/%3E%3C/svg%3E") no-repeat center 54% #fff;
          background-size: 30% !important;
        }
      }
    }
  }
}
</style>