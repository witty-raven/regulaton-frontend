<template>
    <v-card class="mx-auto overflow-y-auto " outlined>
        <v-data-table

                :headers="headers"
                :height="`${tableHeight}px`"
                :items="contacts"
                :items-per-page.sync="pageSize"

                :loading="isDataLoading"
                :page.sync="currentPage"
                :server-items-length="totalContactsCount"
                @click:row="showAlert" class="elevation-1"
                fixed-header>

            <template slot="loading">
                <v-container fluid>
                    <v-row align="center"
                           justify="center"
                           style="height: 200px;">
                        <v-col cols="4">
                            <div class="ma-5">{{loadingText}}</div>
                            <v-progress-linear :color="this.$vuetify.theme.themes.light.contacts"
                                               height="3"
                                               indeterminate

                                               rounded
                                               width="50"
                            ></v-progress-linear>

                        </v-col>
                    </v-row>
                </v-container>
            </template>
            <template slot="items" slot-scope="props">
                <tr @click="showAlert(props.item)">
                    <td>{{ props.item.name }}</td>
                    <td>{{ props.item.type }}</td>
                    <td>{{ props.item.owner }}</td>
                    <td>{{ props.item.status }}</td>
                </tr>
            </template>

            <template v-slot:item.view_bookings="{ item }">
                <v-btn :color="getContactsColor()" @click="showBookingsForContact(item)" class="white--text" rounded
                       x-small>View Booking
                </v-btn>
            </template>
        </v-data-table>
    </v-card>
</template>

<script>

    import router from "../router";
    import axios from "axios";

    export default {
        name: "ContactsDatatable",
        props: ["tableHeight"],
        components: ({}),
        watch: {
            currentPage: function (newPage, oldPage) {
                this.fetchContacts();
            },
            pageSize: function (newPagesize, oldPagesize) {
                this.fetchContacts();
            }
        },
        methods: {

            showAlert(row) {
                // console.log(row);
                //router.push({name: 'inventoryDetail', params: {inventoryItem: row}});

                // alert('alert');
            },
            showBookingsForContact(item) {
                console.log("ITEM CLICKED!!!");
                console.log(item);
            },
            fetchContacts() {
                let __self = this;
                console.log("In Fetch Contacts" + this.currentPage);

                let offset = (this.currentPage - 1) * (this.pageSize);
                axios
                    .post(`http://localhost:3002/api/contact?action=query&limit=${this.pageSize}&offset=${offset}`, {})
                    .then(response => {
                        // handle success
                        __self.isDataLoading = false;
                        __self.contacts = response.data.rows;
                        __self.totalContactsCount = response.data.totalCount;
                        console.log(response);
                    })
                    .catch(error => {
                        // handle error

                        console.log(error);
                    })
                    .finally(function () {
                        // always executed
                    });
            },
            getContactsColor() {
                return this.$vuetify.theme.themes.light.contacts;
            }
        },
        mounted() {
            this.fetchContacts();
        },
        data() {
            return {
                pageSize: 10,
                loadingText: "Fetching your contacts list ... ",
                isDataLoading: true,
                search: "",
                headers: [
                    {
                        text: "Contact Name",
                        align: "left",
                        sortable: true,
                        value: "NAME",
                    },
                    {text: "Address", align: "center", value: "ADDRESS"},
                    {text: "Phone", align: "center", value: "PHONE_NO"},
                    {text: "Service Type", align: "center", value: "SERVICE_TYPE"},
                    {text: " ", align: "center", value: "view_bookings", sortable: false},
                ],
                contacts: [],
                currentPage: 1,
                totalContactsCount: 0
            };
        },

    }
</script>

<style>

</style>
