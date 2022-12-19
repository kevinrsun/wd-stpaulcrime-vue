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
            new_incident_casenumber: null,
            new_incident_date: "",
            new_incident_time: "",
            new_incident_code: null,
            new_incident_type: "",
            new_incident_grid: null,
            new_incident_neigh_id: null,
            new_incident_block: "",
            filterMaxIncidents: 1000,
            selectIncidentAddress: "",
            selectIncidentLat: "",
            selectIncidentLong: "",
            codeTypes: {
                HomeicideCodes: "100,110,120",
                AssaultCodes: "400,410,411,412,420,421,422,430,431,432,440,441,442,450,451,\
                                452,453,810,861,862,863", // Code 810 was spelt Asasult
                RapeCodes: "210,220",
                TheftBurglaryRobberyCodes: "300,311,312,313,314,321,322,323,324,331,333,\
                                            334,341,342,343,344,351,352,353,354,361,363,\
                                            364,371,372,373,374,500,510,511,513,515,516,\
                                            520,521,523,525,526,530,531,533,535,536,540,\
                                            541,543,545,546,550,551,553,555,556,560,561,\
                                            563,565,566,600,603,611,612,613,621,622,623,\
                                            630,631,632,633,640,641,642,643,651,652,653,\
                                            661,662,663,671,672,673,681,682,683,691,692,\
                                            693,700,710,711,712,720,721,722,730,731,732",

                PropertyDamageCodes: "900,901,903,905,911,913,915,921,922,923,925,931,\
                                        933,941,942,951,961,971,972,975,981,982,1400,1401,\
                                        1410,1415,1416,1420,1425,1426,1430,1435,1436",

                NarcoticsCodes: "1800,1810,1811,1812,1813,1814,1815,1820,1822,1823,1824,\
                                1825,1830,1835,1840,1841,1842,1843,1844,1845,1850,1855,\
                                1860,1865,1870,1880,1885",

                OtherCodes: "614,2619,3100,9954,9959,9986",
            },
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

        filterCrimes() {

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
            if (this.filterIncidentType.length > 0) {
                if (query !== "") {
                    query = query + "&code=";
                } else {
                    query = "?code=";
                }

                let codesArr = [
                    this.codeTypes.HomeicideCodes,
                    this.codeTypes.AssaultCodes,
                    this.codeTypes.RapeCodes,
                    this.codeTypes.TheftBurglaryRobberyCodes,
                    this.codeTypes.PropertyDamageCodes,
                    this.codeTypes.NarcoticsCodes,
                    this.codeTypes.OtherCodes
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
            if (query !== "") {
                query = query + "&limit=" + this.filterMaxIncidents;
            } else {
                query = query + "?limit=" + this.filterMaxIncidents;
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
                        // Add field to incident for crime category
                        for (let i = 0; i < Object.keys(this.codeTypes).length; i++) {
                            if (Object.values(this.codeTypes)[i].split(',').includes(incident.code.toString())) {
                                incident.category = Object.keys(this.codeTypes)[i];
                            }
                        }
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

        newLocation() {
            let query = "";

            if (this.selectIncidentAddress !== "") {
                query = "https://nominatim.openstreetmap.org/search?q=" + this.selectIncidentAddress + "&format=json";
            } else {
                if (this.selectIncidentLat > this.leaflet.bounds.nw.lat) {
                    this.selectIncidentLat = this.leaflet.bounds.nw.lat;
                    alert("Latitude value is out of bounds. Set to max possible value.");
                }
                if (this.selectIncidentLat < this.leaflet.bounds.se.lat) {
                    this.selectIncidentLat = this.leaflet.bounds.se.lat;
                    alert("Latitude value is out of bounds. Set to min possible value.");
                }
                if (this.selectIncidentLong > this.leaflet.bounds.se.lng) {
                    this.selectIncidentLong = this.leaflet.bounds.se.lng;
                    alert("Longitude value is out of bounds. Set to max possible value.");
                }
                if (this.selectIncidentLong < this.leaflet.bounds.nw.lng) {
                    this.selectIncidentLong = this.leaflet.bounds.nw.lng;
                    alert("Longitude value is out of bounds. Set to min possible value.");
                }

                query = "https://nominatim.openstreetmap.org/reverse?lat=" + this.selectIncidentLat + "&lon=" + this.selectIncidentLong + "&format=json";
            }

            this.getJSON(query)
                .then((data) => {
                    console.log(data);
                    this.updatedLocation(data);
                })
                .then((err) => {
                    console.log(err);
                });
        },

        updatedLocation(data) {
            this.leaflet.center.lat = data.lat;
            this.leaflet.center.lng = data.lon;
            this.leaflet.map.flyTo([this.leaflet.center.lat, this.leaflet.center.lng], 17); // "this.leaflet.map.panTo" also works
        },

        mapPanOrZoom() {
            this.leaflet.center.lat = this.leaflet.map.getCenter().lat;
            this.leaflet.center.lng = this.leaflet.map.getCenter().lng;

            this.selectIncidentLat = this.leaflet.map.getCenter().lat;
            this.selectIncidentLong = this.leaflet.map.getCenter().lng;
        },

        newIncident() {
            //put together the data for the new incident then pass to uploadJSON???

            //Check if any of the inputs are null/empty before making the PUT request
            let incidentReq = "http://localhost:8000/new-incident";
            let method = 'PUT';
            if (this.new_incident_casenumber === null) {
                alert("Case Number Field Empty");
            }
            if (this.new_incident_date === '') {
                alert("Date Field Empty");
            }
            if (this.new_incident_block === '') {
                alert("Block Field Empty");
            }
            if (this.new_incident_code === null) {
                alert("Code Field Empty");
            }
            if (this.new_incident_grid === null) {
                alert("Grid Field Empty");
            }
            if (this.new_incident_neigh_id === null) {
                alert("Neighbourhood ID Field Empty");
            }
            if (this.new_incident_time === '') {
                alert("Time Field Empty");
            }
            if (this.new_incident_type === '') {
                alert("Type Field Empty");
            }

            let inputData = {
                case_number: this.new_incident_casenumber,
                date: this.new_incident_date,
                time: this.new_incident_time,
                code: this.new_incident_code,
                incident: this.new_incident_type,
                police_grid: this.new_incident_grid,
                neighborhood_number: this.new_incident_neigh_id,
                block: this.new_incident_block
            };

            console.log(inputData);

            //console.log(this.new_incident_casenumber);
            this.uploadJSON(method, incidentReq, inputData)
                .then((data) => {
                    console.log(data);
                    alert("Success");
                })
                .then((err) => {
                    console.log(err);
                })
        },

        selectButtonClicked(case_number) {
            console.log(case_number);
            let addy = case_number.block.toString();
            let parseAddyNum = addy.split(" ", 1);
            let addyNumber = parseAddyNum[0].replaceAll('X', 0);
            let addr = addy.substring(addy.indexOf(' ') + 1);

            let fullAddy = 'https://nominatim.openstreetmap.org/search?q=' + addyNumber + ' ' + addr + ', St. Paul MN' + '&format=json';
            console.log(fullAddy);

            this.getJSON(fullAddy)
                .then((data) => {
                    if (data.length !== 0) {
                        this.leaflet.center.lat = data[0].lat;
                        this.leaflet.center.lng = data[0].lon;
                        this.leaflet.map.flyTo([this.leaflet.center.lat, this.leaflet.center.lng], 17);
                        let yellowIcon = new L.icon({
                            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-yellow.png',
                            shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
                            iconSize: [25, 41],
                            iconAnchor: [12, 41],
                            popupAnchor: [1, -34],
                            shadowSize: [41, 41]
                        });
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.center.lat, this.leaflet.center.lng), { icon: yellowIcon });
                        let markerPopup = L.popup().setContent('<p>Incident: ' + case_number.code + '</p>\n' + '<p>Date: ' + case_number.date + '</p>\n' + '<p>Time: ' + case_number.time + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.map.addLayer(markerPin);
                    } else {
                        alert('No data found');
                    }
                })
                .then((err) => {
                    console.log(err);
                });
            //drop the pin on the incident location. New marker color, has popup with date, time, incident, and delete button.


        },

        deleteButtonClicked(incident, index, result_array) {
            let deleteReq = "http://localhost:8000/remove-incident";
            let method = "DELETE";
            let inputData = { case_number: incident.case_number };

            this.uploadJSON(method, deleteReq, inputData)
                .then(() => {
                    alert("Case number has been delete");
                    result_array.splice(index, 1);
                })
                .catch(() => {
                    alert("Case number not exist");
                });
        },

        neighborhoodMarkers() {
            let incidentReq = "http://localhost:8000/incidents";
            this.getJSON(incidentReq)
            .then((data) => {
                console.log(data);
                let hood1 = 0;
                let hood2 = 0;
                let hood3 = 0;
                let hood4 = 0;
                let hood5 = 0;
                let hood6 = 0;
                let hood7 = 0;
                let hood8 = 0;
                let hood9 = 0;
                let hood10 = 0;
                let hood11 = 0;
                let hood12 = 0;
                let hood13 = 0;
                let hood14 = 0;
                let hood15 = 0;
                let hood16 = 0;
                let hood17 = 0;
                let greenIcon = new L.icon({
                    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
                    shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
                    iconSize: [25, 41],
                    iconAnchor: [12, 41],
                    popupAnchor: [1, -34],
                    shadowSize: [41, 41]
                });
                for (let j = 0; j < data.length; j++) { //set neighborhood incident totals
                    if (data[j].neighborhood_number === 1) {
                        hood1++;
                    } else if (data[j].neighborhood_number === 2) {
                        hood2++;
                    } else if (data[j].neighborhood_number === 3) {
                        hood3++;
                    } else if (data[j].neighborhood_number === 4) {
                        hood4++;
                    } else if (data[j].neighborhood_number === 5) {
                        hood5++;
                    } else if (data[j].neighborhood_number === 6) {
                        hood6++;
                    } else if (data[j].neighborhood_number === 7) {
                        hood7++;
                    } else if (data[j].neighborhood_number === 8) {
                        hood8++;
                    } else if (data[j].neighborhood_number === 9) {
                        hood9++;
                    } else if (data[j].neighborhood_number === 10) {
                        hood10++;
                    } else if (data[j].neighborhood_number === 11) {
                        hood11++;
                    } else if (data[j].neighborhood_number === 12) {
                        hood12++;
                    } else if (data[j].neighborhood_number === 13) {
                        hood13++;
                    } else if (data[j].neighborhood_number === 14) {
                        hood14++;
                    } else if (data[j].neighborhood_number === 15) {
                        hood15++;
                    } else if (data[j].neighborhood_number === 16) {
                        hood16++;
                    } else if (data[j].neighborhood_number === 17) {
                        hood17++;
                    }
                }
                for (let i = 0; i < this.leaflet.neighborhood_markers.length; i++) {  //loop thru the neighborhood markers
                    if (i === 0) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood1 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 1) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood2 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 2) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood3 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 3) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood4 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 4) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood5 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 5) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood6 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 6) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood7 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 7) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood8 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 8) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood9 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 9) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood10 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 10) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood11 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 11) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood12 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 12) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood13 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 13) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood14 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 14) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood15 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 15) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood16 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    } else if (i === 16) {
                        let markerPin = new L.Marker(new L.LatLng(this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]), { icon: greenIcon });
                        let markerPopup = L.popup().setContent('<p>Total Incidents: ' + hood17 + '</p>');
                        markerPin.bindPopup(markerPopup).openPopup();
                        this.leaflet.neighborhood_markers[i].marker = markerPin;
                        this.leaflet.map.addLayer(markerPin);
                    }
                }
            })
            .then((err) => {
                console.log(err);
            });
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

        // Initial coords on mount
        this.selectIncidentLat = this.leaflet.center.lat;
        this.selectIncidentLong = this.leaflet.center.lng;

        // Map events
        this.leaflet.map.on('dragend', this.mapPanOrZoom);
        this.leaflet.map.on('zoom', this.mapPanOrZoom);

        this.neighborhoodMarkers();
    }
}
</script>

<template>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <p :class="'cell small-4 ' + ((view === 'map') ? 'selected' : 'unselected')" style="background-color: blueviolet;" @click="viewMap">Map</p>
            <p :class="'cell small-4 ' + ((view === 'new_incident') ? 'selected' : 'unselected')" style="background-color: blueviolet;"
                @click="viewNewIncident">New Incident</p>
            <p :class="'cell small-4 ' + ((view === 'about') ? 'selected' : 'unselected')" style="background-color: blueviolet;" @click="viewAbout">About</p>
        </div>
    </div>

    <!-- MAP PAGE VIEW -->
    <div v-show="view === 'map'">
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <div id="leafletmap" class="cell auto"></div>

                <h1 class="cell large-12" id="title1">Coordinates:</h1>
                <!-- input box to take address/lat/longs and go button -->
                <!-- input box/s should be updated with new location when map is changed (pan/zoom) -->
                <!-- Clamp input values if lat/long is outside of St. Paul's bounding box -->
                <div class="cell large-12">
                    <label for="address" style="display: inline-block">Address:</label> &nbsp;
                    <input type="text" id="address" v-model="selectIncidentAddress"
                        style="width: 400px; display: inline-block" /> <br>
                    <label for="lat" style="display: inline-block">Lat:</label> &nbsp;
                    <input type="text" id="lat" v-model="selectIncidentLat"
                        style="width: 400px; display: inline-block" /> &nbsp;
                    <!-- should lat/long be 1 or 2 input boxes??? -->
                    <label for="long" style="display: inline-block">Long:</label> &nbsp;
                    <input type="text" id="long" v-model="selectIncidentLong"
                        style="width:400px; display: inline-block" /> &nbsp; &nbsp;
                    <button type="button" id="go-button" @click="newLocation"
                        style="width: 10%; border: 1px solid black; padding: 10px; background-color: blueviolet; font-weight: bold; padding: 10px">Go
                    </button>
                </div>


                <h1 class="cell large-12" id="title1">Filters:</h1>
                <!-- incident_type: list of checkboxes per incident_type -->

                <div class="cell large-12" style="padding-bottom: 15px">
                    <p id="title2">Incident Type: </p>
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
                <div class="cell large-12" style="padding-bottom: 15px">
                    <p id="title2">Neighborhood Name: </p>
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
                <div class="cell large-4" style="padding-bottom: 15px">
                    <p id="title2">Dates: </p>
                    <label for="start-date" style="display: inline-block">Start Date:</label> &nbsp;
                    <input type="text" id="start-date" v-model="filterDates[0]"
                        style="width: 300px; display:inline-block" /> <br>
                    <label for="end-date" style="display:inline-block">End Date:</label> &nbsp;
                    <input type="text" id="end-date" v-model="filterDates[1]"
                        style="width: 300px; display: inline-block" />

                </div>

                <!-- time range: select a start and end time (only show crimes that occurred between those times of day) -->
                <div class="cell large-4">
                    <p id="title2">Times: </p>
                    <label for="start-time" style="display: inline-block">Start Time:</label> &nbsp;
                    <input type="text" id="start-time" v-model="filterTimes[0]"
                        style="width: 300px; display: inline-block" /> <br>
                    <label for="end-time" style="display: inline-block">End Time:</label> &nbsp;
                    <input type="text" id="end-time" v-model="filterTimes[1]"
                        style="width: 300px; display: inline-block" />

                </div>

                <!-- max incidents: select maximum number of incidents to retrieve / show -->
                <div class="cell large-4">
                    <p id="title2">Max Incidents: </p>
                    <label for="max-incidents" style="display: inline-block">Amount:</label> &nbsp;
                    <input type="text" id="max-incidents" v-model="filterMaxIncidents" style="width: 300px">
                    <!--  submit  button -->
                    <div>
                        <button type="button" @click="filterCrimes"
                            style="width: 300px; ; display: inline-block; border: 1px solid black; padding: 10px; margin-top: 20px; background-color: blueviolet; font-weight: bold;">Submit
                        </button>
                    </div>
                </div>



                <br><br><br><br>

                <CrimesResult :result_array="incidents" :selectButtonClicked="selectButtonClicked" :deleteButtonClicked="deleteButtonClicked"/>
                
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
                    <input type="text" id="case-number" style="width: 300px; display: inline-block"
                        v-model="new_incident_casenumber"> <br>

                    <label for="date" style="display: inline-block">Date:</label> &nbsp;
                    <input type="text" id="date" style="width: 300px; display: inline-block"
                        v-model="new_incident_date"> <br>
                    <!-- show the correct format as faded text in box?? -->

                    <label for="time" style="display: inline-block">Time:</label> &nbsp;
                    <input type="text" id="time" style="width: 300px; display: inline-block"
                        v-model="new_incident_time"> <br>
                    <!-- show the correct format as faded text in box?? -->

                    <label for="incident-code" style="display: inline-block">Code:</label> &nbsp;
                    <input type="text" id="incident-code" style="width: 300px; display: inline-block"
                        v-model="new_incident_code"> <br>

                    <!-- do we want to have the code and the incident type linked somehow?? like a dropdown for incident type that auto populates code? -->
                    <label for="incident-type" style="display:inline-block">Incident Type:</label> &nbsp;
                    <input type="text" id="incident-type" style="width: 300px; display: inline-block"
                        v-model="new_incident_type"> <br>

                    <label for="police-grid" style="display: inline-block">Police Grid:</label> &nbsp;
                    <input type="text" id="police-grid" style="width: 300px; display: inline-block"
                        v-model="new_incident_grid"> <br>

                    <label for="neighborhood-id" style="display: inline-block">Neighborhood ID:</label> &nbsp;
                    <input type="text" id="neighborhood-id" style="width: 300px; display: inline-block"
                        v-model="new_incident_neigh_id"> <br>

                    <label for="block" style="display: inline-block">Block:</label> &nbsp;
                    <input type="text" id="block" style="width: 300px; display: inline-block"
                        v-model="new_incident_block"> <br>



                    <button type="button" @click="newIncident"
                        style="border: 1px solid black; padding: 10px; background-color: blueviolet; font-weight: bold; padding: 10px">Submit</button>

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
                    <p>Also a Senior majoring in Computer Science. My favorite color is purple, I don't really like winter, I play guitar & video games. 🥶</p>

                </div>

                <div class="cell large-4" style="border: 1px solid black">
                    <p style="font-size: 25px; margin: auto; text-align: center;">Sam McEnery</p>
                    <!-- photo and short bio-->
                    <!-- This can be on the API and we get using a request url -->
                    <!-- <img src="images/" alt="Sam" style="width: 250px"> -->
                    <p>My name is Sam McEnery, I am a computer science major here at St. Thomas. I am originally from
                        Geneva, Illinois--a Suburb of Chicago. I've played Hockey all my life so I enjoy playing pond
                        hockey when the weather gets cold enough.</p>
                </div>

                <!-- maybe add some horizontal lines to separate the sections??-->

                <h1 class="cell large-12" style="font-size: 40px">Description of tools:</h1>
                <ol>
                    <li>In the Map section there are many tools we can use. First, we can look up crime by address, this
                        is done by typing in the specified address into the search bar. Next, we can also find crimes by
                        latitude and longitude through the same process. These search results can be filtered by
                        checking the boxes of the specified crime type, neighborhood, or date and time in which it
                        occurred.</li>
                    <li>Once a search is requested, the table populates with all the crimes with the specified
                        parameters with colors coordinating the violence of each crime.</li>
                    <li>In the New Incident section, this is where the user would go to input a new crime into the
                        database. All parameters must be met in order to succesfully do this.</li>
                </ol>

                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Video of the application:</h1>
                <div class="cell large-12">

                </div>


                <h1 class="cell large-12" style="font-size: 40px">Six interesting findings:</h1>
                <ol>
                    <li>Making a PUT form through user input correctly was very fascinating to learn.</li>
                    <li>Using longitude and latitude to traverse the program through a map.</li>
                    <li>Using v-model in order to bind data to an object from the website.</li>
                    <li>Learning about the quantity of different crimes happening in St. Paul.</li>
                    <li>Finding out how law enforcement organizes each crime in a database.</li>
                    <li>Learning how to apply colors to table rows conditionally through html.</li>
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

#title1 {
    font-weight: bold;
    text-decoration: underline;
    font-size: 48px;
    font-family: Arial, Helvetica, sans-serif;
}

#title2 {
    font-weight: bold;
    font-size: 28px;
}

</style>
