<template>
  <div class="wrap">
    <div class="map" ref='map'></div>
    <div id="" :class="['map-title',isOverMap ? 'map-title-over' : '']" :style="mapTitlePositon">{{mapName}}</div>
    <div class="info" ref='info'>单击地图获取线条样式</div>
  </div>

</template>

<script>
  import Map3D from 'assets/js/Map3D.js'
  import * as THREE from 'assets/js/three'


  export default {
    name: 'map3d',
    title: '地图-线',

    data () {
      return {
        isOverMap: false,
        mapName: '',
        mapTitlePositon: {
          left: '800px',
          top: '600px'
        },
      }
    },
    methods: {

    },
    mounted(){
      self = this;

      $.getJSON("./static/mapdata/china.json", (geoData) => {

        // for(let i=0;i<1000;i++){
        //   let lat=Math.random()*100+50;
        //   let lon=Math.random()*20+10;
        //   this.mark.data.push({name:'test'+i,coord:[lat,lon],size:Math.random()*3})
        // }
        let opt = {
          name: 'map1',
          el: self.$refs.map,
          // debugger:true,
          geoData,
          area:{
            data:[],
            loadEffect:true,
            color:0x052659,
            lineColor:0x1481ba,
            lineOpacity:0.1,//线透明度
            opacity:.5,
          },
          mark:{
            data:[],
            color:0xffffff,
          },
          line:{
              data:[],
             
            hoverExclusive:false
          },

        }

        geoData.features.forEach((i)=>{
          //点数据
          opt.mark.data.push({name:i.properties.name,color:0xcaffff,coord:i.properties.cp,size:Math.random()*0.5});
          //线数据
          opt.line.data.push({fromName:i.properties.name,
            toName:'北京',
            haloDensity:10,
            hasHalo:true,
            hasHaloAnimate:false,
            spaceHeight:Math.random() * 20,
            color:0x0dbdff,
            haloColor:0x052659,
            haloSize:Math.random() * 10,
            coords:[i.properties.cp,[116.4551,40.2539]],
            value:Math.random()*7});
        })



        let map = new Map3D(opt);
        map.setCameraPosition({x:-2,y:-26,z:30},1000,300);

        map.addEventListener('mousedown', function (event) {
          let area = event.target;
          if(area.type==='Area'){

            let data=[];
            let color = Math.random() * 0xffffff;
            let haloColor = Math.random() * 0xffffff;
            let haloSize=Math.random()*10

           // console.log('color:'+color+',haloColor:'+haloColor)
            geoData.features.forEach((i)=>{
              //线数据
              data.push({
                fromName:i.properties.name,
                toName:area.name,
                haloDensity:10,
                spaceHeight:8,
                haloRunRate:0.05,
                color:color,
                haloSize:haloSize,
                haloColor:haloColor,
                coords:[i.properties.cp,area.userData.cp],
                value:Math.random()*1});

            })
            data.push({
              fromName:area.name,
              toName:area.name,
              haloDensity:10,
              spaceHeight:10,
              haloRunRate:0.5,
              color:color,
              haloColor:haloColor,
              haloSize:12,
              coords:[area.userData.cp,area.userData.cp],
              value:Math.random()*1});

            map.initLine({data});
            self.$refs.info.innerHTML = `haloSize: ${Math.floor(haloSize)}<br>haloColor:# ${new THREE.Color(haloColor).getHexString()}<br>color: #${new THREE.Color(color).getHexString()}`
            //map.mark.data=[];
           // map.mark.data.push({name:'台风-依安',coord:[116,23],value:2,color:0xff0000,size:4,value:12},);
           // map.initMark();
          }
        });

        map.addEventListener('mouseout', (event) => {
          let obj = event.target;
          self.isOverMap = false;

          if(obj.type==='Line')
          {

          }
        });
        var haloColorR,haloColorG,haloColorB;
        map.addEventListener('mouseover', (event) => {
          let obj = event.target;
          if(obj.type==='Line')
          {
            self.mapName = obj.userData.fromName +'-'+obj.userData.toName;

          }
          else
          {
            self.mapName = obj.userData.name + ':'+obj.userData.value;
          }
          self.isOverMap = true;
          self.mapTitlePositon.left = $(window).scrollLeft() + event.clientX + 20 + 'px';
          self.mapTitlePositon.top = $(window).scrollTop() + event.clientY + 20 + 'px';
        })

        map.addEventListener('resize', function (event) {
          console.log('map resize...');
        });
        // map.addCameraPosition({x:-30,y:15,z:15},1000)
        //map.setPosition({x:-13,y:0,z:35})
      });


    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  @import './../../assets/css/index.sass';

  .wrap {
    background-image: url('./../../assets/img/bg2.jpg');
  }

  .map {
    width: 100%;
    height: 100%;
  }

  .map-title {
    border-radius: 5px;
    border: 1px solid #ddd;
    color: #eee;
    background: rgba(0, 0, 0, .4);
    position: absolute;
    z-index: 2;
    opacity: 0;
    padding: 15px;
    text-align: center;
    transition: all .2s;
    pointer-events: none;
  }

  .info{
    position:fixed;
    bottom:90px;
    right:10px;
    border:1px solid rgba(255,255,255,.6);
    border-radius: 5px;
    padding: 10px;
    width: 300px;
    color:rgba(255,255,255,.8);
    background: rgba(0,0,0,.4);
  }

  .map-title-over {
    opacity: 1;
  }
</style>
