<script>
export default {
    props: {
        result_array: Array,
        selectButtonClicked: Function
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
                <td></td>
                <td>Case Number</td>
                <td>Date</td>
                <td>Time</td>
                <td>Incident Type</td>
                <td>Police Grid</td>
                <td>Neighborhood Name</td>
                <td>Block</td>
                <td></td>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(incident, index) in result_array"> <!-- makes table row for each incident result -->
                <td><button type="button" @click="this.selectButtonClicked(incident)" :id="'select-incident-' + incident.case_number" style="border: 1px solid black; padding: 5px; background-color: #30cf3d; font-weight: bold;">Select</button></td>

                <!-- table data for the case number -->
                <td>{{ incident.case_number }}</td>

                 <!-- table data for the Date/Time (Currently two seperate things, need to fix in Project 3 code) -->
                <td>{{ incident.date }}</td>

                <td>{{ incident.time }}</td>

                <!-- table data for the Code -->
                <td>{{ incident.code }}</td>

                <!-- table data for the police grid -->
                <td>{{ incident.police_grid }}</td>

                <!-- table data for the neighborhood id -->
                <td>{{ incident.neighborhood_number }}</td>

                <!-- table data for the block -->
                <td>{{ incident.block }}</td> <!-- REPLACE THE X'S IN THE NUMBER ADDRESS WITH 0'S -->

                <!-- DELETE BUTTON FOR EACH CRIME IN THE TABLE, CLICKING SHOULD DELETE REQUEST TO REMOVE INCIDENT FROM DATABASE...  -->
                <td><button type="button" @click="deleteButtonClicked" :id="'delete-incident-' + incident.case_number" style="border: 1px solid black; padding: 5px; background-color: red; font-weight: bold;">Delete</button></td>

            </tr>
        </tbody>
    </table>

</template>

<!-- include our table styling options here... -->
<!-- Style the background color of rows in the table to categorize crimes as "violent crimes" (crimes against another person), "property crimes" (crimes against a person's or business' property), or "other crimes" (anything else) -->
<style>

</style>
