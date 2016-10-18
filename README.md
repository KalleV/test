#Generic Dashboard List Settings Configuration Document:

A dashboard is LabShare's way of displaying information to the user about data that is stored in a SharePoint list. 
Follow these steps to create a LabShare dashboard:

1.  Create a SharePoint list.
2.  In your browser, input the following URL: (domain_name)/_layouts/ls/(release_name)/default.aspx#/facility/list/(list's internal SP name)/settings
    Example)  rnai.ncats.nih.gov/_layouts/ls/16.922/default.aspx#/facility/list/instruments/settings
3.  Configure the dashboard's settings using the JSON template below with all of the appropriate information. Replace the key "myDashboard" with a unique dashboard name of your choosing.
4.  Click the Save icon to save your new settings.
5.  In your browser, input the following URL: (domain_name)/_layouts/ls/(release_name)/default.aspx#/facility/list/(list's internal SP name)/dashboard/(dashboard's unique name)
    
Below is the placement of a dashboard inside the list settings:

        {
            "dashboards" : {
                "myDashboard" : {
                    "name" : <string>,
                    "title" : <string>,
                    "route" : <string>,
                    "data" : {
                        "rolesRequired" : [
                            0 : <string>,
                            1 : <string>
                        ]
                    },
                    "panels" : {
                        "firstPanel" : {...},
                        "secondPanel" : {...},
                             "Header" : {
                                  "preTitleActions" : [ 
                                       {
                                       "actionId" : <string>,
                                       "title" : <string>,
                                       "icon" : <string>
                                       }
                                  ],
                                  "postTitleActions" : []
                        "Filter" : {...}
                    },
                    "layout" : {
                        "layout" : row,
                        "panels" : [
                            0 : {
                                    "layout" : column,
                                    "width" : <string>,
                                    "panels" : [
                                        0 : Filter
                                    ]
                                },
                            1 : {
                                    "layout" : column,
                                    "panels" : [
                                        0 : firstPanel
                                    ]
                                }
                        ]
                    }
                }
            }
            "forms" : {
                "new" : {...},
                "view" : {...}
            }
        }
                    
##Properties
1.  <a href="#name">name</a> : text
2.  <a href="#title">title</a> : text
3.  <a href="#route">route</a> : text
4.  <a href="#data">data</a> : object
    1. <a href="#rolesRequired">rolesRequired</a> : array of strings
5.  <a href="#panels">panels</a> : object
    1. <a href="#Filter">Filter</a> : text
6.  <a href="#layout">layout</a> : object
    1. <a href="#layout.layout">layout</a> : row 
    2. <a href="#layout.panels">panels</a> : array of objects 
        1. <a href="#layout.panels.layout">layout</a> : column
        2. <a href="#width">width</a> : text
        3. <a href="#layout.panels.panels">panels</a> : array
            1. 0 : Filter
            2. 1 : (panel_name)


##Panels
  
The panels property contains a dictionary of panels, where the key is the panel's ID and the value is an object containing panel configuration.

The main panel configuration properties are:
  * `title` - The name for the panel. It will appear in the panel header.
  * `type` - The LabShare directive to use for the panel.
  * `Header` - Object containing specifications of the actions that will exist in the header.
  * `options` - An optional object used to pass configuration values to the directive. Each value can be referenced from within that directive's isolate scope.
  
####Defining an API request for the panel data

By default, the panel directive will make a GET request to the Facility's Items API to obtain the data rendered in the view.
To override the Items API GET request, you must specify a custom `apiPath`. If you override the default request,
the `listName` and `defaultDataView` panel options will not be used.

You can use the following properties in the panel options to customize the API request:
  * `apiHost` - [optional] The URL host
  * `apiPath` - [optional] The path to a custom views API (e.g. `/_api/packageName/resource/:view`). It __must__ contain a `':view'` parameter.
  * `apiParams` - [optional] - The `apiParams` object can contain key/value pairs used to fill in additional parameters of the custom `apiPath`.
  * `dataPropertyName` - [optional] - This is used to customize the key name on the response data object. Default: `items`. 
  For example, your custom API might respond with `{users: [...]}`. In this case, you would need to set the `dataPropertyName` to `users`.

Specifying request options:
```JSON
"myPanel": {
    "options": {
      "apiHost": "host",
      "apiPath": "path/to/api/:view",
      "apiParams": {},
      "dataPropertyName": "users"
    }
}
```

Specifying "apiParams":
```JSON
"apiHost": "host:port",                                    
"apiPath": "path/:someValue/to/api/:view",                 
"apiParams": {                                             
    "someValue": 5,
    "listName": "<A SharePoint List Name>",               
    "view": "<A SharePoint view name>",                    
    "facilityPath": "<A SharePoint subsite path>",        
    "siteID": "<A SharePoint site ID>"                    
}
```

With the above configuration, the panel would make a GET request to `http://host:port/<A SharePoint subsite path>/<A SharePoint site ID>/path/5/to/api/<A SharePoint view name>`.
The `siteID` is used internally to locate a SharePoint site.

A full panel configuration specifying a custom, cross-domain API request:

```JSON
{
  "name": "Instruments-test",
  "title": "Instruments",
  "data": {
    "rolesRequired": [
      "staff",
      "admin"
    ]
  },
  "panels": {
    "instrumentPanel": {
      "title": "Instruments",
      "type": "ls-list-view",
      "Header": {
        "preTitleActions": [],
        "postTitleActions": []
      },
      "options": {
        "listName": "Instruments",
        "defaultListView": "tile",
        "defaultDataView": "PROBE",
        "onTileSelection": "detail",
        "onGridSelection": "detail",
        "apiPath": "/facility/list/:listName/view/:view",
        "apiParams": {
          "listName": "Instruments",
          "view": "PROBE",
          "siteID": "ort",
          "facilityPath": "/instrumentation"
        },
        "apiMODE": "get",
        "tileOptions": {
          "tileTemplate": "packages/facility/ui/directives/templates/list/projectTile.html",
          "wrapTiles": true
        },
        "tileButtons": [],
        "height": 1,
        "viewSelectionDisabled": true,
        "cacheDisabled": true
      }
    },
    "Filter": {}
  },
  "layout": {
    "layout": "row",
    "panels": [
      {
        "layout": "column",
        "flex": 15,
        "panels": [
          "Filter"
        ]
      },
      {
        "layout": "column",
        "panels": [
          "instrumentPanel"
        ]
      }
    ]
  }
}
```

####Header definition

The two lists in which actions can be defined are in the `preTitleActions` list and `postTitleActions` list. The preTitleActions will be a list of buttons to the left of the panel title. The postTitleActions will be a list of buttons immediately to the right of the panel title.

The required properties for navigational and event actions are:
 * actionId - An identifier for the action
 * title - A Name or title for the action.
 * icon - A Font Awesome icon to use for the action

The required properties for directive actions are:
 * directive - the directive to be used
 * options - configuration for the directive action
 
There are three type of actions that can be specified in the header: event actions, navigational actions, and directive actions. 

#####Event actions
Event actions will broadcast a message from the `$rootScope` to the entire view. The event name is defined by the `panelId` the action is contained in and the `actionId` of that action. 
Example event action definition: 
```JSON
{"actionId": "setListView.grid", "title": "Set Grid View", "icon": "list"}
```
If the panelId for this action is `panel1`, the event name generated by user clicking on this action would be 'panel1.setListView.grid'. Since this event would be broadcast to `$rootScope`, any controller in the same view can define a listener for that action.

#####Navigation Actions
Navigation actions will navigate to the url defined in the path property of the action. Navigation actions can be configured to pass data to the next state by defining a `data` property that contains an array of properties to send to the next state. 

Example navigation action definition:
```JSON
{"actionId": "detail", "title": "Detail", "path": "/facility/projects/view/:ID", "icon": "pencil", "data": ["ID"]}
```

In order for navigation actions to know which object to pull the data from, the actions listen for the event `<panelId>.selectItem` which receives the selected data object as its first argument. This is triggered when you click on an item in the list or grid view. The "panelId" is the ID of the panel containing the action.

For the navigation url to be generated with the correct parameters, the list of values in the "data" array must match the property keys of the selected item.

######Correct
```javascript
// Selected item:
{ID: '12345', projectType: 'Microscopy'}

// Navigation action definition:
{"actionId": "details", "title": "Details", "path": "/facility/projects/:projectType/:ID", "icon": "pencil", "data": ["ID", "projectType"]}

// The resulting url:
'/facility/projects/Microscopy/12345'
```

#####Directive Actions
Directive actions can be use when it is necessary to specify an action with a template that is something other than a button and/or when specifying an action that requires custom behavior. 
Example directive action definition:
```JSON
{"directive": "ls-select-data-view", 
  "options": {
    "listName": "Projects",
    "defaultDataView": "All Items"
  }
}
```
ls-select-data-view is a directive that renders an md-select option which holds the list of views for the specified listName, "Projects".

Example panel definition: 

```JSON
"panels": {
 "panel1": {
    "title": "Projects",
    "type": "ls-list-view",
    "Header": {
      "preTitleActions": [
          {"actionId": "setListView.grid", "title": "Set Grid View", "icon": "list"},
          {"actionId": "setListView.tile", "title": "Set Tile View", "icon": "th"}
      ],
      "postTitleActions": [
          {"actionId": "refresh", "title": "Refresh", "icon": "refresh"},
          {"actionId": "add", "title": "Add", "path": "/facility/projects/add/new", "icon": "plus"},
          {"actionId": "detail", "title": "Detail", "path": "/facility/projects/view/:ID", "icon": "pencil", "data": ["ID"]}
      ]
    },
    "options": {
      "listName": "Projects",
      "defaultListView": "tile",
      "defaultDataView": "All Items",
      "onTileSelection": "detail"
    }
  }
}
```

This example will become `<ls-list-view options="options"></ls-list-view>`, where options will contain all additional properties of view1. 

##Layouts
Layout will contain a object layout definition which will be recursively parsed by the dashboard service in order to generate the final template. 

Example:
```JSON
"layout": {
        "layout": "column",
        "panels": [
            "panel1",
            "panel2"
        ]
}
```

This styling is based on Flexbox. The key properties for each container are the `layout`, `panels`,`width` and `flex`.
*`layout` specifies the layout to use for the panels. If "column" is specified, the panels will be laid out vertically and if "row" is specified the children will be laid out horizontally within the parent container. 
*`panels` is an array that contains either the panel id of the panel to render, or a nested container object with its own layout and panels. 
*`flex` specifies the manner in which the container should flex. Optional. The width config will disable the flex config.  Default value is `true`. 
More info of flex options at: https://material.angularjs.org/latest/layout/grid

Nested example: 
```JSON
    "layout": {
        "layout": "column",
        "panels": [
            "panel1",
            {
                "layout": "row",
                "flex": 30,
                "width":"300px",      
                "hide-sm":"true",     
                "hide-xs":"true",
                "panels": [
                    "panel2",
                    "panel3",
                    "panel4"
                ]
            },
            "panel5"
        ]
    }
```
The resulting template of this layout definition will be 3 rows. The first row containing just panel1. The second row containing panels 2-4 evenly spaced. The third row containing panel5. 

##Dictionary of Terms

<dl id="data">
  <dt>data</dt>
  <dd>Type: "object" </br>
      Description: This object is optional and contains any additional data to pass as route information. Default: "rolesRequired": ["user","staff","admin"]</dd>
</dl>

<dl id="Filter">
  <dt>Filter</dt>
  <dd>Type: "object" </br>
      Description: This object contains all of the filters for this dashboard.</dd>
</dl>

<dl id="layout">
  <dt>layout</dt>
  <dd>Type: "object" </br>
      Description: This object tells the dashboard service where to place the panels.</dd>
</dl>

<dl id="name">
  <dt>name</dt>
  <dd>Type: "string" </br>
      Description: This string is the unique name for the 'dashboard.uion' file. It will be used as the key for the 'dashboard.uion'.</dd>
</dl>

<dl id="panels">
  <dt>panels (dashboards.myDashboard.panels)</dt>
  <dd>Type: "object" </br>
      Description: This object is a dictionary of UI panel configuration. The panels are the UI elements that when put together constitute a dashboard.<br>
      Examples:
      <ul>
          <li>"instrumentPanel" : For an instrumentation dashboard. Shows either a grid or tile view of a collection of instruments.</li>
          <li>"Filter" : For any dashboard. Contains a search bar and a collection of checkboxes so the user can narrow down a list of results.</li>
      </ul>
  </dd>
</dl>

<dl id="layout.panels">
  <dt>panels (dashboard.myDashboard.layout.panels)</dt>
  <dd>Type: "array of objects" </br>
      Description: Each element in this array represents a UI element.</dd>
</dl>

<dl id="layout.panels.panels">
  <dt>panels (dashboard.myDashboard.layout.panels.panels)</dt>
  <dd>Type: "array" </br>
      Description: This is the name of the specific panel that you would like to appear in the dashboard.</dd>
</dl>

<dl id="route">
  <dt>route</dt>
  <dd>Type: "string" </br>
      Description: This text is optional. This is the route to use for the dashboard. If specified, the state will be automatically generated for you with the specified dashboard configuration.</dd>
</dl>

<dl id="title">
  <dt>title</dt>
  <dd>Type: "string" </br>
      Description: This text that will appear towards the top of the dashboard. This is usually the list's name but it does not have to be. </dd>
</dl>






















