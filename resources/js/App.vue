<template>
  <v-app>
    
    <alert />

    <!-- v-model dialog? or that -->
    <keep-alive>
      <v-dialog v-model="dialogStatus" fullscreen persistent transition="dialog-bottom-transition">
        <component :is="currentComponent" @closed="setDialogStatus"></component>
      </v-dialog>
    </keep-alive>

    <v-navigation-drawer app v-model="drawer">
      <v-list>

        <v-list-item v-if="!guest"> 
          <v-list-item-avatar>
            <v-img :src="user.user.photo"> </v-img>
          </v-list-item-avatar>
          <v-list-item-content> 
            <v-list-item-title>{{user.user.name}}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <div class="pa-2" v-if="guest"> 
          <v-btn block color="light-blue accent-4 white--text" class="mb-1" @click="setDialogComponent('login')">
            <v-icon left>mdi-lock</v-icon>
              Login
          </v-btn>
          <v-btn block color="teal accent-4 white--text" >
            <v-icon left>mdi-account</v-icon>
                Register
          </v-btn>
        </div>

        <v-divider></v-divider>

        <v-list-item 
          v-for="(item, index) in menus"
          :key="`menu-`+index"
          :to="item.route" 
        >
              
          <v-list-item-icon> 
            <v-icon left> {{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title> {{ item.title }} </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
          
      </v-list>
      
      <template v-slot:append v-if="!guest">
        <div class="pa-2">
            <v-btn block color="red" dark @click="logout">
              <v-icon left>mdi-lock</v-icon>
                Logout
            </v-btn>
        </div>
      </template>
     
    </v-navigation-drawer>

    <v-app-bar app color="indigo accent-2" dark v-if="isHome">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>WeAllCare</v-toolbar-title>

      <!-- pemisah konten -->
      <v-spacer></v-spacer>

      <v-btn icon>
        <v-badge overlap color="orange" v-if="transactions>0">
          <template v-slot:badge>
            <span>{{ transactions }}</span>
          </template>
          <v-icon>mdi-cash-multiple</v-icon>
        </v-badge>
        <v-icon v-else>mdi-cash-multiple</v-icon>
      </v-btn>

      <v-text-field 
        slot="extension"
        hide-details
        append-icon="mdi-microphone"
        flat
        label="Search"
        prepend-inner-icon="mdi-magnify"
        solor-solo-inverted
        @click="setDialogComponent('search')"
      ></v-text-field>
         
    </v-app-bar>

    <v-app-bar app color="indigo accent-2" dark v-else>
      <v-btn icon @click.stop="$router.go(-1)">
        <v-icon>mdi-arrow-left-circle</v-icon>
      </v-btn>

      <v-spacer></v-spacer>

       <v-btn icon>
        <v-badge overlap color="orange" v-if="transactions>0">
          <template v-slot:badge>
             <span>{{ transactions }}</span>
          </template>
          <v-icon>mdi-cash-multiple</v-icon>
        </v-badge>
        <v-icon v-else>mdi-cash-multiple</v-icon>
      </v-btn>
    </v-app-bar>
      
 

    <!-- Sizes your content based upon application components -->
    <v-main>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <!-- If using vue-router -->
        <v-slide-y-transition>
          <router-view></router-view>
        </v-slide-y-transition>
      </v-container>
    </v-main>

    <v-card >
      <v-footer absolute app color="indigo accent-4 white--text">
        <v-card-text class="text-center">
          &copy; {{ new Date().getFullYear() }} - <strong>Crowdfunding </strong>
        </v-card-text>
      </v-footer>  
    </v-card>
  </v-app>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
// import Alert from './components/Alert.vue';
// import Search from './components/Search.vue';
  export default {
    name: 'App',
    components : {
      Alert : () => import('./components/Alert'),
      Search : () => import('./components/Search'),
      Login : () => import('./components/Login')
    },
    data: () => ({
      drawer: false,
      menus: [
        {title: 'Home', icon:'mdi-home', route:'/'},
        {title: 'Campaigns', icon:'mdi-hand-heart', route:'/campaigns'},
      ],
    }),
    computed: {
      isHome () {
        return (this.$route.path==='/' || this.$route.path === '/home')
      },
      ...mapGetters({
        transactions            : 'transaction/transactions',
        guest                   : 'auth/guest',
        user                    : 'auth/user',
        dialogStatus            : 'dialog/status',
        currentComponent        : 'dialog/component',
      })
    },
    dialog: {
      get() {
        return this.dialogStatus
      },
      set (value) {
        this.setDialogStatus(value)
      }
    },
    methods: {
      ...mapActions({
        setDialogStatus       : 'dialog/setStatus',
        setDialogComponent    : 'dialog/setComponent',
        setAuth               : 'auth/set',
        setAlert              : 'alert/set',
        checkToken            : 'auth/checkToken',
      }),
      logout(){
        let config = {
          headers: {
            'Authorization' : 'Bearer ' + this.user.token,
          }
        }
        axios.post('/api/auth/logout', {}, config)
          .then((response) => {
            this.setAuth({}) //kosongkan auth ketika logout
            this.setAlert({
              status :  true,
              color  : 'success',
              text : 'Logout successfully'
            })
          })
          .catch((error) => {
            let { data } = error.response
            this.setAlert({
              status : true,
              color : 'error',
              text : data.message
            })
          })
      }
    },
    mounted(){
      if(this.user){
        this.checkToken(this.user)
      }
    },
  }
</script>

<style>

</style>