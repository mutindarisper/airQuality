<template>
  <div id="app" style="height: 100vh; width: 100vw">
     <div class="navbar">
      <Navbar />
    </div>
    <div id="map" style="height: 98vh; width: 100%; ">

    <div class="map_controls">
       <div class="zoomin_tool">
      <img src="./assets/images/zoom_in.svg" alt="" title="Zoom in"  @click="zoom_in" >
    </div>

     <div class="zoomout_tool">
      <img src="./assets/images/zoom_out.svg" alt="" title="Zoom out" @click=" zoom_out">
    </div>

    <div class="basemaps">
      <img src="./assets/images/layers.svg" alt=""
      @mouseover="base_map_ctrl_selections = true"
        @mouseleave="handle_baseLayers"
      
      >
    </div>
    <!-- /////////////////////////LOADING VECTOR AND RASTER DIRECTLY FROM THE PARENT COMPONENT////////////////// -->

    <!-- <div class="load" style="position: absolute; top:40%; z-index: 800; ">
      <button  @click="load_vector()" style="background-color:#F5E5A9; height:50px; width:80px; border-radius:10px; font-weight: bold; cursor:pointer;" type="submit">load vector</button>
      <br>
      <button  @click="load_rasters()" style="background-color:#F5E5A9; height:50px; width:80px; border-radius:10px; font-weight: bold; display: flex; margin-top: 10%; cursor:pointer;" type="submit">load raster</button>
    </div> -->


     <div
      class="base_map_ctrl_selections"
      v-if="base_map_ctrl_selections"
      @mouseover="
      (base_map_ctrl_selections = true), (base_map_ctrl_cliked = true)
      "
      @mouseleave="
        (base_map_ctrl_selections = false), (base_map_ctrl_cliked = false)
      "
    > 
    
      <div v-for="base_map in Object.keys(baseMaps)" :key="base_map">
        <div class="base_map" @click="change_base_map(base_map)">
          <div class="base_map_name">{{ base_map }}</div>
        </div>
      </div>
    </div>

  
    <div class="swap_tool">
      <img src="./assets/images/swap.svg" alt="" title="Compare">
    </div>

    <div class="analysis_tool" title="Analyze">
      <img src="./assets/images/analysis.svg" alt=""  @click="handle_analysis = true">
    </div>

     <div class="legend_tool" title="Legend">
      <img src="./assets/images/legend.svg" alt=""  @click="show_legend()">
    </div>
    </div>
    

    
    

    

    </div>

     <!-- right map container -->
    <div class="right_map_container" v-if="handle_analysis">
       <img class="close_right_map_container" src="./assets/images/close.svg" @click="close_container('handle_analysis')" />
       <RightMapContainer />
         <!-- <AnalysisFormComponent
         @country_vector_data2="handle_vector_data"
         @country_raster_data="handle_raster_data"
         /> -->
    </div> 
    <!-- <Chart /> -->


    
    
     

  </div>
</template>

<script>


 

import "leaflet";
import L from "leaflet";
import "leaflet/dist/leaflet.css"

 import baseLayers from "./Helpers/baseLayers";
 

// JS import
import 'leaflet.control.opacity';
import "leaflet-geoserver-request";
import "leaflet.wms";
// import WMSCapabilities from "wms-capabilities/wms-capabilities.min.js";
//import axios from "axios";

import RightMapContainer from './components/RightMapContainer.vue'
import "leaflet-geoserver-request/src/L.Geoserver.js"
 //import {show_rasters} from  './components/RightMapContainer'
import Navbar from './components/Navbar'
// import Chart from './components/Chart'
// import $ from "jquery";
// import "leaflet-side-by-side";
// import "bootstrap-select.min.js";
// import "bootstrap-select.min.css";
import axios from "axios";
// import show_rasters from "./components/RightMapContainer.vue"



 

export default {
  name: 'MyAwesomeMap',
  components: {
    RightMapContainer,
    Navbar,
    // Chart

  
 
    
  },
   data() {
    return {
      center: [0.02, 37.8582273], // set initial map center
      map: null, //instance of map object,
      handle_analysis: false,
       
      base_map_ctrl_selections: false, //show or hide base layers
      base_map_ctrl_cliked: false,
      baseMaps: {}, //holds the base
     show_rasters: true,
      current_geojson: null, 
      current_raster: null,
      legend: null,

       
     

      
  
    }
    
     
  },

     
      
      mounted() {
        this.setupLeafletMap();
      },


methods: {
            
            
                 close_container(container) {
                    this[container] = false;
                  },

                   zoom_in() {
                    this.map.setZoom(this.map.getZoom() + 1);
                  },

                  zoom_out() {
                    this.map.setZoom(this.map.getZoom() - 1);
                  },

   

    setupLeafletMap() {
                const { osm, mapbox, mapboxSatellite} = baseLayers;

                
              
          this.baseMaps = {
            OpenStreetMap: osm,
            MapBox: mapbox,
            MapBoxSatellite: mapboxSatellite,
          
          };

       
                  
              this.map = L.map("map", {
            zoomControl: false,
            layersControl: false,
            center: this.center,
            minZoom: 4.4,
            maxZoom: 17,
            zoom: 7,
            layers: [mapbox],
          }); // add the basemaps to the controls
           
          L.control.layers(this.baseMaps).addTo(this.map);
            
          

                this.current_top_base_layer = "MapBox";

              ///////////////////hide layers control
                var layerControl = document.getElementsByClassName('leaflet-control-layers');
                layerControl[0].style.visibility = 'hidden';

                //  L.control.sideBySide(this.baseMaps.OpenStreetMap, this.baseMaps.MapBox).addTo(this.map);
                this.map.createPane('rasters');
                this.map.getPane('rasters').style.zIndex = 1050;
      },

      load_vector() {
     axios.get(
    "http://localhost:8005/geoserver/airqualitytest/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=airqualitytest%3Akisumu_boundary&maxFeatures=50&outputFormat=application%2Fjson"
  ).then( response => {
    console.log(response.data);
    // this.post = response.data;

    L.geoJSON(response.data).addTo(this.map);
  })

      },
         load_rasters() {
           console.log("loading raster!")
      var wmsLayer = L.tileLayer.wms("http://localhost:8005/geoserver/air_quality_rasters/wms?", {
        pane: 'rasters',
    transparent: true,  
       format: 'image/png',
    layers: 'air_quality_rasters:NO2-2019-NAI_test, air_quality_rasters:NO2-2019-MSA',
    tiles: true,  
});
wmsLayer.addTo(this.map);

      },
      //////////////////////////////////////////   VECTOR   /////////////////////////////////////////////

      handle_vector_data(val) {
      if (this.current_geojson) this.map.removeLayer(this.current_geojson);
      this.current_geojson = L.geoJSON(val, {
        color: "black",
        fillColor: "none",
        opacity: 1,
      });
      this.current_geojson.addTo(this.map);
      this.map.fitBounds(this.current_geojson.getBounds(), {
        padding: [50, 50],
      });
    },

    
///////////////////////////////////////////////  RASTER  .getBounds() ///////////////////////////////////////////////////
       handle_raster_data() {
       if (this.current_raster) this.map.removeLayer(this.current_geojson);
      this.current_raster = L.tileLayer.wms("http://localhost:8005/geoserver/air_quality_rasters/wms?", {
        pane: 'rasters',
    transparent: true,  
       format: 'image/png',
    layers: 'air_quality_rasters:NO2-2019-NAI_test, air_quality_rasters:NO2-2019-MSA',
    tiles: true,  
});
      this.current_raster.addTo(this.map);
      // this.map.fitBounds(this.current_raster, {
      //   padding: [50, 50],
      // });
    },

    //////////////////////// this is a legend plugin /////////////////////////////////////////////////////////////////////////////////
    show_legend() {

    var layerLegend = L.Geoserver.legend("http://localhost:8005/geoserver/wms", {
  layers: "air_quality_rasters:NO2-2019-NAI_test",
  request: 'GetLegendGraphic',
     version: 1.0,
         //pane: 'rasters',
    transparent: true,  
       format: 'image/png',
       width: 20,
       height: 20,
  style: `Raster_Legend`,

});

layerLegend.addTo(this.map);

    },


     

     
              handle_baseLayers() {
            setTimeout(() => {
              if (this.base_map_ctrl_cliked === false)
                this.base_map_ctrl_selections = false;
            }, 500);
          },

           change_base_map(base_map) {
            const index = Object.keys(this.baseMaps).indexOf(base_map);

            let layerControlElement = document.getElementsByClassName(
              "leaflet-control-layers"
            )[0];
            layerControlElement.getElementsByTagName("input")[index].click();
          },


        }

};
</script>

<style scoped>
@import "./assets/app.css";

</style>


