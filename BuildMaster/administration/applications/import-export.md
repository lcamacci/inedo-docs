﻿---
title: Advanced Import & Export
sequence: 60
show-headings-in-nav: true
---

Behind the scenes, the [Application Template](application-templates) and [Backup & Restore](backup-restore) features use the application import and export feature.

## Exporting Applications {#exporting data-title="Exporting Applications"}

You can open the application export page from the your applications advanced settings page (Backup... > Advanced), or from Admin > Export Applications. 

When exporting applications, you have the following options:

{.docs}
- **Applications** - list of applications to export; this option will only appear when the page is accessed from the administration section
- **Package name** - optional; the name of the universal package to create; this defaults to the application name, and is ignored if multiple applications are selected
- **Package version** - optional, the version of the universal package to create; this defaults to 0.0.0
- **Include history** - when selected, the release history and deployment logs will be included in the package. Note that this does not include artifacts

You will also select where to publish the package from a list of Package Sources.

{.docs}
- **Package sources** are configured as a feed on a ProGet instance with a feed url will become a repository for packages that you export, import, backup, restore, or templatize. A package source is simply configured as a url and credentials to a ProGet feed for univeral packages. 


## Importing Applications {#importing data-title="Importing Applications"}
You can import an application or template that has been exported to a pre-configured package source. This will save time by minimizing the time needed to setup an application that would have similar plans and pipelines.

To import an application go to Admin > Import Application. When importing, you must first select a source package:

You will also be presented with the following options:

{.docs}
- **Application name** - optional; the name of the application to create, this will default to the package name
- **Import history** - when set, any of the release and deployment history that is included in the package will also be imported

Before importing from a feed, you will need to setup an Inedo Product resource credential with the URL and optionally an API key to your ProGet server.

## Package File Format {#package-file-formate data-title="Package File Format"}

Applications will be exported as a standard [universal package](/docs/proget/core-concepts/packages) which is essentially a zip file containing application configuration and history, along with a JSON-based manifest file (`upack.json`) that describes the contents of the package.

In addition to the standard name and version properties, BuildMaster will include `a _exportDate` and `_bmVersion` property in `upack.json`. The package contents will be a collection of JSON-formatted files including:

| Filename | Description |
| -------- | ----------- |
| **app.json** | Information about the application |
| **deployables.json** | Deployables and deployable variables |
| **privileges.json** | Application-scoped privileges |
| **pipelines.json** | Pipelines |
| **global/pipelines.json** | Global pipelines that the application uses |
| **release-templates.json** | Release templates |
| **scripts.json** | Script assets |
| **text-templates.json** | Text template assets |
| **database-change-scripts.json** | Database change scripts |
| **config-files.json** | Configuration file assets |
| **issue-sources.json** | Issue sources configured for the application |
| **issues.json** | Already synchronized issues and their status |
| **plans.json** | Deployment plans |
| **counts.json** | Index/counts of all of the items in the import package |
| **releases.json** | Historic; releases |
| **builds.json** | Historic; builds |
| **promotions.json** | Historic; build deployments to stages |
| **executions.json** | Historic; execution records |
| **log-scopes.json** | Historic; log "scopes" (headings) in log files |
| **log-entries.json** | Historic; log text entries within a scope |

This is not an exhaustive list of files, and because these files are intended to only be used by BuildMaster, the precise format of these files is documented only in the source code (but you can [request source code access](/contact)).
