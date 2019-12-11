### 1.2.0

1.2.0 deprecated the Caymland Addon with it's successor, the Plugin.  Caymland addons will continue to work but should be migrated to a plugin before Caymland 2.0 when support will be dropped.

The migration path is as follows:

1. Move the plugin from addons/ to plugins/
2. Replace the namespace `CaymlandAddon` with `CaymlandPlugin`
3. Replace the CaymlandFactory getModel() notation of `addon.` to `plugin.`
4. Replace the permission notation of `addon:` to `plugin:`
5. Change the plugin's bundle class to extend `PluginBundleBase` instead of `AddonBundleBase`
6. In the plugin's bundle class, use the function's `onPluginInstall()` and `onPluginUpdate()` instead of the deprecated `onInstall()` and `onUpdate()` 
7. Migrate Entity use of annotations for ORM to the static PHP function `loadMetadata()`
8. Migrate Entity use of annotations for the serializer (used with the REST API) to the static PHP function `loadApiMetadata()`
