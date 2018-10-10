<template>
  <div class="home">
    <p>
      <img src="/leapai.png"/>
    </p>
    <p>Work in progress... <a href="https://github.com/leapai/leaplate">Contribute!</a></p>
    <p>This application is a template project for LeapAI.com (a.k.a. Leaplate) using Meteor and Vue.</p>
    <p><a href="https://github.com/leapai/leaplate" target="_blank">More info</a></p>

    <p>{{ counter }}</p>
    <p><button @click="onClick">Increment</button></p>
  </div>
</template>

<script>
import { Meteor } from 'meteor/meteor'

export default {
  data () {
    return {
      limit: 10,
      counter: 0,
    }
  },

  methods: {
    onClick () {
      this.counter++
    },
    initiateAPICall() {
      // When you create an App on LeapAI.com, a unique AppID (GUID) will be generated. If you want to 
      // use an API, you need to add an API-binding to this App on the dev center. Under the hood the API will 
      // be added to the allowed_api_list of that App.
      // When you actually call the API through the API Gateway, you need to include the appId as shown below.
      //  
      // appId can be harded-coded on the frontend, or retrieved from the server through a meteor call.
      Meteor.call('getEnv', "LEAPAPP_ENV", (err, results) => {
        let mapEnv = JSON.parse(results);
        let appId = mapEnv.appid;
        let url = 'https://api.leapai.com/leapiot/dt/open/v1/instances?appId=' + appId;

        this.$http.get(url)
          .then(response=> {
            // When API Gateway delegates other APIs, the first call may fail due to 403 or other issues.
            // Even if the first call succeeds, the second (delegated) API call may still fail. 
            // For this reason, we need to check for both the API calls' status before proceeding.  
            if(response.status == 200 && 
              response.data.status == "200") { 
              // You need to save the commandId so that you can lookup the result of that command later.
              this.currentCommandId = response.data.result.commandId;
              console.log('Saved commandId = ' + this.currentCommandId);
            }
          })
          .catch(error=>{
            console.log('API call Error' + error);
          });
      });
    }
  },

  mounted() {
    this.initiateAPICall();
  },

  meteor: {
    $subscribe: {
      // Meteor $subscribe allows you to watch for the changes of a collection 
      'things' () {
        console.log('meteor.$subscribe: things');
        return [10]
      },
      'requestcommandresult' () {
        return [100]
      }
    },
    allThings () {
      console.log('allThings changed');
      return Things.find({}, {
        /*sort: { created: -1 },*/
      })
    },
    updateUIData () {
      console.log('CurrentCommandId = ' + this.currentCommandId);
      var crData = CommandResults.find({/*_id: this.currentCommandId*/}, {}).fetch();

      if (crData.length > 0) {
        for (let i=0,len=crData.length; i<len; i++) {
          let data = crData[i];
          if (data._id == this.currentCommandId) {
            // We only care about the result of the commnad we executed earlier on this page
            let payload = JSON.parse(data.commandResult);
            let pdata = payload.data;
            console.log('Got command data:' + pdata);
          }
        }
      }

      var mdata = Metrics.find({}, {
        sort: {created: -1},
        limit: 20
      }).fetch();

      if (mdata.length > 0) {
        for(let i=0, len=mdata.length; i<len; i++) {
          log(mdata[i]);
        }
      }
    }
  }
}
</script>

<style lang="less" scoped>
.home {
  text-align: center;
}
</style>
