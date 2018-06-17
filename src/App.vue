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
            <h2>Showing {{all_data.length}}  Results</h2>
            <div class="items">
              <div class="item" v-for="city in city_arr" @click="show_item(city)">
                <div class="img_block">
                  <img :src="city.Picture1" alt="">
                </div>
                <div class="info">
                  <h3>{{city.Name}}</h3>
                  <p class="description">{{city.Description}}</p>
                  <p class="ticket">{{city.Ticketinfo}}</p>
                  <div class="addition">
                    <span>地址:</span><p> {{city.Add}}</p>
                    <span>開放時間:</span><p> {{city.Opentime}}</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="content" v-else>
            <div class="inner_data">
              <img :src="inner_data.Picture1" alt="">
              <div class="info">
                <h3>{{inner_data.Name}}</h3>
                <p class="description">{{inner_data.Description}}</p>
                <p class="ticket">{{inner_data.Ticketinfo}}</p>
                <div class="addition">
                  <span>地址:</span><p> {{inner_data.Add}}</p>
                  <span>開放時間:</span><p> {{inner_data.Opentime}}</p>
                </div>
                <div class="back" @click="show_inner = false">返回</div>
              </div>
              
            </div>
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
      this.show_inner = false
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
  @mixin inner_data {
    .img_block {
      max-width: 240px;
      min-width: 240px;
      height: 100%;
      img {
        width: 100%;
        height: 100%;
      }
    }
    .info {
      margin-left: 15px;
      h3 {
        margin: 0;
      }
      .description {
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      .ticket {
        color: #ccc;
        font-weight: bold;
        margin: 0;
      }
      .addition {
        display: flex;
        span {
          font-weight: bold;
          color: #ccc;
        }
        p {
          display: inline-block;
          margin: 0;
          margin: 0 15px;
          font-weight: 500;
        }
      }
    }
  }
  * {
    font-family: '微軟正黑體';
  }
  html, body {
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
  }
  .zone {
    display: inline-block;
  }
  .pagination {
    display: flex;
    list-style: none;
    align-items: center;
    color: blue;
    justify-content: flex-end;
    margin-right: 10px;
    .page-item {
      display: inline-block;
      list-style: none;
      margin: 0 5px;    
      padding: 10px;
    }
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
        background-color: #9013FE;
        height: 40px;
        display: flex;
        align-items: center;
        padding: 10px 0px 10px 40px;
        h2 {
          color: white;
          cursor: pointer;
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
        margin-top: 20px;
        .filters {
          display: flex;
          flex-direction: column;
          width: 20%;
          margin-left: 20px;
          max-width: 400px;
          .block {
            border-radius: 3px;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            background-color: #eee;
            .zone {
              margin: 10px 10px 10px 8px;
            }
          }
        }
        .content {
          width: 80%;
          display: flex;
          flex-direction: column;
          margin-bottom: 20px;
          .inner_data {
            width: 80%;
            display: flex;
            flex-direction: column;
            margin: auto;
            img {
              width: 100%;
            }
            .ticket {
              color: #ccc;
              font-weight: bold;
            }
            .addition {
              display: flex;
              span {
                font-weight: bold;
                color: #ccc;
              }
              p {
                display: inline-block;
                margin: 0px 15px;
                font-weight: 500;
              }
            }
            .back {
              display: inline-block;
              background-color: #ccc;
              text-align: center;
              padding: 8px 20px;
              color: white;
              cursor: pointer;
              border-radius: 3px;
              margin-top: 10px;
            }
          }
          h2 {
            display: inline-block;
            margin: 0;
            margin-left: 30px;
            color: #aaa;
          }
          .items {
            padding: 0 20px;
            .item {
              height: 200px;
              cursor: pointer;
              border: solid 1px #ccc;
              margin: 15px 0;
              display: flex;
              justify-content: space-between;
              align-items: center;
              @include inner_data;
            }
          }
        }
      }
    }
  }
</style>