# Zabbix-ACM [![Latest Release](https://img.shields.io/github/release/CloudRight/Zabbix-ACM.svg)](https://github.com/CloudRight/Zabbix-ACM/releases/latest)

🚦 Application Component Monitoring using Zabbix

## Introduction

Monitoring whether your application-server or database-server is running doesn't guarantee you that your application can actually connect to its database. Or perhaps your application communicates with external API's that might go down and lead to issues within your application.

With **Application Component Monitoring** (ACM) we provide means of monitoring individual components from within the application's perspective.

Your developers know best how their application works and how to monitor its health whereas they generally have little or no Zabbix experience.
With **ACM** developers can easily integrate health checks into their application, and make changes to it on their own (e.g. adding more components or removing old ones) *without* needing any manual changes in Zabbix by your Ops team. All changes in the healthcheck endpoint are automatically picked up within Zabbix. This significantly lowers the bar to have better insights into your application's health.

### Rewrite

This component is a from the ground up rewrite of the ([Zabbix-AppMonitor](https://github.com/syphernl/Zabbix-AppMonitor)).

In comparison with the *old* version:

- We no longer need multiple endpoints for components and status.
- We no longer need an External Script (with additional pip dependencies) for retrieving the status and sending this back to Zabbix.
- We no longer need to create item + trigger prototypes for each severity.

This rewrite significantly increases the useability and performance.

## Requirements

Because of the used functionality (e.g. Javascript, Overrides) Zabbix version **5.0+** is the minimum required version.

## Usage

Your web-application should include a special healthcheck endpoint, providing information about the components and their status.
For this you can create a dynamic endpoint or let your application generate a static JSON file.
An example can be found in [example/health.json].

The template uses the Zabbix HTTP Agent (built-in) to retrieve the healthcheck content using a single HTTP request.
The output of the healthcheck endpoint will **automatically** be used to create items, triggers and graphs for all of the items listed in the output. It will also be used to provide all of the items with their appropriate status.

### Installation

Import the template within your Zabbix installation.
This can be done via the Zabbix web-interface via `Configuration > Templates > Import` (top-right corner).

### Create host in Zabbix

For each application you want to monitor create a host within Zabbix (`Configuration > Hosts > Create Host`).

- `Name` can be anything you like, this is being used for referring to the host within the triggers
- `Groups` should have at least one group selected
- `Interfaces` should have at least one Agent interface with IP Address `0.0.0.0`. We are not making use of this interface, but Zabbix expects one.
- On the `Templates` tab link the `Application Component Monitoring` template to the host.
- On the `Tags` tab you can **optionally** specify any tags that should be applied to triggers etc. This is useful to distinguish between various teams within your company.
- On the `Macro` tab the following macro's are **required**:

| Macro        | Description                                               | Example value                     |
| ------------ | --------------------------------------------------------- | --------------------------------  |
| `{$ACM_URL}` | URL to your [Healthcheck endpoint](#healthcheck_endpoint) | `https://example.com/healthcheck` |

The following macro's are **optional** and can be used to override the defaults:

| Macro                          | Description                                                                   | Default value |
| ------------------------------ | ----------------------------------------------------------------------------- | ------------- |
| `{$ACM_INTERVAL}`              | Update interval for retrieving the component statuses                         | `2m`          |
| `{$ACM_HTTP_USER}`             | (Optional) username to use to query the endpoints                             |               |
| `{$ACM_HTTP_PASS}`             | (Optional) password to use to query the endpoints                             |               |
| `{$ACM_DISCOVERY_KEEP_PERIOD}` | Amount of time to keep items that are no longer part of the discovery result. | `0`           |

### Healthcheck endpoint

The application should include an endpoint following the structure as outlined in our [example](./example/health.json).

Each individual healthcheck component looks like this:

```json
  "some_api": {
    "status": 0,
    "name": "Some upstream API",
    "severity": "high"
  }
```

The **key** of the element should be a unique entry identifying the healthcheck.
It is recommended to use a lowercase value without spaces or dashes.

For the element the following sub-keys are required:

| Key         | Description                                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|
| `status`    | A numerical representation of the status. This can be:<br>- `0` - OK<br>- `1` - Degraded<br>- `2` - Failure |
| `name`      | A "friendly name" of the item to monitor, which is used within the Zabbix frontend (triggers)                                       |
| `severity`  | The text-based severity as used by Zabbix. Can be one of the following: `unclassified`, `information`, `warning`, `average`, `high`, `disaster` |

#### Caching recommendation

By default the health is queried every 2 minutes, but this can be changed using Host Macros.

If a healthcheck queries external API's that are rate-limited / paid per request you might want to consider caching the status locally for a short period of time (e.g. 10-15 minutes) if these do not need to be checked against the same interval as the other items.
This should lower the amount of calls to external API's.

Alternatively you could also generate a static JSON file using a cronjob and use this as healthcheck endpoint.

#### Securing the healthcheck endpoint

There are a few ways to secure the healthcheck endpoint:

- Use **IP Whitelisting** to allow requests to this endpoint solely from your Zabbix instance(s)
- Use **HTTP Basic Auth** to only allow access with credentials. Make sure you set the `{$ACM_HTTP_USER}` and `{$ACM_HTTP_PASS}` in the Host Macro.

## Share the Love

Like this project? Please give it a ★ on [our GitHub repository](https://github.com/CloudRight/Zabbix-ACM)! (it helps us **a lot**)

### Developing

If you are interested in being a contributor and want to get involved in developing this project we would love to hear from you!

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

## Copyright

Copyright © 2021 CloudRight

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```

## Trademarks

All other trademarks referenced herein are the property of their respective owners.
