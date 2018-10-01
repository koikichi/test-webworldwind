<template>
    <div>
        <canvas id="canvasOne" width="1024" height="768"></canvas>
    </div>
</template>

<script>
import WorldWind from '@nasaworldwind/worldwind';
var wwd;
var placemarkLayer = new WorldWind.RenderableLayer('Placemark');
var placemarkAttributes = new WorldWind.PlacemarkAttributes(null);
placemarkAttributes.imageOffset = new WorldWind.Offset(
  WorldWind.OFFSET_FRACTION,
  0.3,
  WorldWind.OFFSET_FRACTION,
  0.0
);

placemarkAttributes.imageSource = 'http://localhost:8000/images/track.png';
placemarkAttributes.imageScale = 0.2;

export default {
  name: 'WebWorldWind',
  data() {
    return {
      wwd: null
    };
  },
  mounted() {
    console.log('web worldwind');
    wwd = new WorldWind.WorldWindow('canvasOne');
    wwd.addLayer(new WorldWind.BMNGOneImageLayer());
    wwd.addLayer(new WorldWind.BMNGLandsatLayer());
    wwd.addLayer(new WorldWind.CompassLayer());
    wwd.addLayer(new WorldWind.CoordinatesDisplayLayer(wwd));
    wwd.addLayer(new WorldWind.ViewControlsLayer(wwd));

    // var position = new WorldWind.Position(55.0, -106.0, 100.0);
    // var placemark = new WorldWind.Placemark(
    //   position,
    //   false,
    //   placemarkAttributes
    // );

    wwd.addLayer(placemarkLayer);
    // console.log('length : ', placemarkLayer.renderables.length);
    // placemarkLayer.addRenderable(placemark);

    // console.log(placemarkLayer.renderables);
    // placemarkLayer.renderables[0].position.latitude = -35;
    // placemarkLayer.renderables[0].position.longitude = 138;
    // console.log(placemarkLayer.renderables);

    var ws = new WebSocket('ws://localhost:8181');
    ws.onopen = function() {
      console.log('websocket is connected ...');
      // ws.send('connected');
    };
    ws.onclose = function() {
      console.log('websocket is disconnected ...');
      // ws.send('disconnected');
      // ws.close();
    };

    ws.onmessage = this.onWsMessage;
  },
  methods: {
    onWsMessage: function(ev) {
      var jsondata = JSON.parse(ev.data);
      console.log(jsondata);

      if (placemarkLayer.renderables.length == 0) {
        for (var i = 0; i < jsondata.length; i++) {
          var position = new WorldWind.Position(
            jsondata[i].latitude,
            jsondata[i].longitude,
            0
          );
          var placemark = new WorldWind.Placemark(
            position,
            false,
            placemarkAttributes
          );
          placemarkLayer.addRenderable(placemark);
        }
      } else {
        for (var i = 0; i < jsondata.length; i++) {
          placemarkLayer.renderables[i].position.latitude =
            jsondata[i].latitude;
          placemarkLayer.renderables[i].position.longitude =
            jsondata[i].longitude;
        }
        wwd.redraw();
      }
    }
  }
};
</script>

<style>
</style>