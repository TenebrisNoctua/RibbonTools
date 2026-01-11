# Plugin Components

In Roblox Studio, supposedly for built-in plugins to communicate with internal C++ APIs, there exists a method for the `plugin` object called `plugin:GetPluginComponent(name: string)`. On the API reference, this method is shown to return a `Variant`, or `any`. In reality, it returns an internal object called `CommandData`.

This `CommandData` userdata object does not have any documentation or any API reference. Due to this, I've decided to write a small documentation about what kind of `CommandData` objects there are, and how they're used internally. Because there exists almost no information about this object, this documentation may be inaccurate. Take it with a grain of salt.

For each requested component, each `CommandData` will have different methods and properties. So I will mention these specialized `CommandData` objects with the names that are used to retrieve them.

-----

# Control Category Components

Each control in Roblox Studio can have different categories of functionality. Some might be just simple actions, others might be more related to a setting. There are currently 5 categories of controls in Roblox Studio: `Actions`, `Settings`, `Widgets`, `Tools`, `Panels`.

The most important part of a specific category's data table is the `Uri` property. It determines which plugin and which item/button it belongs to. 

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

### `ActionsComponent:GetAsync(targetUris: {{[string]: string}}): {{[string]: any}}`

This method allows you to get the data of an action from the target uri(s).

### `ActionsComponent:ActivateAsync(uri: {[string]: string}): ()`

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

### `ActionsComponent:BindToActivatedAsync(uri: {[string]: string}): RBXScriptSignal`

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
Each one of these actions allow you to trigger different kinds of functionality. "Group as Folder" groups the selected selected `Instance` as a `Folder`, while "Group as Model" groups it as a `Model`. 

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

### `SettingsComponent:GetAsync(targetUris: {{[string]: string}}): {{[string]: any}}`

This method allows you to get the data of a setting from the target uri(s).

### `SettingsComponent:UpdateAsync(targetSettings: {{[string]: any}}): {{[string]: any}}`

This method allows you to update the data of the target setting(s).

The required type of the `targetSettings` table is this:

```luau
type targetSettings = {{
    Uri: {
		Category: string,
		DataModel: string,
		ItemId: string,
		PluginId: string
	},
    Value: any
}}
```

Additional values can be added or removed to the table, depending on the setting.

### `SettingsComponent:BindAsync(uri: {[string]: string}): RBXScriptSignal`

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






