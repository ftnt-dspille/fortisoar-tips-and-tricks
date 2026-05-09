---
title: "FortiSOAR Widget Development Guide"
description: "Complete guide to building advanced FortiSOAR widgets using AngularJS framework"
date: 2025-01-25
draft: false
weight: 5
tags: ["fortisoar", "widgets", "angularjs", "development", "advanced"]
featured: true
---

*From AngularJS Basics to Advanced Security Operations Widgets*

---

## Table of Contents
1. [AngularJS Crash Course](#angularjs-crash-course)
2. [FortiSOAR Widget Fundamentals](#fortisoar-widget-fundamentals)
3. [Progressive Widget Examples](#progressive-widget-examples)
4. [Advanced Patterns](#advanced-patterns)
5. [Reference Guide](#reference-guide)

---

## AngularJS Crash Course

### Core Concepts You Need

#### 1. **The $scope Object - Your Data Bridge**
Think of `$scope` as the messenger between your HTML template and JavaScript controller.

```javascript
// In your controller
$scope.message = "Hello FortiSOAR!";
$scope.alerts = [];
$scope.isLoading = false;
```

```html
<!-- In your template -->
<div>{{message}}</div>
<div ng-if="isLoading">Loading alerts...</div>
```

#### 2. **Two-Way Data Binding - Magic Synchronization**
When data changes in JavaScript, the HTML updates automatically (and vice versa).

```html
<input ng-model="searchTerm" placeholder="Search alerts...">
<p>Searching for: {{searchTerm}}</p>
```

#### 3. **Directives - HTML Superpowers**
Directives extend HTML with custom behavior:

- `ng-if` - Show/hide elements
- `ng-repeat` - Loop through data
- `ng-click` - Handle clicks
- `ng-model` - Bind form inputs

```html
<ul>
  <li ng-repeat="alert in alerts" ng-click="selectAlert(alert)">
    {{alert.name}} - Severity: {{alert.severity}}
  </li>
</ul>
```

#### 4. **Services - Shared Functionality**
Services handle data fetching, API calls, and shared logic:

```javascript
// Using FortiSOAR's built-in services
Modules.get({module: 'alerts', id: alertId}).$promise.then(function(alert) {
  $scope.selectedAlert = alert;
});
```

#### 5. **Dependency Injection - Getting What You Need**
AngularJS automatically provides services to your controller:

```javascript
function myController($scope, $http, toaster, Modules) {
  // All these services are automatically injected
}
```

---

## FortiSOAR Widget Fundamentals

### Widget Structure
Every FortiSOAR widget has these key files:

```
my-widget/
├── info.json          # Widget metadata
├── edit.html          # Configuration form
├── edit.controller.js # Configuration logic  
├── view.html          # Display template
└── view.controller.js # Display logic
```

### The Widget Lifecycle

1. **User adds widget** → `edit.html` + `edit.controller.js`
2. **User configures** → Data saved to `config` object
3. **Widget displays** → `view.html` + `view.controller.js` receive `config`

---

## Progressive Widget Examples

### Level 1: Simple Data Display Widget

**Goal**: Show a count of open alerts

#### edit.html (Configuration)
```html
<form name="editWidgetForm" ng-submit="save()">
  <div class="form-group">
    <label>Widget Title</label>
    <input ng-model="config.title" class="form-control" required>
  </div>
  
  <div class="form-group">
    <label>Alert Severity to Count</label>
    <select ng-model="config.severity" class="form-control">
      <option value="critical">Critical</option>
      <option value="high">High</option>
      <option value="medium">Medium</option>
    </select>
  </div>
</form>
```

#### edit.controller.js
```javascript
function editAlertCountCtrl($scope, $uibModalInstance, config) {
  $scope.config = config;
  
  $scope.save = function() {
    if ($scope.editWidgetForm.$valid) {
      $uibModalInstance.close($scope.config);
    }
  };
  
  $scope.cancel = function() {
    $uibModalInstance.dismiss('cancel');
  };
}
```

#### view.controller.js
```javascript
function alertCountCtrl($scope, $http, API, Query, config) {
  $scope.config = config;
  $scope.alertCount = 0;
  $scope.loading = true;

  function loadAlertCount() {
    var query = new Query({
      filters: [{
        field: 'severity.itemValue',
        operator: 'eq', 
        value: $scope.config.severity
      }],
      aggregates: [{
        operator: 'count',
        field: '*',
        alias: 'total'
      }]
    });

    $http.post(API.QUERY + 'alerts', query.getQuery(true))
      .then(function(response) {
        $scope.alertCount = response.data['hydra:member'][0].total;
        $scope.loading = false;
      });
  }

  loadAlertCount();
}
```

#### view.html
```html
<div class="widget">
  <h3>{{config.title}}</h3>
  <div ng-if="loading">
    <cs-spinner></cs-spinner>
  </div>
  <div ng-if="!loading" class="alert-count">
    <span class="count-number">{{alertCount}}</span>
    <span class="count-label">{{config.severity}} alerts</span>
  </div>
</div>
```

**📚 Documentation References**: 
- Services: `$http`, `API`, `Query` (pages 14-15, 28)
- Directives: `ng-if`, `cs-spinner` (pages 31-45)

---

### Level 2: Interactive Button Widget

**Goal**: Buttons that trigger playbooks on selected records

#### edit.html
```html
<form name="editWidgetForm" ng-submit="save()">
  <div class="form-group">
    <label>Playbook Collection</label>
    <div data-cs-typeahead="playbookCollectionField" 
         ng-model="config.collection"
         data-change-method="loadPlaybooks">
    </div>
  </div>

  <div class="form-group" ng-if="playbooks.length">
    <label>Select Playbooks</label>
    <div ng-repeat="playbook in playbooks">
      <label>
        <input type="checkbox" ng-model="playbook.selected">
        {{playbook.name}}
      </label>
    </div>
  </div>
</form>
```

#### edit.controller.js  
```javascript
function editPlaybookButtonsCtrl($scope, $uibModalInstance, config, Field, $resource, API) {
  $scope.config = config;
  $scope.playbooks = [];

  // Create typeahead field for playbook collections
  $scope.playbookCollectionField = new Field({
    name: 'collection',
    title: 'Playbook Collection',
    dataSource: {
      model: 'workflow_collections',
      query: { $limit: 100 }
    }
  });

  $scope.loadPlaybooks = function() {
    if ($scope.config.collection) {
      var collectionId = $scope.config.collection['@id'].split('/').pop();
      $resource(API.BASE + 'workflows').get({
        collection: collectionId,
        $limit: 100
      }).$promise.then(function(response) {
        $scope.playbooks = response['hydra:member'];
      });
    }
  };

  $scope.save = function() {
    $scope.config.selectedPlaybooks = $scope.playbooks.filter(p => p.selected);
    $uibModalInstance.close($scope.config);
  };
}
```

#### view.controller.js
```javascript
function playbookButtonsCtrl($scope, config, $resource, API, toaster) {
  $scope.config = config;
  
  $scope.executePlaybook = function(playbook) {
    // Get selected records from parent scope (if in a grid context)
    var selectedRecords = $scope.$parent.getSelectedRows ? 
                         $scope.$parent.getSelectedRows() : [];
    
    if (selectedRecords.length === 0) {
      toaster.warning({body: 'Please select records first'});
      return;
    }

    var payload = {
      records: selectedRecords.map(r => r['@id']),
      __resource: 'alerts', // or dynamic based on context
      __uuid: playbook.uuid
    };

    $resource(API.ACTION_TRIGGER + playbook.triggerRoute).save(payload)
      .$promise.then(function() {
        toaster.success({body: `Executed ${playbook.name} on ${selectedRecords.length} records`});
      });
  };
}
```

#### view.html
```html
<div class="playbook-buttons-widget">
  <div class="button-group">
    <button ng-repeat="playbook in config.selectedPlaybooks"
            ng-click="executePlaybook(playbook)"
            class="btn btn-primary">
      <i class="{{playbook.icon || 'icon-execute'}}"></i>
      {{playbook.name}}
    </button>
  </div>
</div>
```

**📚 Documentation References**:
- Field Directive: `data-cs-typeahead` (pages 31-34)
- Services: `$resource`, `API` (page 14)
- Toaster notifications (page 15)

---

### Level 3: Data Grid Widget

**Goal**: Display queryable data in a grid with actions

#### edit.html
```html
<form name="editWidgetForm" ng-submit="save()">
  <div class="form-group">
    <label>Data Source</label>
    <select ng-model="config.resource" 
            ng-options="module.type as module.name for module in modules"
            ng-change="loadFields()">
      <option value="">Select Module</option>
    </select>
  </div>

  <div class="form-group" ng-if="config.resource">
    <label>Filters</label>
    <div data-cs-conditional 
         data-fields="moduleFields"
         data-ng-model="config.filters"
         data-form-name="'editWidgetForm'">
    </div>
  </div>

  <div class="form-group" ng-if="config.resource">
    <label>Columns to Show</label>
    <div ng-repeat="field in availableFields">
      <label>
        <input type="checkbox" ng-model="field.show">
        {{field.title}}
      </label>
    </div>
  </div>
</form>
```

#### edit.controller.js
```javascript
function editDataGridCtrl($scope, $uibModalInstance, config, appModulesService, Entity) {
  $scope.config = config;
  $scope.moduleFields = {};
  $scope.availableFields = [];

  appModulesService.load().then(function(modules) {
    $scope.modules = modules;
  });

  $scope.loadFields = function() {
    if ($scope.config.resource) {
      var entity = new Entity($scope.config.resource);
      entity.loadFields().then(function() {
        $scope.moduleFields = entity.getFormFields();
        $scope.availableFields = entity.getFormFieldsArray();
        
        // Set default columns if none selected
        if (!$scope.config.columns) {
          $scope.availableFields.slice(0, 5).forEach(f => f.show = true);
        }
      });
    }
  };

  $scope.save = function() {
    $scope.config.columns = $scope.availableFields.filter(f => f.show);
    $uibModalInstance.close($scope.config);
  };

  // Load fields if resource already selected
  if ($scope.config.resource) {
    $scope.loadFields();
  }
}
```

#### view.controller.js
```javascript
function dataGridCtrl($scope, config, PagedCollection) {
  $scope.config = config;
  $scope.loading = true;

  // Create column definitions for the grid
  $scope.columnDefs = $scope.config.columns.map(function(field) {
    return {
      field: field.name,
      displayName: field.title,
      width: field.name === 'name' ? '*' : 150,
      cellTemplate: getCellTemplate(field)
    };
  });

  function getCellTemplate(field) {
    switch(field.type) {
      case 'datetime':
        return "<div class='ui-grid-cell-contents'>{{row.entity['" + field.name + "'] | date:'short'}}</div>";
      case 'picklist':
        return "<div class='ui-grid-cell-contents'>{{row.entity['" + field.name + "'].itemValue}}</div>";
      default:
        return "<div class='ui-grid-cell-contents'>{{row.entity['" + field.name + "']}}</div>";
    }
  }

  // Set up grid options
  $scope.gridOptions = {
    columnDefs: $scope.columnDefs,
    enableFiltering: true,
    enableSorting: true,
    enableRowSelection: true,
    multiSelect: true,
    onRegisterApi: function(gridApi) {
      $scope.gridApi = gridApi;
    }
  };

  // Create paged collection for data
  $scope.pagedCollection = new PagedCollection($scope.config.resource, null, {
    filters: $scope.config.filters || []
  });

  // Load data
  $scope.pagedCollection.load().then(function() {
    $scope.gridOptions.data = $scope.pagedCollection.records;
    $scope.loading = false;
  });

  $scope.getSelectedRows = function() {
    return $scope.gridApi.selection.getSelectedRows();
  };
}
```

#### view.html
```html
<div class="data-grid-widget">
  <div class="widget-header">
    <h4>{{config.title || 'Data Grid'}}</h4>
    <button ng-click="pagedCollection.load()" class="btn btn-sm btn-default">
      <i class="icon-refresh"></i> Refresh
    </button>
  </div>

  <div ng-if="loading" class="text-center">
    <cs-spinner></cs-spinner>
    <p>Loading data...</p>
  </div>

  <div ng-if="!loading" 
       data-cs-grid 
       data-grid-options="gridOptions"
       data-paged-collection="pagedCollection">
  </div>
</div>
```

**📚 Documentation References**:
- Grid Directive: `data-cs-grid` (pages 37-39)
- Conditional Filters: `data-cs-conditional` (pages 34-36)
- PagedCollection Service (page 28)

---

### Level 4: Chart Widget

**Goal**: Display data as interactive charts

#### edit.controller.js
```javascript
function editChartCtrl($scope, $uibModalInstance, config, appModulesService, Entity) {
  $scope.config = config;
  $scope.config.chartType = $scope.config.chartType || 'pie';
  
  $scope.chartTypes = [
    {value: 'pie', label: 'Pie Chart'},
    {value: 'bar', label: 'Bar Chart'},
    {value: 'line', label: 'Line Chart'},
    {value: 'donut', label: 'Donut Chart'}
  ];

  appModulesService.load().then(function(modules) {
    $scope.modules = modules;
  });

  $scope.loadFields = function() {
    if ($scope.config.resource) {
      var entity = new Entity($scope.config.resource);
      entity.loadFields().then(function() {
        $scope.moduleFields = entity.getFormFields();
        // Get picklist and lookup fields for grouping
        $scope.groupByFields = entity.getFormFieldsArray().filter(function(field) {
          return field.type === 'picklist' || field.type === 'lookup';
        });
      });
    }
  };

  $scope.save = function() {
    if ($scope.editWidgetForm.$valid) {
      $uibModalInstance.close($scope.config);
    }
  };
}
```

#### view.controller.js  
```javascript
function chartCtrl($scope, config, $http, API, Query) {
  $scope.config = config;
  $scope.loading = true;

  function loadChartData() {
    var query = new Query({
      filters: $scope.config.filters || [],
      aggregates: [
        {
          operator: 'count',
          field: '*', 
          alias: 'count'
        },
        {
          operator: 'groupby',
          field: $scope.config.groupByField + '.itemValue',
          alias: 'label'
        },
        {
          operator: 'groupby', 
          field: $scope.config.groupByField + '.color',
          alias: 'color'
        }
      ]
    });

    $http.post(API.QUERY + $scope.config.resource, query.getQuery(true))
      .then(function(response) {
        var data = response.data['hydra:member'];
        $scope.chartConfig = createChartConfig(data);
        $scope.loading = false;
      });
  }

  function createChartConfig(data) {
    return {
      chart: $scope.config.chartType,
      title: $scope.config.title,
      data: data.map(function(item) {
        return {
          label: item.label,
          value: item.count,
          color: item.color
        };
      })
    };
  }

  loadChartData();
}
```

#### view.html
```html
<div class="chart-widget">
  <div ng-if="loading" class="text-center">
    <cs-spinner></cs-spinner>
  </div>
  
  <div ng-if="!loading" data-cs-chart="chartConfig"></div>
</div>
```

**📚 Documentation References**:
- Chart Directive: `data-cs-chart` (pages 39-41)
- Query Service (page 28)

---

### Level 5: Related Data Lookup Widget

**Goal**: Show related information based on current record context

#### view.controller.js
```javascript
function relatedDataCtrl($scope, config, $state, Modules, $http, API, Query) {
  $scope.config = config;
  $scope.relatedData = [];
  $scope.loading = true;
  $scope.currentRecord = null;

  function getCurrentRecord() {
    // Get current record from URL context (View Panel)
    var currentModule = $state.params.module;
    var currentId = $state.params.id;
    
    if (currentModule && currentId) {
      Modules.get({
        module: currentModule,
        id: currentId,
        $relationships: true
      }).$promise.then(function(record) {
        $scope.currentRecord = record;
        loadRelatedData();
      });
    }
  }

  function loadRelatedData() {
    if (!$scope.currentRecord) return;

    // Example: Find alerts related to the same IP address
    var lookupValue = $scope.currentRecord[$scope.config.sourceField];
    
    if (lookupValue) {
      var query = new Query({
        filters: [{
          field: $scope.config.targetField,
          operator: 'eq',
          value: lookupValue
        }],
        limit: $scope.config.maxResults || 10,
        sort: [{ 
          field: 'createDate',
          direction: 'DESC' 
        }]
      });

      $http.post(API.QUERY + $scope.config.targetModule, query.getQuery())
        .then(function(response) {
          $scope.relatedData = response.data['hydra:member'];
          $scope.loading = false;
        });
    } else {
      $scope.loading = false;
    }
  }

  $scope.navigateToRecord = function(record) {
    $state.go('viewPanel.modulesDetail', {
      module: $scope.config.targetModule,
      id: record.uuid
    });
  };

  getCurrentRecord();
}
```

#### view.html
```html
<div class="related-data-widget">
  <h4>{{config.title}}</h4>
  
  <div ng-if="loading">
    <cs-spinner></cs-spinner>
  </div>

  <div ng-if="!loading && relatedData.length === 0" class="text-muted">
    No related {{config.targetModule}} found
  </div>

  <div ng-if="!loading && relatedData.length > 0">
    <div class="related-item" ng-repeat="item in relatedData">
      <div class="item-header" ng-click="navigateToRecord(item)">
        <strong>{{item.name || item.title}}</strong>
        <small class="text-muted">{{item.createDate | date:'short'}}</small>
      </div>
      <div class="item-summary">
        {{item.description | limitTo:100}}{{item.description.length > 100 ? '...' : ''}}
      </div>
    </div>
  </div>
</div>
```

**📚 Documentation References**:
- State Service: `$state` (page 19)
- Modules Service (page 26)

---

## Advanced Patterns

### Real-time Updates with WebSockets

```javascript
function realtimeWidgetCtrl($scope, config, websocketService) {
  var subscription;

  function initWebsocket() {
    websocketService.subscribe('alerts', function(data) {
      // Handle real-time alert updates
      if (data.action === 'create') {
        $scope.alerts.unshift(data.record);
      } else if (data.action === 'update') {
        var index = $scope.alerts.findIndex(a => a.uuid === data.record.uuid);
        if (index !== -1) {
          $scope.alerts[index] = data.record;
        }
      }
      $scope.$apply(); // Trigger digest cycle
    }).then(function(sub) {
      subscription = sub;
    });
  }

  $scope.$on('websocket:reconnect', initWebsocket);
  $scope.$on('$destroy', function() {
    if (subscription) {
      websocketService.unsubscribe(subscription);
    }
  });

  initWebsocket();
}
```

### Multi-step Wizard Widgets

```javascript
function wizardCtrl($scope, config, WizardHandler) {
  $scope.currentStep = 1;
  $scope.wizardData = {};

  $scope.nextStep = function() {
    if ($scope.validateStep($scope.currentStep)) {
      WizardHandler.wizard().next();
      $scope.currentStep++;
    }
  };

  $scope.previousStep = function() {
    WizardHandler.wizard().previous();  
    $scope.currentStep--;
  };

  $scope.validateStep = function(step) {
    // Implement step validation logic
    return true;
  };
}
```

### Widget Interactions & Events

```javascript
// Emitting events to parent widgets
$scope.$emit('widget:dataChanged', {
  widgetId: $scope.config.id,
  newData: $scope.processedData
});

// Listening for events from other widgets  
$scope.$on('widget:dataChanged', function(event, data) {
  if (data.widgetId !== $scope.config.id) {
    $scope.handleExternalDataChange(data.newData);
  }
});
```

---

## Reference Guide

### Essential FortiSOAR Services

| Service | Purpose | Documentation Page |
|---------|---------|-------------------|
| `appModulesService` | Load application modules | 21 |
| `Entity` | Handle module metadata & fields | 23 |
| `Modules` | CRUD operations on records | 26 |
| `Query` | Build API queries | 28 |
| `PagedCollection` | Handle paginated data | 28 |
| `currentPermissionsService` | Check user permissions | 23 |
| `toaster` | Show notifications | 15 |
| `CommonUtils` | Utility functions | 22 |

### Key Directives

| Directive | Purpose | Documentation Page |
|-----------|---------|-------------------|
| `data-cs-field` | Render form fields | 31-34 |
| `data-cs-grid` | Display data grids | 37-39 |
| `data-cs-chart` | Show charts | 39-41 |
| `data-cs-conditional` | Build query filters | 34-36 |
| `data-cs-typeahead` | Autocomplete inputs | Field examples |

### Widget Development Checklist

- [ ] **Plan your widget**: What data does it show? What actions does it perform?
- [ ] **Design the config**: What options do users need to set?
- [ ] **Handle permissions**: Check if user can access the data/actions
- [ ] **Error handling**: Show loading states, handle API failures gracefully
- [ ] **Responsive design**: Test on different screen sizes
- [ ] **Performance**: Limit data queries, use pagination for large datasets
- [ ] **Real-time updates**: Consider if data should refresh automatically

### Common Patterns

1. **Data Loading Pattern**:
   ```javascript
   $scope.loading = true;
   DataService.load().then(function(data) {
     $scope.data = data;
     $scope.loading = false;
   });
   ```

2. **Error Handling Pattern**:
   ```javascript
   .catch(function(error) {
     toaster.error({body: 'Failed to load data'});
     $scope.loading = false;
   });
   ```

3. **Permission Checking Pattern**:
   ```javascript
   $scope.canEdit = currentPermissionsService.availablePermission('alerts', 'update');
   ```

This guide takes you from basic AngularJS concepts to building sophisticated FortiSOAR widgets. Start with simple display widgets and gradually work up to interactive, real-time components. Remember to always refer back to the FortiSOAR Widget Development documentation for detailed API information!