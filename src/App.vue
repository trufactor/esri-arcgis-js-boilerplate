<template lang="pug">
#map
</template>
<script>
import {Trufactor} from '@trufactor/core';
import {loadModules as loadESRI} from 'esri-loader';

export default {
  name: 'app',
  data(){
    return {
      db: new Trufactor({domain: 'demo'}),
      heatmapLayer: {
        title: 'TruFactor Boilerplate',
        copyright: 'TruFactor',
        objectIdField: 'ObjectId',
        fields: [
          {name: 'ObjectId', type: 'oid'},
          {name: 'date', type: 'string'},
          {name: 'total', type: 'integer'}
        ],
        popupTemplate: {
          content: '{date}: {total}'
        },
        renderer: {
          type: 'heatmap',
          field: 'total',
          colorStops: [
            { color: "rgba(63, 40, 102, 0)", ratio: 0 },
            { color: "#472b77", ratio: 0.083 },
            { color: "#4e2d87", ratio: 0.166 },
            { color: "#563098", ratio: 0.249 },
            { color: "#5d32a8", ratio: 0.332 },
            { color: "#6735be", ratio: 0.415 },
            { color: "#7139d4", ratio: 0.498 },
            { color: "#7b3ce9", ratio: 0.581 },
            { color: "#853fff", ratio: 0.664 },
            { color: "#a46fbf", ratio: 0.747 },
            { color: "#c29f80", ratio: 0.83 },
            { color: "#e0cf40", ratio: 0.913 },
            { color: "#ffff00", ratio: 1 }
          ],
          maxPixelIntensity: 1000000,
          minPixelIntensity: 0
        }
      },
      options: {
        zoom: 4,
        container: 'map',
        center: [-98.5, 39.8]
      }
    };
  },
  mounted(){

    // unfortunately as of Oct 31, 2019 ArcGIS still uses antiquated module
    // types instead of ES2015 imports.
    loadESRI([
      'esri/Map',
      'esri/layers/FeatureLayer',
      'esri/views/MapView',
      'esri/Graphic'
    ])
      .then(([Map, FeatureLayer, MapView, Graphic])=>{
        const map = new Map({basemap: 'gray'}),
              view = new MapView({map,...this.options});

        // this is a lifecycle hook called after the api caches the returned
        // data on the client
        this.db.afterGetData = ({data})=>{
          const source = data.features.reduce((features,feature)=>{
            return [
              ...features,
              ...feature.geometry.coordinates[0][0].reduce((points,point)=>{
                return [
                  ...points,
                  new Graphic({
                    geometry: {
                      type: 'point',
                      x: point[0],
                      y: point[1]
                    },
                    attributes: {
                      ObjectId: feature.properties.index,
                      date: feature.properties.date,
                      total: feature.properties.total
                    }
                  })
                ];
              },[])
            ];
          },[]);

          map.add(new FeatureLayer({source,...this.heatmapLayer}));
        };

        // go ahead and initialize the first api data call
        this.db.getData({
          zoom: view.zoom/22, //normalize level between 0 & 1
          query: view.center
        });

      })
      .catch(err=>{

        // ensure we catch any errors if the ESRI modules change and become inconsistent
        console.error(err);
      });
  }
};
</script>
<style lang="stylus" scoped>
#map
  height 100vh
  width 100vw
</style>
