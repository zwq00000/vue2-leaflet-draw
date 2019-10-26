<template>

</template>

<script>
import L from "leaflet";
import {
  OptionsMixin,
  ControlMixin,
  propsBinder,
  optionsMerger,
  findRealParent
} from "vue2-leaflet";
import "leaflet-draw/dist/leaflet.draw-src.css";
import LDraw from "leaflet-draw/dist/leaflet.draw-src";
import drawLocal from "../res/drawLocal.cn.jsonc";

L.drawLocal = drawLocal;

export default {
  name: "l-control-draw",
  mixins: [ControlMixin, OptionsMixin],
  model: {
    prop: "value",
    event: "change"
  },
  data() {
    return {
      ready: false
    };
  },
  props: {
    position: {
      type: String,
      description: "工具栏显示位置,topleft/topright/bottomleft/bottomright",
      default: "bottomleft"
    },
    drawPolyline: {
      type: Boolean,
      description: "是否显示折线绘制工具",
      default: true
    },
    drawPolygon: {
      type: Boolean,
      description: "是否显示多边形绘制工具",
      default: true
    },
    drawMarker: {
      type: Boolean,
      description: "是否显示标记绘制工具",
      default: true
    },
    drawCircleMarker: {
      type: Boolean,
      description: "是否显示圆形标记绘制工具",
      default: true
    },
    drawCircle: {
      type: Boolean,
      description: "是否显示圆形工具",
      default: true
    },
    drawRectangle: {
      type: Boolean,
      description: "是否显示矩形绘制工具",
      default: true
    },
    value: {
      //geoJSON array
      type: Array,
      default: () => []
    }
  },
  mounted() {
    this._drawLayers = L.featureGroup();
    const options = optionsMerger(
      {
        ...this.controlOptions,
        position: this.position,
        draw: {
          polyline: this.drawPolyline,
          polygon: this.drawPolygon,
          rectangle: this.drawRectangle,
          circle: this.drawCircle,
          marker: this.drawMarker,
          circlemarker: this.drawCircleMarker
        },
        edit: {
          featureGroup: this._drawLayers
        }
      },
      this
    );
    this.mapObject = new L.Control.Draw(options);
    propsBinder(this, this.mapObject, this.$options.props);
    this.parentContainer = findRealParent(this.$parent);
    this.mapObject.addTo(this.parentContainer.mapObject);
    this.parentContainer.mapObject.addLayer(this._drawLayers);
    this.init();
    this.initEvents();
    this.ready = true;
    this.$nextTick(() => {
      window._ldraw = this.mapObject;
      this.$emit("ready", this.mapObject);
    });
  },
  beforeDestroy() {
    this.parentContainer.mapObject.off(L.Draw.Event.CREATED, this._onCreated);
    this.parentContainer.mapObject.off(
      L.Draw.Event.DELETESTOP,
      this._onChanged
    );
    this.parentContainer.mapObject.off(L.Draw.Event.DRAWSTOP, this._onChanged);
    this.parentContainer.mapObject.off(L.Draw.Event.EDITSTOP, this._onChanged);
    this.parentContainer.mapObject.removeLayer(this._drawLayers);
  },
  render() {
    return null;
  },
  methods: {
    initEvents() {
      let map = this.parentContainer.mapObject;
      map.on(L.Draw.Event.CREATED, this._onCreated);
      map.on(L.Draw.Event.DELETESTOP, this._onChanged);
      map.on(L.Draw.Event.DRAWSTOP, this._onChanged);
      map.on(L.Draw.Event.EDITSTOP, this._onChanged);
    },
    init() {
      if (this.value && this.value.length > 0) {
        //geoJSON array
        this.value.forEach(j => {
          //geoJSON array
          this._drawLayers.addLayer(L.geoJSON(j));
        });
      }
    },
    getGeoJson(layer) {
      if (layer.options) {
        let feature = layer.toGeoJSON();
        let layerType = layer.options.type;
        switch (layerType) {
          case "polyline":
          case "polygon":
          case "rectangle":
          case "circle":
            feature.properties = { ...layer.options };
            break;
          case "marker":
            feature.properties = { type: layerType };
            break;
          case "circlemarker":
            feature.properties = { type: layerType };
            break;
          default:
            feature.properties = { ...layer.options };
            break;
        }
        return feature;
      } else {
        return layer.toGeoJSON();
      }
    },
    _onCreated(e) {
      var type = e.layerType,
        layer = e.layer;
      layer.options.type = type;
      this._drawLayers.addLayer(layer);
      this._workingLayer = layer;
    },
    _onChanged(e) {
      let value = []; //geoJSON array
      this._drawLayers.eachLayer(l => {
        value.push(this.getGeoJson(l)); //geoJSON array
      });
      this.$emit("change", value); //geoJSON array
    }
  }
};
</script>