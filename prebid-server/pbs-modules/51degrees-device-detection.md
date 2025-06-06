---
layout: page_v2
page_type: pbs-module
title: Prebid Server 51Degrees Device Detection Module
display_name : 51Degrees Device Detection Module
sidebarType : 5
---

{: .alert.alert-warning :}
51Degrees module operates using a data file. You can get started with a free Lite data file that can be downloaded from the [51Degrees data repository](https://github.com/51Degrees/device-detection-data/blob/main/51Degrees-LiteV4.1.hash). The Lite file is updated monthly and detects limited device information. To access the daily updated full data file with comprehensive device information please [contact 51Degrees](https://51degrees.com/contact-us).

# 51Degrees Device Detection Module
{:.no_toc}

* TOC
{:toc }

## Overview

The 51Degrees module enriches an incoming OpenRTB request with [51Degrees Device Data](https://51degrees.com/documentation/_device_detection__overview.html).

The module sets the following fields of the device object: `make`, `model`, `os`, `osv`, `h`, `w`, `ppi`, `pxratio` - interested bidder adapters may use these fields as needed.  In addition the module sets `device.ext.fiftyonedegrees_deviceId` to a permanent device ID which can be rapidly looked up in on premise data exposing over 250 properties including the device age, chip set, codec support, and price, operating system and app/browser versions, age, and embedded features.

## Operation Details

### Evidence

The module uses `device.ua` (User Agent) and `device.sua` (Structured User Agent) provided in the oRTB request payload as input (or 'evidence' in 51Degrees terminology).  There is a fallback to the corresponding HTTP request headers if any of these are not present in the oRTB payload - in particular: `User-Agent` and `Sec-CH-UA-*` (aka User-Agent Client Hints).  To make sure Prebid.js sends Structured User Agent in the oRTB payload - we strongly advise publishers to enable userAgentHints for their wrappers:

```js
pbjs.setConfig({
    firstPartyData: {
        uaHints: [
          'architecture',
          'model',
          'platform',
          'platformVersion',
          'fullVersionList',
        ]
    }
})
```

### Data File Updates

The module operates **fully autonomously and does not make any requests to any cloud services in real time to do device detection**. This is an [on-premise data](https://51degrees.com/developers/deployment-options/on-premise-data) deployment in 51Degrees terminology. The module operates using a local data file that is loaded into memory fully or partially during operation. The data file is occasionally updated to accommodate new devices, so it is recommended to enable automatic data updates in the module configuration. Alternatively `watch_file_system` option can be used and the file may be downloaded and replaced on disk manually. See the configuration options below.

## Setup

The 51Degrees module operates using a data file. You can get started with a free Lite data file that can be downloaded here: [51Degrees-LiteV4.1.hash](https://github.com/51Degrees/device-detection-data/blob/main/51Degrees-LiteV4.1.hash).  The Lite file is capable of detecting limited device information, so if you need in-depth device data, please contact 51Degrees to obtain a license: [https://51degrees.com/contact-us](https://51degrees.com/contact-us?ContactReason=Free%20Trial).

Put the data file in a file system location writable by the system account that is running the Prebid Server module and specify that directory location in the configuration parameters. The location needs to be writable if you would like to enable [automatic data file updates](https://51degrees.com/documentation/_features__automatic_datafile_updates.html).

### Execution Plan

This module supports running at two stages:

* entrypoint: this is where incoming requests are parsed and device detection evidences are extracted.
* raw-auction-request: this is where outgoing auction requests to each bidder are enriched with the device detection data

We recommend defining the execution plan right in the account config
so the module is only invoked for specific accounts. See below for an example.

### Global Config

There is no host-company level config for this module.

### Account-Level Config

To start using current module in PBS-Java you have to enable module and add `fiftyone-devicedetection-entrypoint-hook` and `fiftyone-devicedetection-raw-auction-request-hook` into hooks execution plan inside your config file:
Here's a general template for the account config used in PBS-Java:

```yaml
hooks:
  fiftyone-devicedetection:
    enabled: true
  host_execution_plan: >
    {
      "endpoints": {
        "/openrtb2/auction": {
          "stages": {
            "entrypoint": {
              "groups": [
                {
                  "timeout": 10,
                  "hook-sequence": [
                    {
                      "module-code": "fiftyone-devicedetection",
                      "hook-impl-code": "fiftyone-devicedetection-entrypoint-hook"
                    }
                  ]
                }
              ]
            },
            "raw-auction-request": {
              "groups": [
                {
                  "timeout": 10,
                  "hook-sequence": [
                    {
                      "module-code": "fiftyone-devicedetection",
                      "hook-impl-code": "fiftyone-devicedetection-raw-auction-request-hook"
                    }
                  ]
                }
              ]
            }
          }
        }
      }
    }
```

Sample module enablement configuration in JSON and YAML formats:

```json
{
  "modules":
  {
    "fiftyone-devicedetection":
    {
      "data-file": {
        "path": "/path/to/51Degrees-LiteV4.1.hash"
      }
    }
  }
}
```

```yaml
  modules:
    fiftyone-devicedetection:
      data-file:
        path: "/path/to/51Degrees-LiteV4.1.hash" 
```

PBS-Go version of the same config:

In JSON: 

```json
{
  "hooks": {
    "enabled": true,
    "modules": {
      "fiftyonedegrees": {
        "devicedetection": {
          "enabled": true,
          "data_file": {
            "path": "path/to/51Degrees-LiteV4.1.hash",
            "make_temp_copy": true,
            "update": {
              "auto": true,
              "url": "<optional custom URL>",
              "polling_interval": 1800,
              "license_key": "<your_license_key>",
              "product": "V4Enterprise",
              "watch_file_system": "true"
            }
          }
        }
      },
      "host_execution_plan": {
        "endpoints": {
          "/openrtb2/auction": {
            "stages": {
              "entrypoint": {
                "groups": [
                  {
                    "timeout": 10,
                    "hook_sequence": [
                      {
                        "module_code": "fiftyonedegrees.devicedetection",
                        "hook_impl_code": "fiftyone-devicedetection-entrypoint-hook"
                      }
                    ]
                  }
                ]
              },
              "raw_auction_request": {
                "groups": [
                  {
                    "timeout": 10,
                    "hook_sequence": [
                      {
                        "module_code": "fiftyonedegrees.devicedetection",
                        "hook_impl_code": "fiftyone-devicedetection-raw-auction-request-hook"
                      }
                    ]
                  }
                ]
              }
            }
          }
        }
      }
    }
  }
}
```

and YAML format:

```yaml
hooks:
  enabled: true
  modules:
    fiftyonedegrees:
      devicedetection:
        enabled: true
        data_file:
          path: path/to/51Degrees-LiteV4.1.hash
          make_temp_copy: true
          update:
            auto: true
            url: "<optional custom URL>"
            polling_interval: 1800
            license_key: "<your_license_key>"
            product: V4Enterprise
            watch_file_system: 'true'
    host_execution_plan:
      endpoints:
        "/openrtb2/auction":
          stages:
            entrypoint:
              groups:
                - timeout: 10
                  hook_sequence:
                    - module_code: fiftyonedegrees.devicedetection
                      hook_impl_code: fiftyone-devicedetection-entrypoint-hook
            raw_auction_request:
              groups:
                - timeout: 10
                  hook_sequence:
                    - module_code: fiftyonedegrees.devicedetection
                      hook_impl_code: fiftyone-devicedetection-raw-auction-request-hook
```

## Module Configuration Parameters (for PBS-Java and PBS-Go)

The parameter names are specified with full path using dot-notation.  F.e. `section-name` .`sub-section` .`param-name` would result in this nesting in the JSON configuration:

```json
{
  "section-name": {
    "sub-section": {
      "param-name": "param-value"
    }
  }
}
```

{: .table .table-bordered .table-striped }

| PBS-Java Name | PBS-Go Name | Required| Type | Default  value | Description |
|:-------|:-------|:------|:------|:------|:---------------------------------------|
| `account-filter` .`allow-list` |  `account_filter` .`allow_list`  |  No | list of strings | [] (empty list) | A list of account IDs that are allowed to use this module - only relevant if enabled globally for the host. If empty, all accounts are allowed. Full-string match is performed (whitespaces and capitalization matter). |
| `data-file` .`path`    |  `data_file` .`path`  |  **Yes** | string | null |The full path to the device detection data file. Sample file can be downloaded from the [51Degrees data repository](https://github.com/51Degrees/device-detection-data/blob/main/51Degrees-LiteV4.1.hash), or get an Enterprise data file from the [51Degrees pricing page](https://51degrees.com/pricing). |
| `data-file` .`make-temp-copy` | `data_file` .`make_temp_copy` | No | boolean | true | If true, the engine will create a temporary copy of the data file rather than using the data file directly. |
| `data-file` .`update` .`auto` | `data_file` .`update` .`auto` | No | boolean | true | If enabled, the engine will periodically (at predefined time intervals - see `polling-interval` parameter) check if new data file is available. When the new data file is available engine downloads it and switches to it for device detection. If custom `url` is not specified `license_key` param is required. |
| `data-file` .`update` .`on-startup` | `data_file` .`update` .`on_startup` | No | boolean | true | If enabled, engine will check for the updated data file right away without waiting for the defined time interval. |
| `data-file` .`update` .`url` | `data_file` .`update` .`url` | No | string | null | Configure the engine to check the specified URL for the availability of the updated data file. If not specified the [51Degrees distributor service](https://51degrees.com/documentation/4.4/_info__distributor.html) URL will be used, which requires a License Key. |
| `data-file` .`update` .`license-key` | `data_file` .`update` .`license_key` | No | string | null | Required if `auto` is true and custom `url` is not specified. Allows to download the data file from the [51Degrees distributor service](https://51degrees.com/documentation/4.4/_info__distributor.html). |
| `data-file` .`update` .`watch-file-system` | `data_file` .`update` .`watch_file_system` | No | boolean | true | If enabled the engine will watch the data file path for any changes, and automatically reload the data file from disk once it is updated. |
| `data-file` .`update`.`polling-interval` | `data_file` .`update` .`polling_interval` | No | int | 1800 | The time interval in seconds between consequent attempts to download an updated data file. Default = 1800 seconds = 30 minutes. |
| `performance` .`profile` | `performance` .`profile` | No | string | `Balanced` | `performance.*` parameters are related to the tradeoffs between speed of device detection and RAM consumption or accuracy. `profile` dictates the proportion between the use of the RAM (the more RAM used - the faster is the device detection) and reads from disk (less RAM but slower device detection). Must be one of: `LowMemory`, `MaxPerformance`, `HighPerformance`, `Balanced`, `BalancedTemp`, `InMemory`. Defaults to `Balanced`.  |
| `performance` .`concurrency` | `performance` .`concurrency` | No | int | 10 |  Specify the expected number of concurrent operations that engine does. This sets the concurrency of the internal caches to avoid excessive locking. Default: 10.  |
| `performance` .`difference` | `performance` .`difference` | No | int | 0 |  Set the maximum difference to allow when processing evidence (HTTP headers). The meaning is the difference in hash value between the hash that was found, and the hash that is being searched for. By default this is 0. For more information see [51Degrees documentation](https://51degrees.com/documentation/_device_detection__hash.html).  |
| `performance` .`drift` | `performance` .`drift` | No | int | 0 |  Set the maximum drift to allow when matching hashes. If the drift is exceeded, the result is considered invalid and values will not be returned. By default this is 0. For more information see [51Degrees documentation](https://51degrees.com/documentation/_device_detection__hash.html).  |
| `performance` .`allow-unmatched` | `performance` .`allow_unmatched` | No | boolean | false |  If set to false, a non-matching evidence will result in properties with no values set. If set to true, a non-matching evidence will cause the 'default profiles' to be returned. This means that properties will always have values (i.e. no need to check .hasValue) but some may be inaccurate. By default, this is false. |

## Running the demo (PBS-Java)

1. Build the server bundle JAR as described in [Build Project](https://github.com/prebid/prebid-server-java/blob/master/docs/build.md#build-project), e.g.

```bash
mvn clean package --file extra/pom.xml
```

{:start="2"}
2. Download `51Degrees-LiteV4.1.hash` from [GitHub](https://github.com/51Degrees/device-detection-data/blob/main/51Degrees-LiteV4.1.hash) and put it in the project root directory.

```bash
curl -o 51Degrees-LiteV4.1.hash -L https://github.com/51Degrees/device-detection-data/raw/main/51Degrees-LiteV4.1.hash
```

{:start="3"}
3. Start server bundle JAR as described in [Running project](https://github.com/prebid/prebid-server-java/blob/master/docs/run.md#running-project), e.g.

```bash
java -jar target/prebid-server-bundle.jar --spring.config.additional-location=sample/prebid-config-with-51d-dd.yaml
```

{:start="4"}
4. Run sample request against the server as described in [the sample directory](https://github.com/prebid/prebid-server-java/tree/master/sample), e.g.

```bash
curl http://localhost:8080/openrtb2/auction --data @extra/modules/fiftyone-devicedetection/sample-requests/data.json
```

{:start="5"}
5. See the `device` object be enriched

```diff
                 "device": {
-                    "ua": "Mozilla/5.0 (Linux; Android 11; SM-G998W) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Mobile Safari/537.36"
+                    "ua": "Mozilla/5.0 (Linux; Android 11; SM-G998W) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Mobile Safari/537.36",
+                    "os": "Android",
+                    "osv": "11.0",
+                    "h": 3200,
+                    "w": 1440,
+                    "ext": {
+                        "fiftyonedegrees_deviceId": "110698-102757-105219-0"
+                    }
                 },
```

[Enterprise](https://51degrees.com/pricing) files can provide even more information:

```diff
                 "device": {
                     "ua": "Mozilla/5.0 (Linux; Android 11; SM-G998W) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Mobile Safari/537.36",
+                    "devicetype": 1,
+                    "make": "Samsung",
+                    "model": "SM-G998W",
                     "os": "Android",
                     "osv": "11.0",
                     "h": 3200,
                     "w": 1440,
+                    "ppi": 516,
+                    "pxratio": 3.44,
                     "ext": {
-                        "fiftyonedegrees_deviceId": "110698-102757-105219-0"
+                        "fiftyonedegrees_deviceId": "110698-102757-105219-18092"
                     }
```

## Running the demo (PBS-Go)

1. Download dependencies:

```bash
go mod download
```

{:start="2"}
2. Replace the original config file `pbs.json` (placed in the repository root or in `/etc/config`) with the sample config file:

```bash
cp modules/fiftyonedegrees/devicedetection/sample/pbs.json pbs.json
```

{:start="3"}
3. Download `51Degrees-LiteV4.1.hash` from [[GitHub](https://github.com/51Degrees/device-detection-data/blob/main/51Degrees-LiteV4.1.hash)] and put it in the project root directory.

```bash
curl -o 51Degrees-LiteV4.1.hash -L https://github.com/51Degrees/device-detection-data/raw/main/51Degrees-LiteV4.1.hash
```

{:start="4"}
4. Create a directory for sample stored requests (needed for the server to run):

```bash
mkdir -p sample/stored
```

{:start="5"}
5. Start the server:

```bash
go run main.go
```

{:start="6"}
6. Run sample request:

```bash
curl \
--header "Content-Type: application/json" \
http://localhost:8000/openrtb2/auction \
--data @modules/fiftyonedegrees/devicedetection/sample/request_data.json
```

## Maintainer contacts

Any suggestions or questions can be directed to [support@51degrees.com](mailto:support@51degrees.com) e-mail.

Or just open new [issue](https://github.com/prebid/prebid-server-java/issues/new) or [pull request](https://github.com/prebid/prebid-server-java/pulls) in this repository.

## Further Reading

* [Prebid Server Module List](/prebid-server/pbs-modules/index.html)
