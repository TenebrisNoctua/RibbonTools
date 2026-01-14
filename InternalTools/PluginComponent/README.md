# Plugin Component

In Roblox Studio, supposedly for built-in plugins to communicate with internal C++ APIs, there exists a method for the `plugin` object called `plugin:GetPluginComponent(name: string)`. On the API reference, this method is shown to return a `Variant`, or `any`. In reality, it returns an internal object called `CommandData`.

This `CommandData` userdata object does not have any documentation or any API reference. Due to this, I've decided to write a small documentation about what kind of `CommandData` objects there are, and how they're used internally. Because there exists almost no information about this object, this documentation may be inaccurate at certain places. Take it with a grain of salt.

For each requested component, each `CommandData` will have different methods and properties. So I will mention these specialized `CommandData` objects with the names that are used to retrieve them.

It must be noted that `plugin:GetPluginComponent(name: string)` API is not truly a full component API. It may give you other types of `CommandData` objects that may not be related to a specific plugin component. 

-----

# Plugin Component Categories

This section is about `CommandData` objects that are used to grant advanced functionality to built-in plugin components.

Each plugin component in Roblox Studio can have different categories of functionality. Some might be just simple actions, others might be more related to a setting. 

There are currently 5 categories of components in Roblox Studio: `Actions`, `Settings`, `Widgets`, `Tools`, `Panels`.

*(Note: This list does not represent all `CommandData` objects that can be retrieved through `:GetPluginComponent()` API. It is simply a list of all currently known component related APIs.)*

## Uri

```luau
export type StudioUriDataModel = "Edit" | "PlayClient" | "PlayServer" | "Standalone" | "Null"
export type StudioUriPluginType = "Cloud" | "Local" | "Asset" | "Standalone"
export type StudioUriCategory = "Actions" | "Panels" | "Settings" | "Tools" | "Widgets"

export type StudioUri = {
	DataModel: StudioUriDataModel?,
	PluginType: StudioUriPluginType?,
	PluginId: string?,
	Category: StudioUriCategory?,
	ItemId: string?,
}
```

Every plugin component has a data field, called "Uri", which allows you to retrieve information about that specific plugin component. It gives you important information such as which plugin the component has been created from, and its `ItemId`, which is used to identify the component itself. It also tells you which category the component belongs to. 

-----

## Actions Component

An action is a functionality attached to a certain plugin or button. Every plugin can create an action through its buttons. For example, each time you create a `PluginToolbarButton` with an unique id, an action gets created which is attached to this button.

Here's what an example action data looks like:

```luau
{
    ["Checkable"] = true,
    ["Checked"] = false,
    ["DisableAsTool"] = false,
    ["Enabled"] = true,
    ["Exists"] = true,
    ["Icon"] = "rbxtemp://XXX",
    ["Shortcuts"] = {},
    ["Text"] = "Output",
    ["Tooltip"] = "See the output feed of your scripts",
    ["Uri"] =  ▼  {
       ["Category"] = "Actions",
       ["DataModel"] = "Standalone",
       ["ItemId"] = "Toggle",
       ["PluginId"] = "Output"
    },
    ["Visible"] = true
}
```

This is the action data for the Output window. It contains information about the action that is attached to the Output's toggle button.
The `Icon` property, for example, determines the icon that will be on the Output's toggle button on the Ribbon.

### `ActionsComponent:ListAsync(): {{[string]: any}}`

This method returns all of the currently available actions in a table.

### `ActionsComponent:GetAsync(targetUris: {StudioUri}): {{[string]: any}}`

This method allows you to get the data of an action from the target uri(s).

### `ActionsComponent:ActivateAsync(uri: StudioUri): ()`

This method activates a specific action with the provided `uri`. 

For example:

```luau
Plugin:GetPluginComponent("Actions"):ActivateAsync({
    Category = "Actions",
    DataModel = "Standalone",
    ItemId = "Toggle",
    PluginId = "Output"
})
```

This will activate/de-activate the Output window.

### `ActionsComponent:BindToActivatedAsync(uri: StudioUri): RBXScriptSignal`

This method returns an `RBXScriptSignal` that is fired when the action from the provided `uri` is activated.

For example:

```luau
local Activated: RBXScriptSignal = Plugin:GetPluginComponent("Actions"):BindToActivatedAsync({
	Category = "Actions",
	DataModel = "Standalone",
	ItemId = "Toggle",
	PluginId = "Output"
})

Activated:Connect(function(uri: {[any]: any})
    print("Fired action with the uri: ", uri)
end)
```

-----

## Settings Component

This component is pretty similar to the actions component, except it represents a current setting of a control. This means that instead of firing one action every time, you can have multiple actions that trigger different functionality, and set one of them as the default.

An example is the "Group" `SplitButton` on the Home tab of the Ribbon. It has two actions: "Group as Folder" and "Group as Model".
Each one of these actions allow you to trigger different kinds of functionality. "Group as Folder" groups the selected `Instance` as a `Folder`, while "Group as Model" groups it as a `Model`. 

Upon triggering one of these actions, the control will default to using that action. So if you were to select "Group as Folder", by default, every time you activate the control normally, the selected `Instance` will be grouped as a `Folder`.

Here's what an example setting data looks like:

```luau
{
    ["Enabled"] = true,
    ["Text"] = "",
    ["TextKey"] = "Studio.Ribbon.Plugin.Setting_Group",
    ["TooltipKey"] = "Studio.Ribbon.Plugin.Tooltip_Group",
    ["Uri"] =  ▼  {
       ["Category"] = "Settings",
       ["DataModel"] = "Standalone",
       ["ItemId"] = "Group",
       ["PluginId"] = "BuilderTools"
    },
    ["Value"] = 0,
    ["Values"] =  ▼  {
       [1] =  ▼  {
          ["Action"] =  ▼  {
             ["Category"] = "Actions",
             ["DataModel"] = "Standalone",
             ["ItemId"] = "GroupAsModel",
             ["PluginId"] = "BuilderTools"
          },
          ["Id"] = "GroupAsModel",
          ["Visible"] = true
       },
       [2] =  ▼  {
          ["Action"] =  ▼  {
             ["Category"] = "Actions",
             ["DataModel"] = "Standalone",
             ["ItemId"] = "GroupAsFolder",
             ["PluginId"] = "BuilderTools"
          },
          ["Id"] = "GroupAsFolder",
          ["Visible"] = true
       }
    },
    ["Visible"] = true
}                
```

### `SettingsComponent:ListAsync(): {{[string]: any}}`

This method returns all of the currently available settings in a table.

### `SettingsComponent:GetAsync(targetUris: {StudioUri}): {{[string]: any}}`

This method allows you to get the data of a setting from the target uri(s).

### `SettingsComponent:UpdateAsync(targetSettings: TargetSettings): {{[string]: any}}`

This method allows you to update the data of the target setting(s).

The required type of the `targetSettings` table is as follows:

```luau
type TargetSettings = {{
    Uri: StudioUri,
    Value: any
}}
```

Additional values can be added to the table, depending on the setting.

### `SettingsComponent:BindAsync(uri: StudioUri): RBXScriptSignal`

This method returns an `RBXScriptSignal` that is only fired when the setting from the provided `uri` has been set to a new action.

For example:

```luau
local SettingChanged: RBXScriptSignal = Plugin:GetPluginComponent("Settings"):BindAsync({
    Category = "Settings",
    DataModel = "Standalone",
    ItemId = "Group",
    PluginId = "BuilderTools"
})

SettingChanged:Connect(function(uri: {[any]: any})
    print("Changed setting with the uri: ", uri)
end)
```

-----

## Panels Component

This component type is used to manage internal widgets. It's primarily used with `QWidgetPluginGui` instances.

### `PanelsComponent:SetAttachmentAsync(uri: StudioUri, params: Params)`

This method allows you to attach a widget to another. The `uri` argument represents the uri of the widget that you want to attach. The `params` argument represents a table, which contains information about the target widget that you want to attach your widget to, with additional properties to determine the final position.

```luau
type Params = {
    TargetWidgetUri: StudioUri,
	TargetAnchorPoint: Vector2,
	SubjectAnchorPoint: Vector2,
	Offset: Vector2,
	AllowScreenOverflow: boolean
}
```

For example:

```luau
-- This sets a widget of a plugin below the "Collaborate" button.
Plugin:GetPluginComponent("Panels"):SetAttachmentAsync({
	ItemId = "XXXX", -- The id of the widget.
	DataModel = "Edit",
	PluginId = "XXXX", -- The id of the plugin.
	PluginType = "Local" -- If using a cloud plugin, change to "Cloud".
}, {
	TargetWidgetUri = {
		["Category"] = "Actions",
		["DataModel"] = "Standalone",
		["ItemId"] = "Open",
		["PluginId"] = "ManageCollaborators"
	},
	TargetAnchorPoint = Vector2.new(0, 0),
	SubjectAnchorPoint = Vector2.new(0, 0),
	Offset = Vector2.new(0, 0)
})
```

### `PanelsComponent:SetSizeAsync(uri: StudioUri, size: Vector2)`

This method allows you to set the size of a specific widget from its `uri`.

For example:

```luau
Plugin:GetPluginComponent("Panels"):SetSizeAsync({ 
	ItemId = "Ribbon",
	DataModel = "Standalone",
	PluginId = "Ribbon",
	PluginType = "Standalone"
}, Vector2.new(1920, 35))
```

-----

## Widgets Component

### `WidgetsComponent:RegisterAsync(targets: Targets)`

```luau
type Targets = {{
    Uri: {
		Category: string,
		DataModel: string,
		ItemId: string,
		PluginId: string
	},
    Widget: PluginGui,
    DEPRECATED_PluginGui: PluginGui
}}
```

### `WidgetsComponent:DeregisterAsync(targetUris: {{[string]: string}})`






