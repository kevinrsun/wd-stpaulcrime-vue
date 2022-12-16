<script>
import $ from 'jquery'
import crimesResult from './components/crimesResult.vue'

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [],
            incidents: [],
            leaflet: {
                map: null,
                center: {
                    lat: 44.955139,
                    lng: -93.102222,
                    address: ""
                },
                zoom: 12,
                bounds: {
                    nw: { lat: 45.008206, lng: -93.217977 },
                    se: { lat: 44.883658, lng: -92.993787 }
                },
                neighborhood_markers: [
                    { location: [44.942068, -93.020521], marker: null },
                    { location: [44.977413, -93.025156], marker: null },
                    { location: [44.931244, -93.079578], marker: null },
                    { location: [44.956192, -93.060189], marker: null },
                    { location: [44.978883, -93.068163], marker: null },
                    { location: [44.975766, -93.113887], marker: null },
                    { location: [44.959639, -93.121271], marker: null },
                    { location: [44.947700, -93.128505], marker: null },
                    { location: [44.930276, -93.119911], marker: null },
                    { location: [44.982752, -93.147910], marker: null },
                    { location: [44.963631, -93.167548], marker: null },
                    { location: [44.973971, -93.197965], marker: null },
                    { location: [44.949043, -93.178261], marker: null },
                    { location: [44.934848, -93.176736], marker: null },
                    { location: [44.913106, -93.170779], marker: null },
                    { location: [44.937705, -93.136997], marker: null },
                    { location: [44.949203, -93.093739], marker: null }
                ]
            }
        };
    },
    components: {
        crimesResult
    },
    methods: {
        viewMap(event) {
            this.view = 'map';
        },

        viewNewIncident(event) {
            this.view = 'new_incident';
        },

        viewAbout(event) {
            this.view = 'about';
        },

        getJSON(url) { //Should use this to get the data??? 
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'json',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },

        uploadJSON(method, url, data) { //should use this for the new incident input
            return new Promise((resolve, reject) => {
                $.ajax({
                    type: method,
                    url: url,
                    contentType: 'application/json; charset=utf-8',
                    data: JSON.stringify(data),
                    dataType: 'text',
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },

        filterCrimes(data) {
            //take the input from the filters and apply them to the data...
            //construct a url to pass to getJSON??
            if(this.codes !== ''){ //Add the codes into the codes array??

            }
            if(this.neighborhoods !== ''){ //Add neighborhoods to neighborhoods array??

            }
            if(this.incidents !== ''){ //Add incidents to incidents array??

            }


        },

        newLocation(data){
            //Take the address or lat/long and update the map for new location...
            if(this.leaflet.center.address !== ''){

            } else if(this.leaflet.center.lat !== '' && this.leaflet.center.long !== ''){

            }

        },

        newIncident(data){
            //put together the data for the new incident then pass as json??? 

            //Check if any of the inputs are null/empty


        }

    },
    mounted() { 
        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);

        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });
    }
}
</script>

<template>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <p :class="'cell small-4 ' + ((view === 'map') ? 'selected' : 'unselected')" @click="viewMap">Map</p>
            <p :class="'cell small-4 ' + ((view === 'new_incident') ? 'selected' : 'unselected')"
                @click="viewNewIncident">New Incident</p>
            <p :class="'cell small-4 ' + ((view === 'about') ? 'selected' : 'unselected')" @click="viewAbout">About</p>
        </div>
    </div>
    <div v-show="view === 'map'">
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <div id="leafletmap" class="cell auto"></div>

                <h1 class="cell large-12" style="font-size: 42px">Coordinates:</h1>
                <!-- input box to take address/lat/longs and go button -->
                <!-- input box/s should be updated with new location when map is changed (pan/zoom) -->
                <!-- Clamp input values if lat/long is outside of St. Paul's bounding box -->
                <div class="cell large-12">
                    <label for="address" style="display: inline-block">Address:</label> &nbsp;
                    <input type="text" id="address" style="width: 400px; display: inline-block"> <br>
                    <!-- try to use a auto-filling one?? -->
                    <label for="lat" style="display: inline-block">Lat:</label> &nbsp;
                    <input type="text" id="lat" style="width: 400px; display: inline-block"> &nbsp;
                    <!-- should lat/long be 1 or 2 input boxes??? -->
                    <label for="long" style="display: inline-block">Long:</label> &nbsp;
                    <input type="text" id="long" style="width:400px; display: inline-block"> &nbsp; &nbsp;
                    <button type="button" id="go-button" @click="newLocation" style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Go</button>
                </div>


                <h1 class="cell large-12" style="font-size: 42px">Filters:</h1>
                <!-- incident_type: list of checkboxes per incident_type -->

                <div class="cell large-12">
                    <p style="text-decoration: underline; font-weight: bold;">Incident Type: </p>
                    <input type="checkbox" id="incident1" name="incident1" value="Homicide">
                    <label for="incident1">Homicide</label>
                    <input type="checkbox" id="incident2" name="incident2" value="Rape">
                    <label for="incident2">Rape</label>
                    <input type="checkbox" id="incident3" name="incident3" value="Robbery">
                    <label for="incident3">Robbery</label>

                    <!-- DO WE HAVE A LIST OF ALL INCIDENT TYPES??? -->
                </div>

                <!-- neighborhood_name: list of checkboxes per neighborhood_name -->
                <div class="cell large-12">
                    <p style="text-decoration: underline; font-weight: bold;">Neighborhood Name: </p>
                    <input type="checkbox" id="neighborhood1" name="neighborhood1" value="Conway/Battlecreeek/Highwood">
                    <label for="neighborhood1">Conway/Battlecreek/Highwood</label>
                    <input type="checkbox" id="neighborhood2" name="neighborhood2" value="Greater East Side">
                    <label for="neighborhood2">Greater East Side</label>
                    <input type="checkbox" id="neighborhood3" name="neighborhood3" value="West Side">
                    <label for="neighborhood3">West Side</label>
                    <input type="checkbox" id="neighborhood4" name="neighborhood4" value="Dayton's Bluff">
                    <label for="neighborhood4">Dayton's Bluff</label>
                    <input type="checkbox" id="neighborhood5" name="neighborhood5" value="Payne/Phalen">
                    <label for="neighborhood5">Payne/Phalen</label>
                    <input type="checkbox" id="neighborhood6" name="neighborhood6" value="North End">
                    <label for="neighborhood6">North End</label>
                    <input type="checkbox" id="neighborhood7" name="neighborhood7" value="Thomas/Dale(Frogtown)">
                    <label for="neighborhood7">Thomas/Dale(Frogtown)</label>
                    <input type="checkbox" id="neighborhood8" name="neighborhood8" value="Summit/University">
                    <label for="neighborhood8">Summit/University</label>
                    <input type="checkbox" id="neighborhood9" name="neighborhood9" value="West Seventh">
                    <label for="neighborhood9">West Seventh</label>
                    <input type="checkbox" id="neighborhood10" name="neighborhood10" value="Como">
                    <label for="neighborhood10">Como</label>
                    <input type="checkbox" id="neighborhood11" name="neighborhood11" value="Hamline/Midway">
                    <label for="neighborhood11">Hamline/Midway</label>
                    <input type="checkbox" id="neighborhood12" name="neighborhood12" value="St. Anthony">
                    <label for="neighborhood12">St. Anthony</label>
                    <input type="checkbox" id="neighborhood13" name="neighborhood13" value="Union Park">
                    <label for="neighborhood13">Union Park</label>
                    <input type="checkbox" id="neighborhood14" name="neighborhood14" value="Macalester/Groveland">
                    <label for="neighborhood14">Macalester/Groveland</label>
                    <input type="checkbox" id="neighborhood15" name="neighborhood15" value="Highland">
                    <label for="neighborhood15">Highland</label>
                    <input type="checkbox" id="neighborhood16" name="neighborhood16" value="Summit Hill">
                    <label for="neighborhood16">Summit Hill</label>
                    <input type="checkbox" id="neighborhood17" name="neighborhood17" value="Capitol River">
                    <label for="neighborhood17">Capitol River</label>
                </div>

                <!-- date range: select a start and end date (only show crimes between those dates) -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Dates: </p>
                    <label for="start-date" style="display: inline-block">Start Date:</label> &nbsp;
                    <input type="text" id="start-date" style="width: 300px; display:inline-block"> <br>
                    <label for="end-date" style="display:inline-block">End Date:</label> &nbsp;
                    <input type="text" id="end-date" style="width: 300px; display: inline-block">

                </div>

                <!-- time range: select a start and end time (only show crimes that occurred between those times of day) -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Times: </p>
                    <label for="start-time" style="display: inline-block">Start Time:</label> &nbsp;
                    <input type="text" id="start-time" style="width: 300px; display: inline-block"> <br>
                    <label for="end-time" style="display: inline-block">End Time:</label> &nbsp;
                    <input type="text" id="end-time" style="width: 300px; display: inline-block">

                </div>

                <!-- max incidents: select maximum number of incidents to retrieve / show -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Max Incidents: </p>
                    <input type="text" id="max-incidents" style="width: 200px">

                </div>

                <!--  submit  button -->
                <div class="cell large-12" style="text-align: center">
                    <button type="button" @click="filterCrimes"
                        style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Submit</button>
                </div>

                <br><br><br><br>


                <!-- Will want to build a table here to show the crimes like project 3 did, but using the way spotify-starter does -->
                <crimesResult :result_array="incidents"/>


                <br><br>
            </div>
        </div>
    </div>
    <div v-if="view === 'new_incident'">
        <!-- Replace this with your actual form: can be done here or by making a new component -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell auto" style="text-align: center">New Incident Form</h1>

                <!-- Create a user input form for users to add a new crime incident to the database (i.e. submit the PUT request) -->
                <!-- Ensure all fields are filled out before submitting request, otherwise show some error message -->

                <!-- THINK ABOUT IF WE WANT TEXT BOXES FOR DROPDOWN MENUS FOR SOME OF THESE... -->
                <div class="cell large-12" style="text-align: center">
                    <label for="case-number" style="display: inline-block">Case Number:</label> &nbsp;
                    <input type="text" id="case-number" style="width: 300px; display: inline-block"> <br>

                    <label for="date" style="display: inline-block">Date:</label> &nbsp; 
                    <input type="text" id="date" style="width: 300px; display: inline-block"> <br> <!-- show the correct format as faded text in box?? -->

                    <label for="time" style="display: inline-block">Time:</label> &nbsp; 
                    <input type="text" id="time" style="width: 300px; display: inline-block"> <br> <!-- show the correct format as faded text in box?? -->

                    <label for="incident-code" style="display: inline-block">Code:</label> &nbsp;
                    <input type="text" id="incident-code" style="width: 300px; display: inline-block"> <br>
                    <!-- do we want to have the code and the incident type linked somehow?? like a dropdown for incident type that auto populates code? -->
                    <label for="incident-type" style="display:inline-block">Incident Type:</label> &nbsp;
                    <input type="text" id="incident-type" style="width: 300px; display: inline-block"> <br>

                    <label for="police-grid" style="display: inline-block">Police Grid:</label> &nbsp;
                    <input type="text" id="police-grid" style="width: 300px; display: inline-block"> <br>

                    <label for="neighborhood-id" style="display: inline-block">Neighborhood ID:</label> &nbsp;
                    <input type="text" id="neighborhood-id" style="width: 300px; display: inline-block"> <br>

                    <label for="block" style="display: inline-block">Block:</label> &nbsp;
                    <input type="text" id="block" style="width: 300px; display: inline-block"> <br>



                    <button type="button" @click="newIncident" style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Submit</button>

                </div>



            </div>
        </div>
    </div>
    <div v-if="view === 'about'">
        <!-- Replace this with your actual about the project content: can be done here or by making a new component -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <div class="cell large-12">
                    <h1>About the Project</h1>
                </div>

                <h1 class="cell large-12" style="font-size: 40px">Team:</h1>
                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Jack Landwer</p>
                    <!-- photo and short bio-->
                    <img src="images/jackPic.jpeg" alt="Jack" style="width: 250px">
                    <p>Senior at St. Thomas majoring in Computer Science. In my free time I like to snowboard, golf, and
                        play soccer.</p>

                </div>

                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Kevin</p>
                    <!-- photo and short bio-->
                    <img src="images/" alt="Kevin" style="width: 250px">

                </div>

                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Sam</p>
                    <!-- photo and short bio-->
                    <img src="images/" alt="Sam" style="width: 250px">

                </div>

                <!-- maybe add some horizontal lines to separate the sections??-->

                <h1 class="cell large-12" style="font-size: 40px">Description of tools:</h1>
                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Video of the application:</h1>
                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Six interesting findings that you discovered using
                    your application:</h1>
                <div class="cell large-12">

                </div>






            </div>
        </div>
    </div>
</template>

<style>
#leafletmap {
    height: 500px;
}

.selected {
    background-color: rgb(10, 100, 126);
    color: white;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}

.unselected {
    background-color: rgb(200, 200, 200);
    color: black;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
</style>
