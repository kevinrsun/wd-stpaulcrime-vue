<script>
export default {
    props: {
        result_array: Array
    },
    watch: {
        result_array() {
            console.log(this.result_array);
        }
    }
}
</script>

<template>
    <table>
        <thead>
            <tr>
                <td>Case Number</td>
                <td>Date/Time</td>
                <td>Code</td>
                <td>Incident Type</td>
                <td>Police Grid</td>
                <td>Neighborhood ID</td>
                <td>Block</td>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(incident, index) in result_array"> <!-- makes table row for each incident result -->

                <!-- table data for the case number -->
                <td>{{ incident.case_number }}</td>

                 <!-- table data for the Date/Time (Currently two seperate things, need to fix in Project 3 code) -->
                <td>{{ incident.date }}</td>

                <!-- table data for the Code -->
                <td>{{ incident.code }}</td>

                <!-- table data for the incident type -->
                <td>{{ incident.incident }}</td> 

                <!-- table data for the police grid -->
                <td>{{ incident.police_grid }}</td>

                <!-- table data for the neighborhood id -->
                <td>{{ incident.neighborhood_number }}</td>

                <!-- table data for the block -->
                <td>{{ incident.block }}</td>

                <!-- DELETE BUTTON FOR EACH CRIME IN THE TABLE, CLICKING SHOULD DELETE REQUEST TO REMOVE INCIDENT FROM DATABASE...  -->
                <button type="button" id="delete-incident" style="border: 1px solid black; padding: 10px; background-color: red; font-weight: bold; padding: 10px">Delete</button>

            </tr>
        </tbody>
    </table>

</template>

<!-- include our table styling options here... -->
<!-- Style the background color of rows in the table to categorize crimes as "violent crimes" (crimes against another person), "property crimes" (crimes against a person's or business' property), or "other crimes" (anything else) -->
<style>

</style>