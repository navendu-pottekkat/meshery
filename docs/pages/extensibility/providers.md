---
layout: default
title: "Extensibility: Providers"
permalink: extensibility/providers
type: Reference
#redirect_from: architecture/adapters
abstract: "Meshery offers support for more adapters than any other project or product in the world. Meshery uses adapters for managing the various service meshes."
language: en
list: include
---
Meshery offers Providers as a point of extensibility. With a built-in Local Provider (named "None"), Meshery Remote Providers are designed to be pluggable. Remote Providers offer points of extension to users / integrators to deliver enhanced functionality, using Meshery as a platform.

1. **Extensibility points offer clean separation of open vs closed source capabilities.**
   - Meshmap is an example of a feature to be delivered via Remote Provider. 
1. **Remote Providers should be able to offer custom RBAC, custom UI components, and custom backend components**
   - Dynamically loadable frameworks need to be identified or created to serve each of these purposes.

### Design Principles: Meshery Remote Provider Framework

Meshery's Remote Provider extensbility framework is designed to enable:

1. **Pluggable UI Functionality:**
   - Out-of-tree custom UI components with seamless user experience.
   - A system of remote retrieval of extension packages (ReactJS components and Golang binaries).

1. **Pluggable Backend Functionality:**
   - Remote Providers have any number of capabilities unbeknownst to Meshery.

1. **Pluggable AuthZ**
   - Design an extensible role based access control system such that Remote Providers can determine their own set of controls. Remote Providers to return JWTs with custom roles, permission keys and permission keychains.

![Providers](/assets/img/providers/provider_screenshot.png)

### What functionality do Providers perform?

What a given Remote Provider offers might vary broadly between providers. Meshery offers extension points that Remote Providers are able to use to inject different functionality - functionality specific to that provider.

- **Authentication and Authorization**
  - Examples: session management, two factor authentication, LDAP integration.
- **Long-Term Persistence**
  - Examples: Storage and retrieval of performance test results.
  - Examples: Storage and retrieval of user preferences.
- **Enhanced Visualization**
  - Examples: Creation of a visual service mesh topology.
  - Examples: Different charts (metrics), debug (log viewer), distributed trace explorers.
- **Reporting**
  - Examples: Using Meshery's GraphQL server to compose new dashboards.

## Types of providers

Two types of providers are defined in Meshery: `local` and `remote`. The Local provider is built-into Meshery. Remote providers are may be implemented by anyone or organization that wishes to integrate with Meshery. Any number of Remote providers may be available in your Meshery deployment.

### Remote Providers

Use of a Remote Provider, puts Meshery into multi-user mode and requires user authentication. Use a Remote provider when your use of Meshery is ongoing or used in a team environment (used by multiple people).

Name: **“Meshery”** (default)

- Enforces user authentication.
- Long-term term persistence of test results.
- Save environment setup.
- Retrieve performance test results.
- Retrieve conformance test results.
- Free to use.

### Local Provider

Use of the Local Provider, "None", puts Meshery into single-user mode and does not require authentication. Use the Local provider when your use of Meshery is intended to be shortlived.

Name: **“None”**

- No user authentication.
- Container-local storage of test results. Ephemeral.
- Environment setup not saved.
- No performance test result history.
- No conformance test result history.
- Free to use.

## Building a Provider

Meshery interfaces with Providers through a Go interface. The Provider implementations have to be placed in the code and compiled together today. A Provider instance will have to be injected into Meshery when the program starts.

Meshery keeps the implementation of Remote Providers separate so that they are brought in through a separate process and injected into Meshery at runtime (OR) change the way the code works to make the Providers invoke Meshery.

Providers as an object have the following attributes:

### Remote Provider Extension Points

Interwoven into Meshery’s web-based, user interface are a variety of extension points. Each extension point is carefully carved out to afford a seamless user experience. Each extension point is identified by a name and type. The following Meshery UI extension points are available:

- **Name:** navigator 
   **Type:** Menu Items
  **Description:** This is supposed to be a full page extension which will get a dedicated endpoint in the meshery UI. And will be listed in the meshery UI’s navigator/sidebar. Menu items may refer to full page extensions.

**Name:** user_prefs 
**Type:** Single Component
**Description:** This is supposed to be remote react components which will get placed in a pre-existing page and will not have a dedicated endpoint. As of now, the only place where this extension can be loaded is the “User Preference” section under meshery settings.

**Name:** /extension/<your name here>
**Type:** Full Page
Description: 

### Capabilities Endpoint Example
Providers as an object have the following attributes (this must be returned as a response to `/capabilities` endpoint):

```json
{
  "provider_type": "remote",
  "package_version": "v0.1.0",
  "package_url": "https://layer5labs.github.io/meshery-extensions-packages/provider.tar.gz",
  "provider_name": "Meshery",
  "provider_description": [
    "Persistent sessions",
    "Save environment setup",
    "Retrieve performance test results",
    "Free use"
  ],
  "extensions": {
    "navigator": [
      {
        "title": "MeshMap",
        "href": {
          "uri": "/meshmap",
          "external": false
        },
        "component": "provider/navigator/meshmap/index.js",
        "icon": "provider/navigator/img/meshmap-icon.svg",
        "link:": true,
        "show": true,
        "children": [
          {
            "title": "View: Single Mesh",
            "href": {
              "uri": "/meshmap/mesh/all",
              "external": false
            },
            "component": "navigator/meshmap/index.js",
            "icon": "navigator/img/singlemesh-icon.svg",
            "link": false,
            "show": true
          }
        ]
      }
    ],
    "user_prefs": [
      {
        "component": "userprefs/meshmap-preferences.js"
      }
    ]
  },
  "capabilities": [
    { "feature": "sync-prefs", "endpoint": "/user/preferences" },
    { "feature": "persist-results", "endpoint": "/results" },
    { "feature": "persist-result", "endpoint": "/result" },
    { "feature": "persist-smi-results", "endpoint": "/smi/results" },
    { "feature": "persist-metrics", "endpoint": "/result/metrics" },
    { "feature": "persist-smp-test-profile", "endpoint": "/user/test-config" }
  ]
}

```

Meshery enables you as a service mesh owner to customize your service mesh deployment.

## Managing your Remote Provider Extension Code

Remote Provider extensions are kept out-of-tree from Meshery (server and UI). You might need to build your extensions under the same environment and set of dependencies as Meshery. The Meshery framework of extensibility has been designed such that in-tree extensions can be safely avoided while still providing a robust platform from which to extend Meshery’s functionality. Often, herein lies the delineation of open vs. closed functionality within Meshery. Remote Providers can bring (plugin) what functionality that they want behind this extensible interface (more about Meshery extensibility), at least that is up to the point that Meshery has provided a way to plug that feature in.

Offering out-of-tree support for Meshery extensions means that:

1. source code to your Meshery extensions are not required to be open source, 
1. liability to Meshery’s stability is significantly reduced, avoiding potential bugs in extended components. 

Through clearly defined extension points, Meshery extensions may be offered as closed source capabilities that plug into open source Meshery code. To facilitate integration of your Meshery extensions, you might automate the building and releasing of your separate, but interdependent code repositories. You will be responsible for sustaining both your ReactJS and Golang-based extensions.