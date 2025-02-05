---
title: TeamCity
subtitle: Integrating TeamCity with BuildMaster
sequence: 200
keywords: buildmaster, builds, teamcity
show-headings-in-nav: true
---

[TeamCity](https://www.jetbrains.com/teamcity) is a web-based continuous integration server by JetBrains. The software can be hosted by JetBrains in the cloud (known as InCloud), or installed and managed on-premise (Standalone). TeamCity offers all the typical functionality of a CI server:

{.docs}
 - intelligent CI builds
 - support for unit tests and static analysis
 - artifact storage

TeamCity is such a powerful CI tool, and there are a lot of good reasons to continue using it:

{.docs}
 - **Save time and resources on retraining** - if your developers are already familiar with TeamCity and processes in your organization, they do not require training for a brand new system
 - **Reuse complex automations** - processes like automated testing are complex to develop and are often fragile, and moving these processes to a new system will require some degree of effort that might be better spent elsewhere
 - **Keep your existing infrastructure** - you can use the TeamCity infrastructure that you've already built and is providing you with value

::: {.attention .best-practice}
**See it live!** See an example of TeamCity integration by creating a new application using the *TeamCity Import* template.
:::

## Installing the TeamCity Extension {#extension data-title="TeamCity Extension"}

Simply navigate to the Admin > Extensions page in your instance of BuildMaster and click on the TeamCity extension to install it.

If your instance doesn't have internet access, you can [manually install the TeamCity extension](https://docs.inedo.com/docs/buildmaster/reference/extensions#manual-install) after downloading the [TeamCity Extension Package](https://proget.inedo.com/feeds/Extensions/inedox/TeamCity).

## Authentication with TeamCity {#authentication data-title="Authentication with TeamCity"}

[Resource Credentials](/docs/buildmaster/administration/resource-credentials) are used to connect BuildMaster to your Continuous Integration (CI) server, and each supported CI server has its own type of Resource Credential, and that type is defined in the corresponding extension. You can create as many resource credentials as you need, at both the system- and application-level, which means you can connect to multiple CI servers and share those connections across applications, letting your teams continue working with whatever tool each prefers.

Authentication with TeamCity uses the TeamCity-type [Resource Credential](/docs/buildmaster/administration/resource-credentials). These credentials are effectively a username and password, so we recommend creating a specific account with the minimum amount of privileges required to interact with TeamCity, typically the permissions to the specific project or build configuration. Public servers (i.e., open-source) or internal servers that are firewalled off that don't require authentication can simply use guest authentication by leaving the username and password empty.

To connect to a standalone instance of TeamCity, make sure the API URL of the resource credentials is configured to the hostname for your installation, for example: `https://teamcity.jetbrains.com`

## Queuing Builds in TeamCity {#queuing-builds data-title="Queuing Builds"}

BuildMaster supports queuing builds in TeamCity, which can be useful as an automated notification of a release team's desire to create a build artifact (or artifacts). To accomplish this, use the `TeamCity::Queue-Build` operation in a build plan. Once the build is created (and optionally waited on), the build number of the TeamCity build can be captured as an output variable using the `TeamCityBuildNumber => $buildNumber` property:

```
TeamCity::Queue-Build
(
	Credentials: KramericaTeamCity,
	Project: ProfitCalc,
	BuildConfiguration: v8builds,
	Artifact: profit-calc-web.zip,
	TeamCityBuildNumber => $tcBuildNumber
); 
```

Once this variable is captured, a [variable value renderer](/docs/buildmaster/administration/value-renderers) can be used to link `$TeamCityBuildNumber` back to TeamCity on the Build Overview page, for example:

```
<a class="ci-icon teamcity" href="https://teamcity.jetbrains.com/viewLog.html?buildNumber=$TeamCityBuildNumber&buildTypeId=acejump_buildplugin">Build #$TeamCityBuildNumber</a>
```

## Importing TeamCity Artifacts {#importing-builds data-title="Importing Builds"}

Once a build is queued, importing an artifact ensures BuildMaster will be able to deploy it to future stages whether TeamCity is accessible or not. Use the `TeamCity::Import-Artifact` operation (and reference `$TeamCityBuildNumber` obtained from the `TeamCity::Queue-Build` operation, or reference a [special build name](#special-build-names)) to create a BuildMaster build artifact from an existing TeamCity artifact:

```
TeamCity::Import-Artifact
(
    Credentials: TeamCity,
    Project: $TeamCityProjectName,
    BuildConfiguration: $TeamCityBuildConfiguration,
    Artifact: $TeamCityArtifactName,
    TeamCityBuildNumber => $TeamCityBuildNumber
);
```

For organizations that have their continuous integration process fully configured in TeamCity, leave the build number set to `lastSuccessful` and capture the actual build number using the `$TeamCityBuildNumber` output variable on `TeamCity::Import-Artifact` operation. This will create a visible association between the BuildMaster build and TeamCity build.

Both the import and queue operations support branches in TeamCity. This is generally only useful in conjunction with [special build names](#special-build-names) because specific build numbers are already unique per branch. While leaving the `Branch` property blank will refer to the `<default>` branch for the project, you may also specify one in the plan to limit the results of `lastSuccessful`, and optionally filter branch names with BuildMaster variables, for example: `Branch: feature-$ReleaseNumber`.

## Special Build Names {#special-build-names data-title="Special Build Names"}

BuildMaster can reference TeamCity builds using the following special build names instead of direct build numbers:

{.docs}
 - `lastSuccessful` - refers to the last build that completed successfully
 - `lastPinned` - refers to the last build that has been pinned
 - `lastFinished` - refers to the latest build whether it was successful or not (should rarely be referenced in BuildMaster)

Additionally, builds may be referenced by their underlying TeamCity ID as opposed to name/number. To specify a build ID instead, append `:id` as a suffix to the identifier, for example: `1234:id`

## Pushing Artifacts from TeamCity to an Artifact Repository {#push data-title="Pushing Artifacts from TeamCity"}

To push artifacts from TeamCity to BuildMaster or an artifact repository like [ProGet](https://inedo.com/proget), refer to the [Trigger from External Systems](https://docs.inedo.com/docs/buildmaster/builds/external-systems/trigger-via-api) documentation to see the variety of ways this can be accomplished.

## Deploying TeamCity Artifacts {#deployment data-title="Deploying TeamCity Artifacts"}

Once an artifact is captured via the optional queuing followed by "import" operations in a build plan, future stages simply need to use the `Deploy-Artifact` operation to deploy to any number of servers or targets:

```
Deploy-Artifact profitcalc-web
(
    To: E:\Websites\ProfitCalc
);
```

## Self-Service TeamCity Builds {#self-service data-title="Self-Service Builds"}

With [release templates](/docs/buildmaster/releases/templates), you can build a deployment pipeline that prompts for build numbers and/or build configurations based on data directly from TeamCity. When creating the release template, the following list variable source options are available:

{.docs}
 - TeamCity Project Name
 - TeamCity Build Configuration
 - TeamCity Build Number

Refer to the [Choosing a specific TeamCity Build with BuildMaster](https://inedo.com/support/tutorials/buildmaster/teamcity/choosing-specific-artifact-from-teamcity) tutorial for more information.
