<template>
  <header>
    <router-link to="/">
      <h1>NTU EAT</h1>
    </router-link>
    
    <div class="notLoggedin" :class="{'display': status}">

      <router-link to="/signin">登入&nbsp;</router-link>
      /
      <router-link to="/signup">&nbsp;註冊</router-link>

    </div>
    
    <div class="loggedIn" :class="{'display': status}">
      
      <router-link to="/getplaceid" v-if="userRole > 3">
        <span class="material-symbols-outlined">edit_note</span>
      </router-link>
      
      <router-link to="#">
        <span class="material-symbols-outlined">bookmark</span>
      </router-link>
      <router-link to="/profile">
        <img :src="avatar" v-if="userRole > 3 && (avatar !== null)">
        <span class="material-symbols-outlined avatar" v-else>account_circle</span>
      </router-link>

    </div>
  
  </header>
</template>

<style scoped lang="scss">
  
header{
  display: flex;
  background-color: #F0E4D4;
  align-items: center;
  justify-content: space-between;
  h1{
    margin-left: 18px;
    font-size: 24px;
    line-height: 29px;
    text-align: center;
    font-weight: 600;
    color: #575757;
    display: inline-block;
  }
  a{
    &:first-child{
      margin-right: 0; 
    }
    display: inline-block;
    line-height: 23px;
    margin-right: 20px;
    font-size: 16px;
    font-weight: 600;
    color: #707070;
    text-decoration: none;
  }
  .loggedIn{
    display: none;
    a{
      margin: 0 18px 0 0;
      vertical-align: middle;
      cursor: pointer;
      span{
        vertical-align: middle;
        font-variation-settings:
        'wght' 300,
      }
      .avatar{
        font-size: 36px;
      }
    }
    a{
      margin: 0 18px 0 0;
      vertical-align: middle;
      cursor: pointer;
      img{
        display: inline-block;
        border-radius: 50%;
        height: 32px;
        width: 32px;
        object-fit: cover;
        vertical-align: middle;
      }
    } 
  }
  .display.loggedIn{
    display: inline;
  }
  .notLoggedin.display{
    display: none;
  }
  .vld-overlay{
    backdrop-filter: blur(2px);
  }
}
</style>


<script>
import { getApi, getData } from '@/GlobalSettings.js'
export default {
  name: 'NeHeader',
  data() {
    return {
      status: false,
      userRole: null,
      avatar: null,
    }
  },

  setup() {
    const api = getApi();
    const data = getData();
    return{
      api,
      data
    }
  },

  created(){
    this.getUserStatus()
  },
  
  methods: {
    getUserStatus() {
      const _this = this
      if (localStorage.getItem('loginState')) {
        this.status = localStorage.getItem('loginState')
        this.userRole = localStorage.getItem('userRole')
        this.getAvatar()
      } else {
        this.axios.get(_this.api + "user/status/", {withCredentials: true})
        .then((res) => {
          if (res.data.data.status){
            _this.status = res.data.data.status.isLoggedIn
            _this.userRole = res.data.data.status.userRole.num
            if (this.userRole >= 5) {
              this.getAvatar()
            }
          } else{
            const errorMsg = res.data.data.status
            console.log(errorMsg)
          }
        })
        .catch(function(error) {
          console.log(error)
        })
      }
    },

    getAvatar() {
      const _this = this
      if (localStorage.getItem('avatar')) {
        this.avatar = localStorage.getItem('avatar')
      } else {
        this.axios.get(_this.api + "user/info/", {withCredentials: true})
        .then((res) => {
          if (res.data.data){
            if (res.data.data.superUser.avatar[4].filename == null) {
              this.avatar = null
            } else{
              this.avatar = this.data + 'user/' + res.data.data.superUser.avatar[4].filename
              localStorage.setItem('avatar', this.data + 'user/' + res.data.data.superUser.avatar[4].filename)
            }
          } else{
            const errorMsg = res.data.data
            console.log(errorMsg)
          }
        })
        .catch(function(error) {
          console.log(error)
        }) 
      }
    },
  }
}
</script>
