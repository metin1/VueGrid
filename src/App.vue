<template>
  <div class="app-container company-list">
    <div class="grid-container">
      <dx-data-grid
        ref="grid"
        height="100%"
        :data-source="companies"
        :allow-column-resizing="true"
        :allow-column-reordering="true"
        :column-auto-width="false"
        :show-borders="true"
        :row-alternation-enabled="true"
        :show-column-lines="true"
        :show-row-lines="true"
        @toolbar-preparing="onToolbarPreparing($event)"
        @editor-preparing="onEditorPreparing($event)"
        @selectionChanged="onSelectionChanged($event)"
      >
        <dx-editing
          :allow-updating="true"
          :allow-deleting="true"
          :allow-adding="false"
          :use-icons="true"
          start-edit-action="click"
          mode="batch"
        />
        <dx-selection
          select-all-mode="allPages"
          show-check-boxes-mode="onClick"
          mode="multiple"
          width="20"
        />
        <dx-column-fixing :enabled="true"/>
        <dx-group-panel :visible="true"/>
        <dx-grouping :auto-expand-all="true" :contextMenuEnabled="true"/>
        <dx-sorting mode="multiple"/>
        <dx-filter-row :visible="showFilter"/>
        <dx-column-chooser :enabled="true"/>
        <dx-search-panel :visible="true" :width="160"/>
        <dx-export :enabled="true" :allow-export-selected-data="true" file-name="Companies"/>
        <dx-load-panel :enabled="true"/>
        <dx-scrolling
          mode="virtual"
          rowRenderingMode="virtual"
          columnRenderingMode="virtual"
          :preloadEnabled="true"
        />
        <dx-column data-field="id" :visible="false"/>
        <dx-column data-field="name"/>
        <dx-column
          data-field="createdAt"
          data-type="datetime"
          format="dd/MM/yyyy HH:mm"
          sort-order="asc"
          :allow-sorting="true"
          :visible="true"
        />
        <dx-column
          data-field="updatedAt"
          data-type="datetime"
          format="dd/MM/yyyy HH:mm"
          :visible="true"
        />
        <dx-column data-field="type">
          <dx-lookup :data-source="types" value-expr="id" display-expr="name"/>
        </dx-column>
        <dx-column data-field="tags" :calculate-filter-expression="tagFilterExpression"/>
        <dx-column data-field="member" data-type="boolean"/>
        <dx-column data-field="noEmployees"/>
        <dx-column type="buttons" width="35" :buttons="commandButtons"/>
        <dx-column type="selection" width="30"/>
        <dx-summary>
          <dx-group-item column="member" summary-type="count" :customize-text="groupTotalText"/>
          <dx-total-item column="name" :customize-text="totalText" summary-type="count"/>
        </dx-summary>
        <dx-sort-by-group-summary-info summary-item="count"/>
        <div slot="filterButton" slot-scope="{ data }">
          <dx-button icon="fa fa-filter" :data="data" @click="toggleFilter"/>
        </div>
        <div slot="contactsButton" slot-scope="{ data }">
          <dx-button
            icon="fa fa-user-friends"
            :disabled="noneSelected"
            :data="data"
            @click="toggleFilter"
          />
        </div>
        <div slot="addButton" slot-scope="{ data }">
          <dx-button icon="fa fa-plus" type="default" :data="data" @click="toggleFilter"/>
        </div>
        <div slot="editButton" slot-scope="{ data }">
          <dx-button
            icon="fa fa-edit"
            class="dx-button-edit"
            :disabled="!oneSelected"
            :data="data"
            @click="toggleFilter"
          />
        </div>
      </dx-data-grid>
    </div>
  </div>
</template>

<script>
import companies from "./companies.js";
import tags from "./tags.js";
import types from "./types.js";
import _ from "lodash";
import DataSource from "devextreme/data/data_source";
import { DxButton } from "devextreme-vue";
import {
  DxDataGrid,
  DxColumn,
  DxGrouping,
  DxSummary,
  DxGroupItem,
  DxTotalItem,
  DxSortByGroupSummaryInfo,
  DxGroupPanel,
  DxColumnChooser,
  DxScrolling,
  DxSelection,
  DxSorting,
  DxFilterRow,
  DxSearchPanel,
  DxLoadPanel,
  DxColumnFixing,
  DxEditing,
  DxLookup,
  DxExport
} from "devextreme-vue/data-grid";

export default {
  name: "Companies",
  components: {
    DxButton,
    DxDataGrid,
    DxColumn,
    DxGrouping,
    DxGroupPanel,
    DxSummary,
    DxGroupItem,
    DxTotalItem,
    DxSortByGroupSummaryInfo,
    DxColumnChooser,
    DxScrolling,
    DxSelection,
    DxSorting,
    DxFilterRow,
    DxLoadPanel,
    DxSearchPanel,
    DxColumnFixing,
    DxEditing,
    DxLookup,
    DxExport
  },

  data() {
    return {
      tags: tags.getTags(),
      types: types.getTypes(),
      companies: companies.getCompanies(),
      showFilter: false,
      selectedRows: [],
      commandButtons: ["delete"]
    };
  },

  computed: {
    noneSelected() {
      return this.selectedRows.length === 0;
    },
    oneSelected() {
      return this.selectedRows.length === 1;
    },
    manySelected() {
      return this.selectedRows.length > 1;
    }
  },
  methods: {
    toggleFilter: function() {
      this.showFilter = !this.showFilter;
      if (!this.showFilter) {
        this.$refs.grid.instance.clearFilter();
      }
    },
    groupTotalText: function(data) {
      return data.value + " Companies";
    },
    totalText: function(data) {
      return data.value + " Companies";
    },

    tagFilterExpression(filterValues, selectedFilterOperation) {
      if (typeof window.CALC_FILTER === "undefined") {
        window.CALC_FILTER = 0;
      }
      window.CALC_FILTER++;
      //console.log("window.CALC_FILTER", window.CALC_FILTER);

      return function(itemData) {
        if (typeof window.RUN_FILTER == "undefined") {
          window.RUN_FILTER = 0;
        }
        window.RUN_FILTER++;
        //console.log("window.RUN_FILTER", window.RUN_FILTER);

        return true;
      };
    },
    onSelectionChanged(e) {
      this.selectedRows = e.selectedRowKeys;
    },
    onToolbarPreparing(e) {
      const groupPanel = _.find(e.toolbarOptions.items, { name: "groupPanel" });
      const saveButton = _.find(e.toolbarOptions.items, { name: "saveButton" });
      const revertButton = _.find(e.toolbarOptions.items, {
        name: "revertButton"
      });
      const exportButton = _.find(e.toolbarOptions.items, {
        name: "exportButton"
      });
      const columnChooserButton = _.find(e.toolbarOptions.items, {
        name: "columnChooserButton"
      });
      const searchPanel = _.find(e.toolbarOptions.items, {
        name: "searchPanel"
      });
      const filterButton = {
        location: "before",
        template: "filterButton"
      };
      const contactsButton = {
        location: "before",
        template: "contactsButton"
      };
      const addButton = {
        location: "after",
        template: "addButton"
      };
      const editButton = {
        location: "after",
        template: "editButton"
      };

      searchPanel.location = "before";
      columnChooserButton.location = "before";
      exportButton.location = "before";
      saveButton.options.type = "save";
      saveButton.options.icon = "fa fa-save";
      saveButton.location = "after";
      revertButton.location = "after";
      revertButton.options.type = "revert";
      groupPanel.location = "center";

      const newItems = [
        filterButton,
        searchPanel,
        contactsButton,
        exportButton,
        columnChooserButton,
        groupPanel,
        addButton,
        editButton,
        saveButton,
        revertButton
      ];

      e.toolbarOptions.items = newItems;
    },
    onEditorPreparing(e) {
      if (
        (e.parentType === "dataRow" || e.parentType === "filterRow") &&
        e.dataField === "tags"
      ) {
        e.editorName = "dxTagBox";
        var tagBoxData = new DataSource({
          store: tags.getTags()
        });
        e.editorOptions.dataSource = tagBoxData;
        e.editorOptions.showSelectionControls = true;
        e.editorOptions.displayExpr = "name";
        e.editorOptions.valueExpr = "name";
        e.editorOptions.value = e.value || [];
        if (e.parentType === "dataRow") {
          // accept new tags
          e.editorOptions.acceptCustomValue = true;
          e.editorOptions.onCustomItemCreating = function(e) {
            // Creates a new entry
            e.customItem = { name: e.text };
            // Adds the entry to the data source
            tagBoxData.store().insert(e.customItem);
            // Reloads the data source
            tagBoxData.reload();
          };
        }
        e.editorOptions.onValueChanged = function(args) {
          e.setValue(args.value);
        };
      }
    }
  }
};
</script>
<style lang="scss">
.company-list {
  .grid-container {
    position: absolute;
    top: 80px;
    bottom: 0;
    right: 0;
    left: 0;
    padding: 20px;
  }
  .dx-button-save {
    background: #42b983;
    .dx-icon {
      color: #fff;
    }
  }
  .dx-button-revert {
    background: orange;
    .dx-icon {
      color: #fff;
    }
  }
  .dx-button-edit {
    background: #00bcd4;
    .dx-icon {
      color: #fff;
    }
  }
  .dx-datagrid-search-panel {
    margin-left: 0;
  }

  .dx-link.dx-link-delete.dx-icon-trash {
    color: #888;
    font-size: 16px !important;
  }
  .dx-link.dx-link-delete.dx-icon-trash:hover {
    color: #d9534f;
  }
}
</style>
