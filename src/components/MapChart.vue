<template>
<div class="main">
  <highcharts :constructor-type="'mapChart'" :options="mapOptions" class="map"></highcharts>
  <ul class="cardlist">
    <li v-for="item in clist"  v-bind:key="item" >{{ item.name }} <br> GDP: {{item.gdp }} <br> % land Agriculture: {{item.agi}}</li>
  </ul>
</div>
</template>





<script>
//want to define and display iso-a3 codes
// restful countries "alpha3Code":"AFG",
//"population":27657145,

//import Highcharts from 'highcharts'
//using a global to hold the vue component becuse I dont know what else to do. 
var rootObj = {}; 


//Copped this code from here
//https://stackoverflow.com/questions/12460378/how-to-get-json-from-url-in-javascript
//highcharts.getJSON was not able to source correctly, and I couldn't figure out how to fix it
//Modified code to include obj paramater which is used to populate the map series data. 
var getJSON = function(url, obj, callback) {
    var xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);
    xhr.responseType = 'json';
    xhr.onload = function() {
      var status = xhr.status;
      if (status === 200) {
        callback(null, obj, xhr.response) ;
      } else {
        callback(status, obj,  xhr.response);
      }
    };
    xhr.send();
};



export default {
  methods:{
    addCard: function(data) {
      this.clist.push(data);
    }
  },
  data () {
    rootObj = this; 
    //console.log(rootObj)
    return {
      clist: [
      ],
      mapOptions: {
        chart: {
          map: 'myMapName'
        },
        title: {
          text: 'Country Population Heatmap'
        },
        subtitle: {
          text: 'Source map: <a href="http://code.highcharts.com/mapdata/custom/world.js">World, Miller projection, high resolution</a>' //this map 
        },
        mapNavigation: {
          enabled: true,
          buttonOptions: {
            alignTo: 'spacingBox'
          }
        },
        colorAxis: {
          min: 0
        },

        series: [{
          name: 'Population', //modify this for the name
          joinBy:['iso-a3', 'code'],
          point: {
            events: {
              click: function() {
                var self = this //inidicator gdp
                getJSON('https://api.worldbank.org/v2/country/' + this.code + '/indicator/NY.GDP.MKTP.CD?date=2016&format=json', this, function(err, obj, data) {
                  if (err !== null) {
                      alert('Something went wrong: ' + err);
                  } else {
                      //inidicator % of land agrictultural purposes
                  getJSON('https://api.worldbank.org/v2/country/' + self.code + '/indicator/AG.LND.AGRI.ZS?date=2016&format=json', this, function(err, dummyobj, idata2) {
                      if (err !== null) {
                      alert('Something went wrong: ' + err);
                       } else {
                   // console.log(rootObj) //globaly actually still in scope, I would say expected if this hadn't all been nonsense so far. 
                   // console.log(rootObj.data)
                     //finally we have data
               
                  // for(var i in shadowArray) {
                    // console.log('shadowArray index' + i)
                    // console.log(shadowArray[i])

                  // }
               //   console.log('data')
               //   console.log(data)
              //    console.log('Idata 2')
              //    console.log(idata2)

                  var cardData = {}
                   var res = false; 
                   var spliceIndex = 0; 
                //   console.log('temp array')
                   try {
                     var tempArray = rootObj.clist //JSON.parse(JSON.stringify(rootObj.clist))
                     //console.log(tempArray)
                     for(var i = 0; i < tempArray.length; i++){
                   //      console.log('temp array at index: ' + i)
                   //      console.log(tempArray[i])
                      //   console.log(' shadowArray code: ' + tempArray[i][1][0].countryiso3code + ' data code: ' + data[1][0].countryiso3code)
                         if(tempArray[i].code === data[1][0].countryiso3code){
                           res = true
                           spliceIndex = i; 
                           break; 
                         }
                     }
                  //   console.log('response code: ' + data[1][0].countryiso3code)
                  //   console.log('result in try: '  + res)
                   }
                   catch (err){
                     console.log('ignore me')
                   }

                    
                   // console.log('result after try: ' + res)
                    if(res == false){
                      //Think I can just construct the data here
                      cardData.code = data[1][0].countryiso3code;
                      cardData.name = data[1][0].country.value;
                      cardData.gdp = data[1][0].value;
                      cardData.agi = idata2[1][0].value;
                      /////
                      console.log('cardData')
                      console.log(cardData)
                      rootObj.clist.push(cardData)
                  //    console.log('data being pushed: ' + data  +', Index: ' + shadowArray.length)
                    } else {
                 //     console.log('Index: ' + spliceIndex)
                      rootObj.clist.splice(spliceIndex,1)
                    }
                   // console.log(rootObj.clist)
                   // rootObj.clist.push(data)
                    //alert(JSON.stringify(data))
                       }

                  

                })}});
              }
            }
          },
          states: {
            hover: {
              color: '#BADA55'
            }
          },
          dataLabels: {
            enabled: true,
            format: '{point.name}'
          },
          allAreas: false,
          data: [  //data specifciation
            ['fo', 0],
            ['np', 212]
          ]
        }]
      }
    }
  },
  mounted () {
   getJSON('https://restcountries.eu/rest/v2/all', this, function(err, obj, data) {
    if (err !== null) {
      alert('Something went wrong: ' + err);
    } else {
     // console.log(data)
    //run through data in series? 
    //obj.$children[0].chart.series[0].setData(data, true); //Some shit with the point options. 
    //obj.$children[0].chart.redraw(); //re-draw the map, but the map data didn't change need to update some other shit. 
    for(var i in data) {
        var country = data[i];
       // console.log("new object!")
        country.code = country.alpha3Code
        country.value = country.population
        delete country.population
        delete country.alpha3Code
       // console.log(country)
      }
     obj.$children[0].chart.series[0].setData(data, true);
    }});
  }
}
</script>
 <style scoped>
.map {
  min-height: 800px;
  border-radius: 25px; 
}

li {
 /* float: left; */
  border: 10px solid rgba(0, 0, 0, 0);
  font-size: 18px;
  height: 100px;
  width: 400px;
  line-height: 18px;
  background: white;
  display: block;
  text-align: center;
  float: left;
  background-clip: padding-box;
  border-radius: 25px; 

}


 </style>
