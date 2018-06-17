<template>
  <div id="app">
    <div class="wrapper">
      <div class="container">
        <nav>
          <h2 @click="show_inner = false">HaveFun</h2>
          <div class="search">
            <input type="text" value="" v-model="condition.search" @keyup.enter="filter_data" style="display: block">
            <i class="fa fa-search"></i>
          </div>
        </nav>
        <div class="main">
          <div class="filters">
            <div class="block">
              <div class="zone" v-for="zon in zone">
                <input type="checkbox" :value="zon" :id="zon" v-model="condition.zone" @click="check_zone">
                <label :for="zon">{{zon}}</label>
              </div>
            </div>
            <hr>
            <div class="block">
              <div class="zone" v-for="open in is_open">
                <input type="checkbox" :value="open" :id="open" v-model="condition.is_open" @click="check_open">
                <label :for="open">{{open}}</label>
              </div>
            </div>
          </div>
          <div class="content" v-if="!show_inner">
            <ul>
              <li v-for="city in city_arr" @click="show_item(city)">{{city.Name}}</li>
            </ul>
          </div>
          <div class="content" v-else>
            <h1>{{inner_data.Name}}</h1>
          </div>
        </div>
      </div>
      <paginate
        v-if="!show_inner"
        :page-count="page_count"
        :page-range="5"
        :margin-pages="2"
        :click-handler="show_pagi_data"
        :container-class="'pagination'"
        :page-class="'page-item'">
      </paginate>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data () {
    return {
      page_count: 0,
      current_page: 1,
      init_url: 'https://data.kcg.gov.tw/api/action/datastore_search?resource_id=92290ee5-6e61-456f-80c0-249eae2fcc97&limit=10',
      data_url: 'https://data.kcg.gov.tw/api/action/datastore_search?resource_id=92290ee5-6e61-456f-80c0-249eae2fcc97',
      city_arr: [],
      all_data: [],
      init_data: [],
      show_inner: false,
      is_open: ['免費參觀', '全天候開放'],
      zone: [],
      inner_data: {},
      condition: {
        zone: [],
        search: '',
        is_open: [],
      }      
    }
  },
  methods: {
    filter_data () {
      const datas = []
      let final = []
      if(this.condition.zone.length == 0 && this.condition.search == '' && this.condition.is_open.length == 0) {
          this.all_data = this.init_data
          this.city_arr = this.city_arr = this.all_data.slice((this.current_page - 1) *10, (this.current_page*10))
          this.page_count = Math.ceil(this.all_data.length / 10)
          return
      }
      else {
        this.all_data = this.init_data
        if (this.condition.search != '') {
          this.all_data = this.all_data.filter(data => {
            if (data.Name.indexOf(this.condition.search) != -1) {
              return data
            }
          })
        }
        if (this.condition.zone.length > 0) {
          this.all_data = this.all_data.filter(data => {
            if (this.condition.zone.includes(data.Zone)) {
              return data
            }
          })
        }
        if (this.condition.is_open.length > 0) {
          this.all_data = this.all_data.filter(data => {
            if (this.condition.is_open.includes(data.Ticketinfo) || this.condition.is_open.includes(data.Opentime)) {
             return data
            }
          })
        }
      }
      this.city_arr = this.all_data.slice((this.current_page - 1) *10, (this.current_page*10))
      this.page_count = Math.ceil(this.all_data.length / 10)
    },
    check_open (e) {
      if (e.target.checked) {
        this.condition.is_open.push(e.target.value)
      }
      if (!e.target.checked) {
        if (this.condition.is_open.includes(e.target.value)) {
          let is_open_arr = this.condition.is_open.filter((value) => value !== e.target.value)
          this.condition.is_open = is_open_arr
        }
      }
      this.filter_data()
    },
    check_zone(e) {
      if (e.target.checked) {
        this.condition.zone.push(e.target.value)
      }
      if (!e.target.checked) {
        if (this.condition.zone.includes(e.target.value)) {
          let zone_arr = this.condition.zone.filter((value) => value !== e.target.value)
          this.condition.zone = zone_arr
        }
      }
      this.filter_data()
    },
    show_item (item) {
      this.inner_data = item
      this.show_inner = true
    },
    show_pagi_data (pageNum) {
      this.city_arr = this.all_data.slice((pageNum - 1) *10, (pageNum*10))
      return this.city_arr
    }
  },
  created () {
    axios.get(this.init_url)
      .then(res => {
        this.city_arr = res.data.result.records
      })
    axios.get(this.data_url)
      .then(res => {
        this.all_data = [res.data.result.records][0]
        this.init_data = [res.data.result.records][0]
        this.page_count = Math.ceil(this.all_data.length / 10)
        this.init_data.forEach((data) => {
          if (!this.zone.includes(data.Zone)) {
            this.zone.push(data.Zone)
          }
        })
      })
  }
}
</script>

<style lang="scss">
  html, body {
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
  }
  .zone {
    display: inline-block;
  }
  .page-item {
    display: inline-block;
    list-style: none;
  }
  .wrapper {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    .container {
      display: flex;
      flex-direction: column;
      nav {
        width: 100%;
        background-color: #9013FE;
        height: 40px;
        display: flex;
        align-items: center;
        padding: 10px 0px 10px 40px;
        h2 {
          color: white;
        }
        .search {
          margin-left: 10%;
          position: relative;
          input {
            border-radius: 5px;
            font-size: 14px;
            padding: 5px 30px;
            width: 200px;
          }
          i {
            color: #ccc;
            position: absolute;
            left: 10px;
            top: 7px;
          }
        }
      }
      .main {
        display: flex;
        .filters {
          display: flex;
          flex-direction: column;
          width: 35%;
          max-width: 400px;
          .block {
            width: 95%;
            margin-left: 5%;
            border-radius: 5px;
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            background-color: #ddd;
            .zone {
              margin: 10px 10px 10px 8px;
            }
          }
        }
      }
    }
  }
</style>