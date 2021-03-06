<template>
  <section class="templates">
    <header class="columns">
      <div class="column is-two-thirds">
        <h1 class="title is-4">Templates
          <span v-if="templates.length > 0">({{ templates.length }})</span></h1>
      </div>
      <div class="column has-text-right">
        <b-button type="is-primary" icon-left="plus" @click="showNewForm">New</b-button>
      </div>
    </header>

    <b-table :data="templates" :hoverable="true" :loading="loading.templates"
      default-sort="createdAt">
        <template slot-scope="props">
            <b-table-column field="name" label="Name" sortable>
                <a :href="props.row.id" @click.prevent="showEditForm(props.row)">
                  {{ props.row.name }}
                </a>
                <b-tag v-if="props.row.isDefault">default</b-tag>
            </b-table-column>

            <b-table-column field="createdAt" label="Created" sortable>
                {{ $utils.niceDate(props.row.createdAt) }}
            </b-table-column>

            <b-table-column field="updatedAt" label="Updated" sortable>
                {{ $utils.niceDate(props.row.updatedAt) }}
            </b-table-column>

            <b-table-column class="actions" align="right">
              <a href="#" @click.prevent="previewTemplate(props.row)">
                <b-tooltip label="Preview" type="is-dark">
                  <b-icon icon="file-find-outline" size="is-small" />
                </b-tooltip>
              </a>
              <a href="#" @click.prevent="showEditForm(props.row)">
                <b-tooltip label="Edit" type="is-dark">
                  <b-icon icon="pencil-outline" size="is-small" />
                </b-tooltip>
              </a>
             <a v-if="!props.row.isDefault" href="#"
                @click.prevent="$utils.confirm(null, () => makeTemplateDefault(props.row))">
                <b-tooltip label="Make default" type="is-dark">
                  <b-icon icon="check-circle-outline" size="is-small" />
                </b-tooltip>
              </a>
              <a v-if="!props.row.isDefault"
                href="#" @click.prevent="$utils.confirm(null, () => deleteTemplate(props.row))">
                <b-tooltip label="Delete" type="is-dark">
                  <b-icon icon="trash-can-outline" size="is-small" />
                </b-tooltip>
              </a>
            </b-table-column>
        </template>

        <template slot="empty" v-if="!loading.templates">
          <empty-placeholder />
        </template>
    </b-table>

    <!-- Add / edit form modal -->
    <b-modal scroll="keep" :aria-modal="true" :active.sync="isFormVisible"
      :width="1200" :can-cancel="false" class="template-modal">
      <template-form :data="curItem" :isEditing="isEditing"
        @finished="formFinished"></template-form>
    </b-modal>

    <campaign-preview v-if="previewItem"
      type='template'
      :id="previewItem.id"
      :title="previewItem.name"
      @close="closePreview"></campaign-preview>
  </section>
</template>

<script>
import Vue from 'vue';
import { mapState } from 'vuex';
import TemplateForm from './TemplateForm.vue';
import CampaignPreview from '../components/CampaignPreview.vue';
import EmptyPlaceholder from '../components/EmptyPlaceholder.vue';

export default Vue.extend({
  components: {
    CampaignPreview,
    TemplateForm,
    EmptyPlaceholder,
  },

  data() {
    return {
      curItem: null,
      isEditing: false,
      isFormVisible: false,
      previewItem: null,
    };
  },

  methods: {
    // Show the edit form.
    showEditForm(data) {
      this.curItem = data;
      this.isFormVisible = true;
      this.isEditing = true;
    },

    // Show the new form.
    showNewForm() {
      this.curItem = {};
      this.isFormVisible = true;
      this.isEditing = false;
    },

    formFinished() {
      this.$api.getTemplates();
    },

    previewTemplate(c) {
      this.previewItem = c;
    },

    closePreview() {
      this.previewItem = null;
    },

    makeTemplateDefault(tpl) {
      this.$api.makeTemplateDefault(tpl.id).then(() => {
        this.$api.getTemplates();

        this.$buefy.toast.open({
          message: `'${tpl.name}' made default`,
          type: 'is-success',
          queue: false,
        });
      });
    },

    deleteTemplate(tpl) {
      this.$api.deleteTemplate(tpl.id).then(() => {
        this.$api.getTemplates();

        this.$buefy.toast.open({
          message: `'${tpl.name}' deleted`,
          type: 'is-success',
          queue: false,
        });
      });
    },
  },

  computed: {
    ...mapState(['templates', 'loading']),
  },

  mounted() {
    this.$api.getTemplates();
  },
});
</script>
