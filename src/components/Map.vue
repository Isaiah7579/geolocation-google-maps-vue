<template>
    <div>
        <div>
            <p><input type="file"  accept="image/*" name="image" id="file" @input="loadFile" style="display: none;"></p>
            <p><label for="file" style="cursor: pointer;">Upload Image</label></p>
            <p><img id="output" @click='imageClicked' width="200" /></p>
        </div>
        <div style="max-width: 800px; margin: 0 auto; display: flex; align-items: center; justify-content: space-between">
            <div>
                <h1>Your coordinates:</h1>
                <p>{{ myCoordinates.lat }} Latitude, {{ myCoordinates.lng }} Longitude</p>
            </div>
            <div>
                <h1>Map coordinates:</h1>
                <p>{{ mapCoordinates.lat }} Latitude, {{ mapCoordinates.lng }} Longitude</p>
            </div>
            <div>
                <h1>Photo coordinates:</h1>
                <p>{{ photoCoordinates.lat }} Latitude, {{ photoCoordinates.lng }} Longitude</p>
            </div>
        </div>
        <GmapMap
            :center="myCoordinates"
            :zoom="zoom"
            style="width:640px; height:360px; margin: 32px auto;"
            ref="mapRef"
            @dragend="handleDrag"
        ></GmapMap>
    </div>
</template>


<script>
    //Import EXIF script
    var EXIF = require('exif-js');

    export default {
        data() {
            return {
                map: null,
                myCoordinates: {
                    lat: 0,
                    lng: 0
                },
                photoCoordinates: {
                    lat: 0,
                    lng: 0
                },
                zoom: 7
            }
        },
        created() {
            // does the user have a saved center? use it instead of the default
            if(localStorage.center) {
                this.myCoordinates = JSON.parse(localStorage.center);
            } else {
                // get user's coordinates from browser request
                this.$getLocation({})
                    .then(coordinates => {
                        this.myCoordinates = coordinates;
                    })
                    .catch(error => alert(error));
            }

            // does the user have a saved zoom? use it instead of the default
            if(localStorage.zoom) {
                this.zoom = parseInt(localStorage.zoom);
            }
        },
        mounted() {
            // add the map to a data object
            this.$refs.mapRef.$mapPromise.then(map => this.map = map);
        },
        methods: {
            handleDrag() {
                // get center and zoom level, store in localstorage
                let center = {
                    lat: this.map.getCenter().lat(),
                    lng: this.map.getCenter().lng()
                };
                let zoom = this.map.getZoom();

                localStorage.center = JSON.stringify(center);
                localStorage.zoom = zoom;
            },
            onInput(e) {
                this.file_path = document.querySelector('myFile')
                console.log(this.file_path)
            } ,
            loadFile(e) {
                var image = document.getElementById('output');
                image.src = URL.createObjectURL(e.target.files[0]);
            },
            imageClicked(e) {
                // Retrieve coordinates from the EXIF data when the image is clicked
                let image = document.getElementById('output')
                EXIF.getData(image, function() {
                let myData = this;
                console.log(myData.exifdata);
                console.log("here4")
                let lat_data = myData.exifdata.GPSLatitude
                let long_data = myData.exifdata.GPSLongitude
                console.dir(this.photoCoordinates)
                this.photoCoordinates.lat = lat_data[0].numerator +lat_data[1].numerator/60 + lat_data[2]/3600;
                console.log(this.photoCoordinates.lat)
                this.photoCoordinates.lng = long_data[0].numerator +long_data[1].numerator/60 + long_data[2]/3600;
                console.log(this.photoCoordinates.long)
                });
            }
        },
        computed: {
            mapCoordinates() {
                if(!this.map) {
                    return {
                        lat: 0,
                        lng: 0
                    };
                }

                return {
                    lat: this.map.getCenter().lat().toFixed(4),
                    lng: this.map.getCenter().lng().toFixed(4)
                }
            }
        }
    }
</script>