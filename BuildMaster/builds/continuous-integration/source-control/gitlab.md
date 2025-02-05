---
title: GitLab
subtitle: Integrating BuildMaster with GitLab
sequence: 200 
keywords: buildmaster, gitlab, git
show-headings-in-nav: true
---

[GitLab](https://about.gitlab.com/) is a DevOps platform that has support for source control hosting, issue tracking, collaboration tools, and cloud-native deployment. While GitLab has support for its own pipelines, BuildMaster is designed to work alongside GitLab to provide a self-managed CI/CD solution, especially when:

{.docs}
 - modeling legacy applications you are already deploying
 - application portfolios span multiple hosting environments
 - internal deployment targets, or a variety of targets throughout an organization
 - adding a layer of release management 

This is accomplished in BuildMaster through the following features:

{.docs}
 - [Source Control](/docs/buildmaster/builds/continuous-integration/source-control)
 - [Continuous Integration](/docs/buildmaster/builds/continuous-integration)
 - [Issue Tracking](/docs/buildmaster/verification/issue-tracking)
	
::: {.attention .best-practice}
**See it live!** See an example of GitLab integration by creating a new application using the *GitLab CI/CD* template.
:::

## Installing the GitLab Extension {#extension data-title="GitLab Extension"}

Simply navigate to the *Admin* > *Extensions* page in your instance of BuildMaster and click on the GitLab extension to install it.

If your instance doesn't have internet access, you can [manually install the GitLab extension](https://docs.inedo.com/docs/buildmaster/reference/extensions#manual-install) after downloading the [GitLab Extension Package](https://proget.inedo.com/feeds/Extensions/inedox/GitLab).

## Authentication with GitLab {#authentication data-title="Authentication with GitLab"}

Authentication with GitLab is handled through [Resource Credentials](/docs/buildmaster/administration/resource-credentials). In most cases, creating a resource credential using the GitLab type is sufficient, but you can also use Git for "generic" source control integration that doesn't rely on GitLab-specific concepts like "organizations". These credentials are effectively a username and password, so we recommend creating a specific account with the minimum amount of privileges required to interact with GitLab, typically this is just *Guest* permission to pull source code, but if there are requirements to tag source code for example, at least the *Developer* permission will be required. Refer to the [GitLab Permissions documentation](https://gitlab.com/help/user/permissions) for more information.

To connect to a self-managed instance of GitLab Enterprise, make sure the API URL of the credentials is configured to the API URL (`https://host/api`) for your GitLab Enterprise installation.

::: {.attention .technical}
SSH connections are not supported using the built-in GitLab integration in BuildMaster, so make sure to use the HTTPS endpoint when supplying the repository URL to the resource credentials or any operations. If your organization requires SSH to connect, you must install and configure the Git CLI manually and then instruct BuildMaster to use the [Git CLI instead](#cli).
:::

## Source Control Integration {#source-control data-title="Source Control"}

BuildMaster's primary integration point with GitLab is source control, including pulling source code, tagging it, and capturing commit IDs to maintain association between build artifacts and specific commits.

### Working with GitLab Source Code

#### Getting source code

To get source code from a repository, use the `GitLab::Get-Source` operation. This operation supports various optional levels of granularity including the branch name to pull from, the tag name, or even specific Git references such as the commit ID.

#### Tagging source code

 To tag source code in a repository, use the `GitLab::Tag-Source`, which will assign a friendly name (such as `$ReleaseNumber.$BuildNumber`) to a specific commit. This is recommended during later stages of a pipeline to indicate which specific commits have actually been released, as opposed to ones leading up to the eventual release.
 
#### Prompting for branch name before a build

To prompt for user input when creating a build, configure a [Release Template](/docs/buildmaster/releases/templates) for your application. The most common use-case for this is selecting a branch when building. This can be accomplished using a "GitLab Branches" dynamic list variable source as the type of build variable.

#### Built-in Git Client vs. `git` CLI {#cli data-title="Built-in Git Client vs. Git CLI"}

By default, BuildMaster does not require Git to be installed on a build server in order to perform source control operations. However, some users who require more advanced configuration for their Git integration may instruct BuildMaster to run the CLI instead by:

{.docs}
 - setting the `GitExePath` operation property to a valid Git executable path, e.g. `/usr/bin/git` on Linux or `C:\Program Files\Git\bin\git.exe` on Windows
 - configuring a `$DefaultGitExePath` variable at the server or system level in BuildMaster; this will force all Git source control operations to use the CLI instead of the built-in library

## Continuous Integration {#ci data-title="Continuous Integration"}

### Automated Builds

Automated builds are the process of creating builds at certain intervals or in response to commits and pushes to a repository. For GitLab, they can be achieved in multiple ways:

{.docs}
 - scheduled to create a build at a specified frequency (hourly, daily, etc.)
 - create a build as a result of a webhook monitor (see https://github.com/Inedo/inedox-git/wiki/configuring-gitlab-hooks for how to configure this)
 - poll for commits and create a new build using a [respository monitor](/docs/buildmaster/builds/continuous-integration/build-triggers-and-monitors/repository-monitors), useful if BuildMaster is firewalled off from the internet

### Capturing commit IDs

Git repositories use a a 20-byte SHA1 hashcode that represents the current state of the repository and identifies the commit. Capturing this reference at build time is very important so you can see exactly what code was used to build your application. 

In the `GitLab::Get-Source` operation, use the output property to capture into a variable (`CommitHash => $commitHash`), then use `Set-BuildVariable` to store it as `$CommitId`. Once this value is captured, you can use [variable value renderers](/docs/buildmaster/administration/variable-renderers) to link back to GitLab wherever the commit ID, most commonly on the overview page for a specific build.

### Displaying a CI badge for a GitLab project

GitLab supports CI badges per project. Before they can be used, the [CI Badge API](/docs/buildmaster/reference/api/ci-badge) must be enabled. In GitLab, visit the *Settings* > *General* tab, and find the Badges section. Adding the following link and image URLs will render a badge on the project overview page:

{.docs}
 - **Link** - `https://{buildmaster-server}/api/ci-badges/link?key=<badge-key>&$ApplicationName=%{project_path}`
- **Badge image URL** - `https://{buildmaster-server}/api/ci-badges/image?key=<badge-key>&$ApplicationName=%{project_path}`

Refer to the [CI Badges](/docs/buildmaster/builds/continuous-integration/badges) documentation for examples with further customization.

## Issue Tracking {#issues data-title="Issue Tracking"}

### Integrating with BuildMaster's Issue Tracking

A combination of Issue Sources and resource credentials are used to connect BuildMaster to GitLab's issue tracker. To associate issues, create a GitLab [Issue Source](/docs/buildmaster/verification/issue-tracking), which will link issues to releases in BuildMaster using the GitLab milestone.

### Creating milestones

A milestone is the mechanism used to associate release numbers in BuildMaster with a specific GitLab issue and is intended to be a version number of sorts; it does not need to match the version used by marketing, but needs to be uniquely-identifiable so that BuildMaster can use it to apply gating logic in pipeline stages (i.e. all issues closed for *this* release). You can use the `GitLab::Ensure-Milestone` operation at an early stage to ensure that a milestone exists, or could be created at the end of a previous release.

### Closing milestones

Closing a milestone in GitLab means the version represented by that milestone was "released" so no more issues should be associated with it. To close a milestone at the end of a pipeline, use the `GitLab::Ensure-Milestone` operation with the property: `State: closed`

### Creating and updating issues

To create issues for a particular GitLab milestone, use the `GitLab::Create-Issue` operation. To add a comment to an existing issue, use the `GitLab::Create-IssueComment` operation.
