<template>
  <main>
    <div v-if="!!!url & editMode" class='help-text-container'>
      <p>No image configured</p>
      <p v-if="agentCustomProperties.length > 0">Use a slug or URL</p>
      <p v-if="agentCustomProperties.length == 0">No custom properties configured</p>
    </div>
    <div v-if="!!!url & editMode" class='slug-table-container' v-on:scroll.passive="handleScroll">
      <div v-if="contentScrollTop > 0" class="table-header-drop-shadow" v-bind:style="{width: this.$refs.table.clientWidth + 'px'}"/>
      <div class='slug-table' ref="table">
        <table v-if="agentCustomProperties.length > 0" class="base-table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Slug</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="property in agentCustomProperties" :key="property.slug">
              <td>
                  <span class="current-value">{{ property.name }}</span>
              </td>
              <td>
                  <span class="current-value">{{ property.slug }}</span>
                  <span v-on:click="handleCopy" class="copy-button" :data-value="property.slug"><content-copy></content-copy></span>
              </td>
            </tr>
          </tbody>
        </table>
        <div v-if="!this.agent" class="center-text">
          <p>No agent selected</p>
        </div>
      </div>
    </div>
    <div v-if="!!!url & !editMode" class='help-text-container-view'>
      <p>No image configured</p>
    </div>
    <div v-if="url" class="image-container">
      <img :src="url" :style="imageFit">
    </div>
  </main>
</template>

<script>
import { ContentCopy } from 'mdue';

export default {
  name: "image-plus",
  props: { contextFactory: Function },
  data() {
    return {
      agent: null,
      customProperties: [],
      contentScrollTop: 0,
    }
  },
  created() {
    this.context = this.contextFactory();
    this.client = this.context.createResourceDataClient();
    this.client.query(
      {
        selector: 'Agent',
        fields: ['custom','name','publicId'],
      },
      (results) => (this.agent = results && results[0] ? results[0].data : null),
    );
    this.client.query(
      {
        selector: 'CustomPropertyList',
        fields: ['flags', 'name', 'publicId', 'scopeType', 'slug'],
      },
      (results) => (this.customProperties = results && results[0] ? results[0].data : []),
    );
  },
  methods: {
    handleScroll(e) {
      this.contentScrollTop = e.target.scrollTop;
    },
    handleCopy(e) {
      navigator.clipboard.writeText(e.currentTarget.dataset.value);
    }
  },
  components: {
    ContentCopy
  },
  computed: {
    editMode() {
      return this.context.mode === 'edit';
    },
    url() {
      if (!!this.context.inputs.customPropertySlug) {
        let props = this.agentCustomProperties
          .filter((customProperty) => customProperty.slug === this.context.inputs.customPropertySlug)

        if (props.length){
          return props[0].value;
        }
        // Return empty gif when no custom propert is found or when its not loaded yet
        return 'data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw=='
      }
      return this.context.inputs.url;
    },
    agentCustomProperties() {
      if (this.agent && this.agent.custom && this.customProperties) {
        return this.customProperties.map(
          (customProperty) => ({
            name: customProperty.name,
            slug: customProperty.slug,
            value: this.agent.custom[customProperty.scopeType + customProperty.slug],
          }))
          .filter((customProperty) => !!customProperty.value);
      }
      return []
    },
    imageFit() {
      return {
        '--object-fit': this.context.inputs.objectFit
      }
    }
  },
};
</script>

<style scoped>
main {
  height: 100%;
  display: flex;
  flex-direction: column;
}

img {
  text-align: center;
  object-fit: var(--object-fit);
  width: 100%;
  height: 100%;
}

.center-text {
  text-align: center;
}

.image-container {
  width: 100%;
  height: 100%;
}

.help-text-container {
  padding: 1em;
  background-color: var(--primary);
  color: var(--accent-color);
}

.help-text-container-view {
  text-align: center;
}

.help-text-container p {
  margin: 0;
  text-align: center;
}

.slug-table-container {
  overflow: auto;
}

.slug-table {
  padding-left: 1em;
  padding-right: 1em;
}

.base-table {
  width: 100%;
  border-collapse: collapse;
}

.base-table tr {
  text-align: left;
  height: 34px;
  border-bottom: 1px solid #dadce0;
}

.base-table th {
  position: sticky;
  top: 0;
  z-index: 100;
}

.table-header-drop-shadow {
  position: absolute;
  height: 34px;
  width: 100%;
  background: var(--basic);
  box-shadow: 0 2px 2px 0 var(--card-border-color);
  z-index: 10;
}

.copy-button {
  position: relative;
  top: 2px;
  float: right;
}

.copy-button:hover {
  cursor: pointer;
}

</style>
