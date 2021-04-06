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
  <div class="list-projects-wrapper content-wrapper">
    <b-loading :is-full-page="false" :active="loading" />
    <div class="box error" v-if="error">
      <h2>{{ $t("error") }}</h2>
      <p>{{ $t("unexpected-error-info-message") }}</p>
    </div>
    <div v-else-if="!loading" class="panel">
      <p class="panel-heading">
        {{ $t("projects") }}
      <button v-if="!currentUser.guestByNow" class="button is-link" @click="creationModal = true">
        {{$t('new-project')}}
      </button>
    </p>
    <div class="panel-block"> 
      <cytomine-table
        :collection="projectCollection"
        class="table-projects"
        :currentPage.sync="currentPage"
        :perPage.sync="perPage"
        :openedDetailed.sync="openedDetails"
        :sort.sync="sortField"
        :order.sync="sortOrder"
        :revision="revision"
      >
        <template #default="{row: project}">
          <b-table-column field="currentUserRole" label="" centered width="1" sortable>
            <icon-project-member-role
              :is-manager="project.currentUserRoles.admin"
              :is-representative="project.currentUserRoles.representative"
            />
          </b-table-column>

          <b-table-column field="name" :label="$t('name')" sortable width="250">
            <router-link :to="`/project/${project.id}`">
              {{ project.name }}
            </router-link>
          </b-table-column>

          <b-table-column field="membersCount" :label="$t('members')" centered sortable width="150">
            {{ project.membersCount }}
          </b-table-column>

          <b-table-column field="numberOfImages" :label="$t('images')" centered sortable width="150">
            <router-link :to="`/project/${project.id}/images`">{{ project.numberOfImages }}</router-link>
          </b-table-column>

          <b-table-column field="lastActivity" :label="$t('last-activity')" centered sortable width="180">
            {{ Number(project.lastActivity) | moment('ll') }}
          </b-table-column>


          <!-- implement field, label, and variable -->
          <b-table-column field="numberOfComparisons" :label="$t('comparisons')" centered sortable width="180">
            <!-- wrap in router link to /project/${project.id}/comparisons when implemented -->
            <!-- {{ Number(project.comparisons) | moment('ll') }} -->
            0
          </b-table-column>

          <b-table-column label=" " centered width="150">
            <router-link :to="`/project/${project.id}/compare`" class="button is-small is-link">
              <!-- {{$t('button-Compare')}} change to a translatable string -->
              Begin Compare
            </router-link>
          </b-table-column>
        </template>

        <template #detail="{row: project}">
          <project-details
            :project="project"
            :excluded-properties="excludedProperties"
            editable
            @update="updateProject()"
            @delete="deleteProject(project)"
          />
        </template>

        <template #empty>
          <div class="content has-text-grey has-text-centered">
            <p>{{$t('no-project')}}</p>
          </div>
        </template>
        </cytomine-table>
      </div>
    </div>
  </div>
</template>


<script>
import CytomineTable from '@/components/utils/CytomineTable';
import CytomineMultiselect from '@/components/form/CytomineMultiselect';
import CytomineSlider from '@/components/form/CytomineSlider';
import ProjectDetails from '@/components/project/ProjectDetails';
import AddProjectModal from '@/components/project/AddProjectModal';

import {get, sync, syncBoundsFilter, syncMultiselectFilter} from '@/utils/store-helpers';

import {ProjectCollection, OntologyCollection, TagCollection} from 'cytomine-client';
import IconProjectMemberRole from '@/components/icons/IconProjectMemberRole';
export default {
  name: 'compare-list-projects',
  components: {
    IconProjectMemberRole,
    CytomineTable,
    ProjectDetails,
    AddProjectModal,
    CytomineMultiselect,
    CytomineSlider
  },
  data() {
    return {
      loading: true,
      error: false,

      projects: [],
      ontologies: [],
      availableTags:[],


      contributorLabel: this.$t('contributor'),
      managerLabel: this.$t('manager'),

      creationModal: false,

      excludedProperties: [
        'name',
        'membersCount',
        'numberOfImages',
        'numberOfAnnotations',
        'numberOfJobAnnotations',
        'numberOfReviewedAnnotations',
        'lastActivity'
      ],

      maxNbMembers: 10,
      maxNbImages: 10,
      maxNbUserAnnotations: 100,
      maxNbJobAnnotations: 100,
      maxNbReviewedAnnotations: 100,

      revision: 0
    };
  },
  computed: {
    currentUser: get('currentUser/user'),

    searchString: sync('listProjects/searchString', {debounce: 500}),
    filtersOpened: sync('listProjects/filtersOpened'),

    availableRoles() {
      return [this.contributorLabel, this.managerLabel];
    },

    availableOntologies() {
      return [{id: 'null', name: this.$t('no-ontology')}, ...this.ontologies];
    },

    selectedOntologies: syncMultiselectFilter('listProjects', 'selectedOntologies', 'availableOntologies'),
    selectedRoles: syncMultiselectFilter('listProjects', 'selectedRoles', 'availableRoles'),
    selectedTags: syncMultiselectFilter('listProjects', 'selectedTags', 'availableTags'),
    boundsMembers: syncBoundsFilter('listProjects', 'boundsMembers', 'maxNbMembers'),
    boundsImages: syncBoundsFilter('listProjects', 'boundsImages', 'maxNbImages'),
    boundsUserAnnotations: syncBoundsFilter('listProjects', 'boundsUserAnnotations', 'maxNbUserAnnotations'),
    boundsJobAnnotations: syncBoundsFilter('listProjects', 'boundsJobAnnotations', 'maxNbJobAnnotations'),
    boundsReviewedAnnotations: syncBoundsFilter('listProjects', 'boundsReviewedAnnotations', 'maxNbReviewedAnnotations'),

    nbActiveFilters() {
      return this.$store.getters['listProjects/nbActiveFilters'];
    },

    selectedOntologiesIds() {
      return this.selectedOntologies.map(ontology => ontology.id);
    },

    boundsFilters() {
      return [
        {prop: 'numberOfImages', bounds: this.boundsImages},
        {prop: 'membersCount', bounds: this.boundsMembers},
        {prop: 'numberOfAnnotations', bounds: this.boundsUserAnnotations},
        {prop: 'numberOfJobAnnotations', bounds: this.boundsJobAnnotations},
        {prop: 'numberOfReviewedAnnotations', bounds: this.boundsReviewedAnnotations},
      ];
    },

    projectCollection() {
      let collection = new ProjectCollection({
        withMembersCount: true,
        withLastActivity: true,
        withCurrentUserRoles: true
      });
      if(this.selectedOntologiesIds.length > 0 && this.selectedOntologiesIds.length < this.availableOntologies.length){
        collection['ontology'] = {
          in: this.selectedOntologiesIds.join()
        };
      }
      if(this.selectedRoles.length > 0){
        collection['currentUserRole'] = {
          in: this.selectedRoles.join().toLowerCase()
        };
      }
      if(this.searchString) {
        collection['name'] = {
          ilike: encodeURIComponent(this.searchString)
        };
      }
      if(this.selectedTags.length > 0 && this.selectedTags.length < this.availableTags.length){
        collection['tag'] = {
          in: this.selectedTags.map(t => t.id).join()
        };
      }
      for(let {prop, bounds} of this.boundsFilters) {
        collection[prop] = {
          lte: bounds[1]
        };
        if(bounds[0] > 0) collection[prop]['gte'] = bounds[0];
      }
      return collection;
    },

    currentPage: sync('listProjects/currentPage'),
    perPage: sync('listProjects/perPage'),
    sortField: sync('listProjects/sortField'),
    sortOrder: sync('listProjects/sortOrder'),
    openedDetails: sync('listProjects/openedDetails')
  },
  watch: {
    revision() {
      this.fetchOntologies();
      this.fetchMaxFilters();
    }
  },
  methods: {
    async fetchOntologies() {
      let ontologies = (await OntologyCollection.fetchAll({light: true})).array;
      ontologies.sort((a, b) => a.name.localeCompare(b.name));
      this.ontologies = ontologies;
    },
    async fetchMaxFilters() {
      let stats = await new ProjectCollection.fetchBounds({withMembersCount:true});

      this.maxNbMembers = Math.max(10, stats.members.max);
      this.maxNbImages = Math.max(10, stats.numberOfImages.max);
      this.maxNbUserAnnotations = Math.max(100, stats.numberOfAnnotations.max);
      this.maxNbJobAnnotations = Math.max(100, stats.numberOfJobAnnotations.max);
      this.maxNbReviewedAnnotations = Math.max(100, stats.numberOfReviewedAnnotations.max);
    },
    async fetchTags() {
      this.availableTags = [{id: 'null', name: this.$t('no-tag')}, ...(await TagCollection.fetchAll()).array];
    },
    toggleFilterDisplay() {
      this.filtersOpened = !this.filtersOpened;
    },
    updateProject() {
      this.revision++;
    },
    async deleteProject(projectToDelete) {
      try {
        await projectToDelete.delete();
        this.revision++;
        this.$notify({
          type: 'success',
          text: this.$t('notif-success-project-deletion', {projectName: projectToDelete.name})
        });
      }
      catch(error) {
        this.$notify({
          type: 'error',
          text: this.$t('notif-error-project-deletion', {projectName: projectToDelete.name})
        });
        return;
      }
    }
  },
  async created() {
    try {
      await Promise.all([
        this.fetchOntologies(),
        this.fetchMaxFilters(),
        this.fetchTags()
      ]);
    }
    catch(error) {
      console.log(error);
      this.error = true;
    }

    this.loading = false;
  }
};
</script>

<style scoped>
.panel-block {
  padding-top: 0.8em;
}

.panel-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.search-block {
  display: flex;
}

.legend {
  margin-top: 0.8rem;
  margin-bottom: 1rem;
  border-radius: 10px;
  padding: 1rem 1.5em;
  background: #f8f8f8;
}

.legend p:not(:last-child) {
  margin-bottom: 0.4em;
}
</style>

<style>
.search-projects {
  max-width: 25em;
  margin-right: 1em;
}

.table-projects {
  margin-top: 1rem;
}

.list-projects-wrapper td, .list-projects-wrapper th {
  vertical-align: middle !important;
}
</style>