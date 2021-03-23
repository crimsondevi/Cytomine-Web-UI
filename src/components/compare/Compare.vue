<!-- Copyright (c) 2009-2020. Authors: see NOTICE file.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.-->


<template>
  <div v-if="error" class="box error">
    <h2> {{ $t('error') }} </h2>
    <p>{{ $t('error-loading-image') }}</p>
  </div>
  <div v-else class="content">
    <b-loading :is-full-page="false" :active="loading" />
    <div v-if="!loading" class="maps-wrapper">
      <div class="content__header">
        <h1>Which sample is of the highest quality?</h1>
      </div>

      <div class="images">
        <template v-for="(image, id, index) in images">
          <CompareImage
            :key="id"
            :image="image"
            :index="index"
          />
        </template>
      </div>
    </div>
  </div>
</template>


<script>
import {get} from '@/utils/store-helpers';

import CompareImage from '@/components/compare/CompareImage.vue';

import {ImageInstance} from 'cytomine-client';

import axios from 'axios'
import { postData, getData } from './requests';

export default {
  name: 'compare',

  components: {
    CompareImage
  },

  data() {
    return {
      imageIds: [258, 8479],
      images: {},
      chosenImage: null,
      error: false,
      loading: true,
    };
  },

  computed: {
    project: get('currentProject/project'),

  },

  methods: {
    async fetchIds () {
      let imgs = await axios.get('http://127.0.0.1:5000/get_images/244');
      console.log(imgs.data)
    },


    async fetchImages() {
 
      try {
        await Promise.all(this.imageIds.map(async id => {
          let image = await ImageInstance.fetch(id);
          this.images[id] = image;
        }));
        this.loading = false;
      }
      catch(err) {
        console.log(err);
        this.error = true;
      }
    }
  },

  async created() {
    await this.fetchImages();
    this.fetchIds();
  }

};
</script>

<style scoped>

  .images {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
  }



</style>