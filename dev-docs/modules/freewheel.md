---
layout: page_v2
page_type: module
title: Module - Freewheel
description: Passes key value targeting to Freewheel SDK for adpod mediaType adUnits.
module_code : freeWheelAdserverVideo
display_name : Freewheel Video Support
enable_download : true
vendor_specific: true
sidebarType : 1
---

# Freewheel Module

This module returns the targeting key value pairs for the FreeWheel ad server.

## How to use the module

If you are using FreeWheel as your ad server for long-form header bidding then include this module while creating Prebid.js build. Use the exposed getTargeting method to get targeting key value pairs.

### Example

```javascript
pbjs.adServers.freewheel.getTargeting({
    codes: [adUnitCode1],
    callback: function(err, targeting) {
        //pass targeting to player api
    }
});

// Sample return value when brandCategoryExclusion flag is set to true
{
  'adUnitCode-1': [
    {
      'hb_pb_cat_dur': '10.00_400_15s'
    },
    {
      'hb_pb_cat_dur': '15.00_402_30s'
    },
    {
      'hb_cache_id': '123'
    }
  ]
}

// Sample return value
{
  'adUnitCode-1': [
    {
      'hb_pb_cat_dur': '10.00_15s'
    },
    {
      'hb_pb_cat_dur': '15.00_30s'
    },
    {
      'hb_cache_id': '123'
    }
  ]
}
```

### Parameters

{: .table .table-bordered .table-striped }

| Name | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| `codes` | Array<string> | AdUnit codes to retrieve targeting for | yes | n/a |
| `callback` | Function | Function invoked with error and targeting map | yes | n/a |

The values returned by `getTargeting` are concatenation of CPM, industy code, and video duration. FreeWheel SDK will send those values to FreeWheel Ad Server within the following query:

```text
http://[customerId].v.fwmrm.net/ad/g/1[globalParams];hb_pb_cat_dur=10.00_400_15s&hb_pb_cat_dur=15.00_402_30s&hb_cacheid=123;[ParamsForSlot1];[ParamsForSlot2];...;[ParamsForSlotN];
```

## Further Reading

- [Prebid.js for Developers](/dev-docs/getting-started.html)  
- [Prebid Video](/prebid-video/video-overview.html)  
- [Category Translation](/dev-docs/modules/categoryTranslation.html)
