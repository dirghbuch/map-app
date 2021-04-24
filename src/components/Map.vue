<template>
  <vl-map data-projection="EPSG:4326" style="height: 400px">
    <template>
      <b-container fluid>
        <b-row class="my-1">
          <b-col sm="3">
            <b-form-input @change="onChange()" v-model.lazy="search" type="text" placeholder="Search"></b-form-input>
          </b-col>
          <b-col sm="9">
            <b-form-group
              label=""
              v-slot="{ ariaDescribedby }"
            >
              <b-form-checkbox-group
                v-model="selected"
                :options="options"
                :aria-describedby="ariaDescribedby"
                switches
                @change="onChange()"
              ></b-form-checkbox-group>
            </b-form-group>
          </b-col>
        </b-row>
      </b-container>

    </template>
    <vl-view :zoom.sync="zoom" :center.sync="center" :rotation.sync="rotation"></vl-view>
    <vl-layer-tile>
      <vl-source-osm></vl-source-osm>
    </vl-layer-tile>
      <vl-overlay v-for="outlet in data" :key="outlet.locationid" :id="outlet.locationid" :position="outlet.coordinates">
        <template>
          <div  v-b-modal="'modal-'+outlet.locationid" class="overlay-content">
            <b-icon icon="cone" scale="2" :variant="iconMapping[outlet.Status]"></b-icon>
          </div>
        </template>
        <div>
          <b-modal :id="'modal-' + outlet.locationid" :title="outlet.Applicant">
            <b-container fluid>
              <b-row class="mb-1 text-center">
                <b-col>Address:{{outlet.Address}}</b-col>
              </b-row>
              <b-row class="mb-1 text-center">
                <b-col>{{outlet.FacilityType}}</b-col>
                </b-row>
            </b-container>
         </b-modal>
        </div>
      </vl-overlay>
    </vl-map>
</template>
<script>
  import axios from 'axios';
  const querystring = require('querystring');
  const serverUrl = 'https://main-map-app-uleqtuu142bbxchv-gtw.qovery.io/'
  export default {
    data () {
      return {
        zoom: 10,
        center: [-122.43301077434302,37.80457786909011],
        rotation: 0,
        iconMapping: {
          APPROVED: "success",
          SUSPENDED: "danger",
          EXPIRED: "warning",
          REQUESTED: "info"
        },
        options: [
          { text: 'Approved', value: 'APPROVED' },
          { text: 'Suspended', value: 'SUSPENDED' },
          { text: 'Expired', value: 'EXPIRED' },
          { text: 'Requested', value: 'REQUESTED' }
        ],
        data: null,
        search: null,
        selected: [],
        count: 20
      }
    },
    mounted () {
    axios
      .get(serverUrl + 'permits?count='+this.count)
      .then(response => (this.data = response.data))
    },
    methods: {
            onChange () {
                let queryString = {
                  count: this.count,
                  search: this.search,
                  status: this.selected
                }
                this.data = null;
                queryString = querystring.stringify(queryString);
                console.log(queryString);
                axios
                .get(serverUrl + 'permits?'+ queryString)
                .then(response => (this.data = response.data))
            }
      }
  }
</script>