<template>
    <div class="map-box" id="animation-box">
        <b-map
            :center="mapConfig.center"
            :zoom="mapConfig.zoom"
            :return-map="true"
            :context-menu="mapContextMenu"
            :location="true"
            :enable-scroll-wheel-zoom="true"
            :enable-high-accuracy="false"
            :map-ready="mapReady"
            :locate-succeed="locateSucceed"
            :locate-failed="locateFailed"
            @b-rightclick="mapRightClickCallback"
        >
            <b-search
                :position="{
                    x: '10',
                    y: '10'
                }"
            ></b-search>
            <b-infowindow
                :visible="location.visible"
                :size="{
                    width: 220,
                    height: 60
                }"
                :offset="{
                    x: 0,
                    y: -10
                }"
                :position.sync="location.position"
                @b-open="location.visible = true"
                @b-close="location.visible = false"
            >
                <h2>嘻嘻嘻 =,=</h2>
                <p>{{{ location.address }}}</p>
            </b-infowindow>
            <b-marker
                v-for="item in markerConfigList"
                :icon="item.config"
                :position.sync="item.position"
                :label="item.label"
                :enable-dragging="true"
                @b-click="markerClickCallback"
                @b-dragend="markerDragendCallback"
            ></b-marker>
            <b-polyline
                v-for="item in polylineConfigList"
                :points="item.points"
                :enable-editing="false"
                :style="{
                    strokeColor: '#32b1fb',
                    strokeWeight: 4,
                    strokeOpacity: 1
                }"
                @b-lineupdate="lineUpdate"
            ></b-polyline>

            <!-- controller-section -->
            <b-fullscreen
                :position="{
                    x: '-10',
                    y: '10'
                }"
            ></b-fullscreen>
            <b-box
                :position="{
                    x: '-0',
                    y: '-0'
                }"
            >
                <button @click="addMarkers" type="button" name="button">添加五百个随机点</button>
            </b-box>
            <b-box
                :position="{
                    x: '-0',
                    y: '-20'
                }"
            >
                <button @click="changeLabel" type="button" name="button">修改label</button>
            </b-box>
            <b-box
                :position="{
                    x: '-0',
                    y: '-40'
                }"
            >
                <button @click="toggleInfoWindow" type="button" name="button">toggleInfoWindow</button>
            </b-box>
            <b-box
                :position="{
                    x: '-0',
                    y: '-60'
                }"
            >
                <button @click="addLine" type="button" name="button">添加Lines</button>
            </b-box>
            <b-locate
                :position="{
                    x: '-10',
                    y: '40'
                }"
                :enable-high-accuracy="true"
                :locate-succeed="locateSucceed"
                :locate-failed="locateFailed"
            ></b-locate>
            <b-zoom
                :position="{
                    x: '-10',
                    y: '70'
                }"
            ></b-zoom>
            <b-type
                :position="{
                    x: '-10',
                    y: '130'
                }"
            ></b-type>
        </b-map>
        <!--vue-baidu-map end-->
    </div>
</template>
<script>
    import { load, map, mapMarker, mapPolyline, controllerBox, infoWindow, fullScreenController, localSearchController, locateController, zoomController, toggleTypeController } from "../lib/index.js";
    import { poltMarkerConfig } from "../lib2/componentConfig/markerConfig";

    export default {
        data () {
            return {
                // 地图配置信息
                mapConfig: {                                // map 组件配置项
                    zoom: 5,
                    center: {
                        lat: 36,
                        lng: 103
                    }
                },
                // polyline 配置信息
                polylineConfig: {
                    strokeColor: "#FF0000",
                    strokeWeight: 5,
                    strokeOpacity: 1
                },
                location: {
                    position: {
                        lat: 0,
                        lng: 0
                    },
                    visible: false,
                    address: `<h4 style="font-size: 16px; color: #444;">你的位置</h4>`,
                    succeedCallback: undefined,
                    failedCallback: function () {
                        console.log( "error" );
                    }
                },
                poltMarker: [
                    {
                        position: {
                            lng: 118,
                            lat: 31
                        },
                        visible: true
                    },
                    {
                        position: {
                            lng: 100,
                            lat: 32
                        },
                        visible: true
                    },
                ],
                markerConfigList: [],                       // 需要绘制到 map 组件上的 marker 组件配置项
                polylineConfigList: [],                     // 需要绘制到 map 组件上的 polyline 组件配置项
                markerContextMenu: [],                      // marker 右键菜单配置项
                mapContextMenu: [],                         // map 右键菜单配置项
                $map: undefined,                            // 地图组件对象
            }
        },
        ready () {
            let _this = this;
            // 请使用自己的 key，此 key 为个人账号，每日使用次数有限
            load( { key: "hPKNosYElVbALMK2ySnseejdXN7y8nqT", version: "2.0" } );
            // map 右键菜单
            _this.mapContextMenu = [
                {
                    text: "创建我的第一个marker",
                    callback: function ( e ) {
                        let len = _this.markerConfigList.length;
                        let point = {
                            config: poltMarkerConfig,
                            position: {
                                lng: e.lng,
                                lat: e.lat
                            },
                            label: {
                                text: "&nbsp;" + ( len + 1 ).toString() + "&nbsp;",
                                offset: {
                                    x: 28,
                                    y: -2
                                }
                            },
                            infoWindowId: 123,
                            cid: Math.random().toString()
                        };
                        _this.markerConfigList.push( point );
                        len = _this.markerConfigList.length;
                        if ( len > 1 ) {
                            let prev = _this.markerConfigList[len-2].position;
                            let next = _this.markerConfigList[len-1].position;
                            _this.polylineConfigList.push({
                                points: [prev, next],
                            });
                        }
                        _this.mapContextMenu[0].text = "创建途经点";
                    }
                },
                {
                    text: "清空地图",
                    callback: function ( e ) {
                        let { markerConfigList, polylineConfigList } = _this;
                        markerConfigList.splice( 0, markerConfigList.length );
                        polylineConfigList.splice( 0, polylineConfigList.length );
                        _this.mapContextMenu[0].text = "创建起点";
                    }
                }
            ];

            // // marker 右键菜单
            this.markerContextMenu = [
                {
                    text: "删除途经点",
                    callback: function ( e, ee, marker ) {
                        // this 指向marker对象
                        console.log( e );
                        console.log( ee );
                        console.log( marker );

                        // let $marker = _this.getVueComponent( marker );
                        // let index = _this.getMarkerIndex( $marker );
                        // if ( index !== -1 ) {
                        //     if ( index ===  0 ) {
                        //         // 删除第一个途经点
                        //         _this.markerConfigList.shift();
                        //         _this.polylineConfigList.shift();
                        //     } else if ( index === _this.markerConfigList.length - 1 ) {
                        //         // 删除最后一个途经点
                        //         _this.markerConfigList.pop();
                        //         _this.polylineConfigList.pop();
                        //     } else {
                        //         // 删除中间点并设置新的路径
                        //         _this.markerConfigList.splice( index, 1 );
                        //     }
                        //     if ( _this.markerConfigList.length === 0 ) {
                        //         _this.mapContextMenu[0].text = "创建起点";
                        //     }
                        // } else {
                        //     alert( "删除失败，不存在该标记点" )
                        // }
                    }
                }
            ];
            //
            // this.location.succeedCallback = function( res ) {
            //     _this.location.position = res.point;
            //     _this.location.visible = true;
            //     _this.location.address += `<p style="margin-top: 10px;font-size:14px;color:#333;">${res.address.city}-${res.address.district}-${res.address.street}</p>`;
            // }
        },
        methods: {
            lineUpdate (a,b,c) {
                // console.log(a);
                // console.log(b);
                // console.log(c);
            },

            mapReady () {
                console.log("map ready");
            },

            locateSucceed (e) {
                alert(e);
            },

            locateFailed (e) {
                alert(e);
            },

            toggleInfoWindow () {
                this.location.visible = !this.location.visible;
            },

            addLine () {
                let _this = this;
                let count = 500;
                while ( count > 0 ) {
                    let prev = {
                        lng: 130*Math.random(),
                        lat: 30*Math.random()
                    };
                    let next = {
                        lng: 130*Math.random(),
                        lat: 30*Math.random()
                    };
                    this.polylineConfigList.push({
                        points: [prev, next],
                    });
                    count--;
                }
            },

            addMarkers (){
                let _this = this;
                let count = 500;
                while ( count > 0 ) {
                    this.markerConfigList.push({
                        config: poltMarkerConfig,
                        position: {
                            lng: 130*Math.random(),
                            lat: 30*Math.random()
                        },
                        label: {
                            text: "==&nbsp;" + ( _this.markerConfigList.length + 1 ).toString() + "&nbsp;",
                            offset: {
                                x: 28,
                                y: -2
                            }
                        },
                        infoWindowId: 123,
                        cid: Math.random().toString()
                    });
                    count--;
                }
            },

            changeLabel () {
                this.markerConfigList.map( ( item, index, arr ) => {
                    item.label =  {
                        text: "&nbsp;" + ( Math.random() ).toString() + "&nbsp;",
                        offset: {
                            x: 28,
                            y: -2
                        }
                    };
                });
            },

            mapRightClickCallback () {

            },
            /** marker 左键点击事件 */
            markerClickCallback: function ( e, $marker ) {
                // let index =  this.getMarkerIndex( $marker );
                // let infowindow = this.$map.$data.infoWindowList[$marker.bindInfoWindow].componentObj;
                console.log(e);
                console.log($marker);
                this.location.position.lng = e.point.lng;
                this.location.position.lat = e.point.lat;
                this.location.visible = true;
            },

            /** marker 拖动结束事件 */
            markerDragendCallback: function ( e, $marker ) {
                let _this = this;
                let index = _this.getMarkerIndex( $marker );
                /** NOTICE: don't make a new position , maybe we do not need this */
                // _this.markerConfigList[index].position.lat = e.point.lat;
                // _this.markerConfigList[index].position.lng = e.point.lng;
                // _this.polylineConfigList[index-1].points.splice(1, 1, e.point);
            },

            /** 获取 marker 在 markerArr 中的索引值 */
            getMarkerIndex: function ( $marker ) {
                let _index = -1;
                if ( $marker ) {
                    this.markerConfigList.map( function ( item, index ) {
                        // 判断 marker 组件的 position 配置项是否与 item.position 相同
                        if ( item.position === $marker.position ) {
                            _index = index;
                        }
                    });
                }
                return _index;
            },

            /** 获取 overlay 所在的 vue 组件 */
            getVueComponent: function ( overlayObj ) {
                let _this = this;
                let $component = undefined;
                if ( _this.$map && _this.$map.$children ) {
                    for ( let i = 0; i < _this.$map.$children.length; i++ ) {
                        if ( _this.$map.$children[i].mapComponentObj === overlayObj ) {
                            $component = _this.$map.$children[i];
                            break;
                        }
                    }
                }
                return $component;
            },
        },
        events: {
            "bmap-ready": function ( $map ) {
                if ( $map )  {
                    this.$map = $map;
                }
            }
        },
        components: {
            "b-map": map,
            "b-marker": mapMarker,
            "b-box": controllerBox,
            // "b-poltmarker": mapMarker,
            "b-polyline": mapPolyline,
            "b-infowindow": infoWindow,
            "b-zoom": zoomController,
            "b-type": toggleTypeController,
            "b-search": localSearchController,
            "b-fullscreen": fullScreenController,
            "b-locate": locateController,
        }
    }
</script>
<style lang="sass-loader" scoped>
    .map-box {
        position: relative;
        width: 100%;
        height: 100%;
        overflow: hidden;
    }
</style>
<style lang="sass-loader">
    html,
    body {
        width: 100%;
        height: 100%;
        * {
            padding: 0;
            margin: 0;
        }
    }
    #app {
        margin-left: 50px;
        margin-top: 50px;
        height: 90%;
        width: 90%;
    }
</style>
