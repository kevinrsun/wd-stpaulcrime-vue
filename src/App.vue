<script>
import $ from 'jquery'
import CrimesResult from './components/CrimesResult.vue'

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [],
            incidents: [],
            filterNeighborhood: [],
            filterIncidentType: [],
            filterDates: [],
            filterTimes: [],
            filterMaxIncidents: 1000,
            selectIncidentAddress: '',
            selectIncidentLat: '',
            selectIncidentLong: '',
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
        CrimesResult
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

        getJSON(url) {
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

            let incidentReq = "http://localhost:8000/incidents";
            let codesReq = "http://localhost:8000/codes";
            let neighborhoodsReq = "http://localhost:8000/neighborhoods";
            let query = "";

            if (this.filterNeighborhood.length > 0) {
                query = "?neighborhood=" + this.filterNeighborhood[0];

                for (let i = 1; i < this.filterNeighborhood.length; i++) {
                    query = query + "," + this.filterNeighborhood[i];
                }
            }
            if (this.filterIncidentType > 0) {
                if (query !== "") {
                    query = query + "&code=";
                } else {
                    query = "?code=";
                }

                let HomeicideCodes = "100,110,120";
                let AssaultCodes = "400,410,411,412,420,421,422,430,431,432,440,441,442,450,451,\
                                    452,453,810,861,862,863"; // Code 810 was spelt Asasult
                let RapeCodes = "210,220";
                let TheftBurglaryRobberyCodes = "300,311,312,313,314,321,322,323,324,331,333,\
                                                334,341,342,343,344,351,352,353,354,361,363,\
                                                364,371,372,373,374,500,510,511,513,515,516,\
                                                520,521,523,525,526,530,531,533,535,536,540,\
                                                541,543,545,546,550,551,553,555,556,560,561,\
                                                563,565,566,600,603,611,612,613,621,622,623,\
                                                630,631,632,633,640,641,642,643,651,652,653,\
                                                661,662,663,671,672,673,681,682,683,691,692,\
                                                693,700,710,711,712,720,721,722,730,731,732";

                let PropertyDamageCodes = "900,901,903,905,911,913,915,921,922,923,925,931,\
                                            933,941,942,951,961,971,972,975,981,982,1400,1401,\
                                            1410,1415,1416,1420,1425,1426,1430,1435,1436";

                let NarcoticsCodes = "1800,1810,1811,1812,1813,1814,1815,1820,1822,1823,1824,\
                                        1825,1830,1835,1840,1841,1842,1843,1844,1845,1850,1855,\
                                        1860,1865,1870,1880,1885";

                let OtherCodes = "614,2619,3100,9954,9959,9986";

                let codesArr = [
                    HomeicideCodes,
                    AssaultCodes,
                    RapeCodes,
                    TheftBurglaryRobberyCodes,
                    PropertyDamageCodes,
                    NarcoticsCodes,
                    OtherCodes
                ];

                for (let i = 0; i < this.filterIncidentType.length; i++) {
                    if (i == 0) {
                        query = query + codesArr[this.filterIncidentType[i]];
                    } else {
                        query = query + "," + codesArr[this.filterIncidentType[i]];
                    }
                }
            }
            if (this.filterDates.length > 0) { //check for date filters
                if (query !== "") {
                    if (this.filterDates[0]) {
                        query = query + "&start_date=" + this.filterDates[0];
                    }
                } else {
                    if (this.filterDates[0]) {
                        query = query + "?start_date=" + this.filterDates[0];
                    }
                }
                if (query !== "") {
                    if (this.filterDates[1]) {
                        query = query + "&end_date=" + this.filterDates[1];
                    }
                } else {
                    if (this.filterDates[1]) {
                        query = query + "?end_date=" + this.filterDates[1];
                    }
                }
            }
            if (this.filterTimes.length > 0) { //check for time filters
                if (query !== "") {
                    if (this.filterTimes[0]) {
                        query = query + "&start_time=" + this.filterTimes[0];
                    }
                } else {
                    if (this.filterTimes[0]) {
                        query = query + "?start_time=" + this.filterTimes[0];
                    }
                }
                if (query !== "") {
                    if (this.filterTimes[1]) {
                        query = query + "&end_time=" + this.filterTimes[1];
                    }
                } else {
                    if (this.filterTimes[1]) {
                        query = query + "?end_time=" + this.filterTimes[1];
                    }
                }
            }
            if (this.filterMaxIncidents !== 1000) { //check for max incident filter, 1000 by default
                console.log(this.filterMaxIncidents[0])
                if (query !== "") {
                    query = query + "&limit=" + this.filterMaxIncidents;
                } else {
                    query = query + "?limit=" + this.filterMaxIncidents;
                }

            }

            incidentReq = incidentReq + query;
            console.log("incidentReq: " + incidentReq);

            // Hitting Submit button for filter triggers API request
            Promise.all([this.getJSON(incidentReq), this.getJSON(codesReq), this.getJSON(neighborhoodsReq)])
                .then((data) => {
                    console.log(data);

                    this.incidents = data[0];
                    this.codes = data[1];
                    this.neighborhoods = data[2];

                    // Update incidents to use neighborhood_name rather than neighborhood_number, and incident_type rather than code
                    this.incidents.map((incident) => {
                        // Change code to incident_type
                        for (let i = 0; i < this.codes.length; i++) {
                            if (incident.code == this.codes[i].code) {
                                incident.code = this.codes[i].type;
                            }
                        }
                        // Change neighborhood_number to neighborhood_name
                        for (let i = 0; i < this.neighborhoods.length; i++) {
                            if (incident.neighborhood_number == this.neighborhoods[i].id) {
                                incident.neighborhood_number = this.neighborhoods[i].name;
                            }
                        }
                    });
                })
                .then((err) => {
                    console.log(err);
                });
        },

        newLocation(data) {
            //Take the address or lat/long and update the map for new location...
            let getAddress = "https://nominatim.openstreetmap.org/reverse?lat=<value>&lon=<value>&<params>";


            if (this.leaflet.center.address !== '') {

            } else if (this.leaflet.center.lat !== '' && this.leaflet.center.long !== '') {

            }

        },

        newIncident(data) {
            //put together the data for the new incident then pass to uploadJSON??? 

            //Check if any of the inputs are null/empty before making the PUT request



        },

        selectButtonClicked(data, case_number) {
            //how do we get the data for that row...
            //set the lat and long, kinda like top line in mounted() i think...
            console.log(case_number);
            //when have the case number, loop through the cases and get the lat/long for the case for the case number

            let cnAddress = '';
            let cnLat = '';
            let cnLong = '';

            if(this.selectIncidentAddress !== 0){

            }
            if(this.selectIncidentLat !== ''){

            }
            if(this.selectIncidentLong !== ''){

            }

            for (let i = 0; i < this.codes.length; i++) {
                if (case_number == this.codes[i].code) {
                    //set lat/long to variable
                    cnAddress = this.codes[i].address;
                    cnLat = this.codes[i].lat;
                    cnLong = this.codes[i].long;
                    //Should I just break out right here..??
                }
            }
            //Want to drop a pin on the location and set the view on it
            this.leaflet.center.lat = cnLat;
            this.leaflet.center.lng = cnLong;
            this.leaflet.center.address = cnAddress;

            this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);

            //drop the pin on the incident location. New marker color, has popup with date, time, incident, and delete button. REPLACE X's in address #
             


        },

        deleteButtonClicked(data, case_number) {
            console.log(case_number);



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

    <!-- MAP PAGE VIEW -->
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
                    <input type="text" id="address" v-model="selectIncidentAddress"
                        style="width: 400px; display: inline-block" /> <br>
                    <!-- try to use a auto-filling one?? -->
                    <label for="lat" style="display: inline-block">Lat:</label> &nbsp;
                    <input type="text" id="lat" v-model="selectIncidentLat"
                        style="width: 400px; display: inline-block" /> &nbsp;
                    <!-- should lat/long be 1 or 2 input boxes??? -->
                    <label for="long" style="display: inline-block">Long:</label> &nbsp;
                    <input type="text" id="long" v-model="selectIncidentLong"
                        style="width:400px; display: inline-block" /> &nbsp; &nbsp;
                    <button type="button" id="go-button" @click="newLocation"
                        style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Go</button>
                </div>


                <h1 class="cell large-12" style="font-size: 42px">Filters:</h1>
                <!-- incident_type: list of checkboxes per incident_type -->

                <div class="cell large-12">
                    <p style="text-decoration: underline; font-weight: bold;">Incident Type: </p>
                    <input type="checkbox" id="1" name="incident-type0" v-model="filterIncidentType" value="0">
                    <label for="incident-type0"> Homicide</label>
                    <input type="checkbox" id="2" name="incident-type1" v-model="filterIncidentType" value="1">
                    <label for="incident-type1"> Assault</label>
                    <input type="checkbox" id="3" name="incident-type2" v-model="filterIncidentType" value="2">
                    <label for="incident-type2"> Rape</label>
                    <input type="checkbox" id="4" name="incident-type3" v-model="filterIncidentType" value="3">
                    <label for="incident-type3"> Theft / Burglary / Robbery</label>
                    <input type="checkbox" id="5" name="incident-type4" v-model="filterIncidentType" value="4">
                    <label for="incident-type4"> Property Damage</label>
                    <input type="checkbox" id="6" name="incident-type5" v-model="filterIncidentType" value="5">
                    <label for="incident-type5"> Narcotics</label>
                    <input type="checkbox" id="7" name="incident-type6" v-model="filterIncidentType" value="6">
                    <label for="incident-type6"> Other</label>
                </div>

                <!-- neighborhood_name: list of checkboxes per neighborhood_name -->
                <div class="cell large-12">
                    <p style="text-decoration: underline; font-weight: bold;">Neighborhood Name: </p>
                    <input type="checkbox" id="1" name="neighborhood1" v-model="filterNeighborhood" value="1">
                    <label for="neighborhood1">Conway/Battlecreek/Highwood</label>
                    <input type="checkbox" id="2" name="neighborhood2" v-model="filterNeighborhood" value="2">
                    <label for="neighborhood2">Greater East Side</label>
                    <input type="checkbox" id="3" name="neighborhood3" v-model="filterNeighborhood" value="3">
                    <label for="neighborhood3">West Side</label>
                    <input type="checkbox" id="4" name="neighborhood4" v-model="filterNeighborhood" value="4">
                    <label for="neighborhood4">Dayton's Bluff</label>
                    <input type="checkbox" id="5" name="neighborhood5" v-model="filterNeighborhood" value="5">
                    <label for="neighborhood5">Payne/Phalen</label>
                    <input type="checkbox" id="6" name="neighborhood6" v-model="filterNeighborhood" value="6">
                    <label for="neighborhood6">North End</label>
                    <input type="checkbox" id="7" name="neighborhood7" v-model="filterNeighborhood" value="7">
                    <label for="neighborhood7">Thomas/Dale(Frogtown)</label>
                    <input type="checkbox" id="8" name="neighborhood8" v-model="filterNeighborhood" value="8">
                    <label for="neighborhood8">Summit/University</label>
                    <input type="checkbox" id="9" name="neighborhood9" v-model="filterNeighborhood" value="9">
                    <label for="neighborhood9">West Seventh</label>
                    <input type="checkbox" id="10" name="neighborhood10" v-model="filterNeighborhood" value="10">
                    <label for="neighborhood10">Como</label>
                    <input type="checkbox" id="11" name="neighborhood11" v-model="filterNeighborhood" value="11">
                    <label for="neighborhood11">Hamline/Midway</label>
                    <input type="checkbox" id="12" name="neighborhood12" v-model="filterNeighborhood" value="12">
                    <label for="neighborhood12">St. Anthony</label>
                    <input type="checkbox" id="13" name="neighborhood13" v-model="filterNeighborhood" value="13">
                    <label for="neighborhood13">Union Park</label>
                    <input type="checkbox" id="14" name="neighborhood14" v-model="filterNeighborhood" value="14">
                    <label for="neighborhood14">Macalester/Groveland</label>
                    <input type="checkbox" id="15" name="neighborhood15" v-model="filterNeighborhood" value="15">
                    <label for="neighborhood15">Highland</label>
                    <input type="checkbox" id="16" name="neighborhood16" v-model="filterNeighborhood" value="16">
                    <label for="neighborhood16">Summit Hill</label>
                    <input type="checkbox" id="17" name="neighborhood17" v-model="filterNeighborhood" value="17">
                    <label for="neighborhood17">Capitol River</label>
                </div>

                <!-- date range: select a start and end date (only show crimes between those dates) -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Dates: </p>
                    <label for="start-date" style="display: inline-block">Start Date:</label> &nbsp;
                    <input type="text" id="start-date" v-model="filterDates[0]"
                        style="width: 300px; display:inline-block" /> <br>
                    <label for="end-date" style="display:inline-block">End Date:</label> &nbsp;
                    <input type="text" id="end-date" v-model="filterDates[1]"
                        style="width: 300px; display: inline-block" />

                </div>

                <!-- time range: select a start and end time (only show crimes that occurred between those times of day) -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Times: </p>
                    <label for="start-time" style="display: inline-block">Start Time:</label> &nbsp;
                    <input type="text" id="start-time" v-model="filterTimes[0]"
                        style="width: 300px; display: inline-block" /> <br>
                    <label for="end-time" style="display: inline-block">End Time:</label> &nbsp;
                    <input type="text" id="end-time" v-model="filterTimes[1]"
                        style="width: 300px; display: inline-block" />

                </div>

                <!-- max incidents: select maximum number of incidents to retrieve / show -->
                <div class="cell large-4">
                    <p style="text-decoration: underline; font-weight: bold;">Max Incidents: </p>
                    <input type="text" id="max-incidents" v-model="filterMaxIncidents" style="width: 200px">

                </div>

                <!--  submit  button -->
                <div class="cell large-12" style="text-align: center">
                    <button type="button" @click="filterCrimes"
                        style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Submit</button>
                </div>

                <br><br><br><br>


                <CrimesResult :result_array="incidents" />


                <br><br>
            </div>
        </div>
    </div>

    <!-- NEW INCIDENT PAGE VIEW -->
    <div v-if="view === 'new_incident'">
        <!-- Replace this with your actual form: can be done here or by making a new component -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell auto" style="text-align: center">New Incident Form</h1>

                <!-- Ensure all fields are filled out before submitting request, otherwise show some error message -->

                <!-- THINK ABOUT IF WE WANT TEXT BOXES FOR DROPDOWN MENUS FOR SOME OF THESE... -->
                <div class="cell large-12" style="text-align: center">
                    <label for="case-number" style="display: inline-block">Case Number:</label> &nbsp;
                    <input type="text" id="case-number" style="width: 300px; display: inline-block"> <br>

                    <label for="date" style="display: inline-block">Date:</label> &nbsp;
                    <input type="text" id="date" style="width: 300px; display: inline-block"> <br>
                    <!-- show the correct format as faded text in box?? -->

                    <label for="time" style="display: inline-block">Time:</label> &nbsp;
                    <input type="text" id="time" style="width: 300px; display: inline-block"> <br>
                    <!-- show the correct format as faded text in box?? -->

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



                    <button type="button" @click="newIncident"
                        style="border: 1px solid black; padding: 10px; background-color: #30cf3d; font-weight: bold; padding: 10px">Submit</button>

                </div>



            </div>
        </div>
    </div>

    <!-- ABOUT PAGE VIEW -->
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
                    <!-- This can be on the API and we get using a request url -->
                    <!-- <img src="images/jackPic.jpeg" alt="Jack" style="width: 250px"> -->
                    <p>Senior at St. Thomas majoring in Computer Science. In my free time I like to snowboard, golf, and
                        play soccer.</p>

                </div>

                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Kevin</p>
                    <!-- photo and short bio-->
                    <!-- This can be on the API and we get using a request url -->
                    <!-- <img src="images/" alt="Kevin" style="width: 250px"> -->
                    <p>fkadhjbfgkahsdbgfkjhasbfkhabsfkjhbas</p>

                </div>

                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Sam McEnery</p>
                    <!-- photo and short bio-->
                    <!-- This can be on the API and we get using a request url -->
                    <!-- <img src="images/" alt="Sam" style="width: 250px"> -->
                    <p>sdojghbnsdjlgbljsdbglsdjgbsdljgblj.</p>
                </div>

                <!-- maybe add some horizontal lines to separate the sections??-->

                <h1 class="cell large-12" style="font-size: 40px">Description of tools:</h1>
                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Video of the application:</h1>
                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Six interesting findings:</h1>
                <ol>
                    <li>I learned everything</li>
                    <li>I learned nothing</li>
                </ol>
                    
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
