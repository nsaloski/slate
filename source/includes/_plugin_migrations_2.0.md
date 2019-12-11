### 2.0.0

The big ticket item with 2.0.0 is the deprecation of CaymlandFactory which will be phased out during the 2.x release cycles and to be removed in 3.0. Where possible, please use direct dependency injection of services rather than relying on CaymlandFactory.

1. CaymlandFactory deprecated - use dependency injection of required services. Many CaymlandFactory helper functions have been replaced with [services](#services).
2. Models need to be registered as services using a specific nomenclature. See [Models](#models) for more information.
3. The static callback function for campaign actions and decisions has been deprecated. Please see [Extending Campaigns](#extending-campaigns) for more information on the new event based method. (Form submit actions, point triggers, and the like will follow suit with a similar migration throughout the lifecycle of 2.0 but for now still uses the static callback method).
4. Minimum PHP version has been increased to 5.6.19 and thus newer PHP code goodies are available for developers (traits, etc)
5. Themes have been completely revamped although the old format is still supported for now. Please see [Themes](#themes) for the new format.
6. Some routes for /leads/* were removed in favor of /contacts/*. I.e. /api/leads is now /api/contacts, /s/leads is now /s/contacts, and so forth. The route names were also updated. For example, `caymland_lead_action` is now `caymland_contact_action` and so forth. 
