---
title: Source Control Branches
sequence: 10
show-headings-in-nav: true
---

Branches are one of the most powerful features that most source control system offer. Although they may take a little getting used, they ultimately can save a lot of development resources and allow teams to deliver better software releases faster because:

- **Releases can be developed in parallel.** Even though two-week sprints are becoming the norm, it doesn't mean that everyone needs to be sprinting all the time and focusing only on the "next" release. Release branches allow teams to think about and work on releases that are weeks or months away without stressing about the present
- **Features can be independently developed and tested**. By using "feature branches" to develop and test changes to an application, developers can be certain that code changes from *other* features won't interfere with their work, and that their work won't interfere others
- **Experiment with innovative changes.** Feature branches also enable business stakeholders to sponsor proofs-of-concept without worrying about experimental code being released into production

Despite these benefits, many development teams avoid using branches to their fullest because of the complexity of setting up and of maintaining automated builds and deployments (CI/CD). Some of the commonly cited concerns include:

- **Continuous Integration servers are not designed with branching in mind**. Tools like Jenkins and TeamCity have only recently introduced branching support as a feature, and many have reported that it feels like an afterthought. It's complex to use, and you often have to design a process around what the tool supports
- **Deploying "feature branches" is very risky**. Introducing builds that were created from "feature branches" into your software delivery pipeline means that it could not only interrupt testing for planned releases but also that it might make its way to production
- **Testing "feature branches" is very difficult.** When different teams are working on dozens of different features, it's nearly impossible to keep track of which build was deployed to which environment, and by the time testing teams find that out, a different build is often deployed on top of it
- **Managing "parallel releases" is complicated.** Software delivery pipelines are designed for a repeatable process, but many tools are not flexible enough to deal with hotfixes or parallel releases

BuildMaster simplifies this complexity and makes it safe to introduce branches, thus easily enabling the true benefits of branching.

## Checking Out from Branches {#checking-out-branches data-title="Checking Out from Branches"}

Because most source control interactions are handled within [OtterScript](/docs/otter/reference/otter-script), getting / checking out code from a branch is just a matter of specifying the branch you want to use in a parameter. In some source control systems (like Git), the branch is a parameter of its own, while in others (like Subversion) the branch name is part of the source path.

### Basic  Example: Getting Source from a Branch in Git 

    GitHub::Get-Source
    (
	    Credential: KramericaGitHub,
	    Branch: dev
    );
   
   ### Using Variables to Specify Branches

One of the greatest strengths of OtterScript is variables: Instead of putting the branch name (e.g., `dev`) directly on the operation, you can specify a variable (e.g., `$BranchName`) instead. You can then have the `$BranchName` variable set from any number of places, including:

- Selected by the user when creating a build
- Set from the [Build API](/docs/buildmaster/reference/api/release-and-build) when programmatically creating a build
- Configured on the release-level so that different releases use different branches
- Programmatically determined at build-time in OtterScript using custom logic
- Preventing builds from feature branch from being deployed

This flexibility introduces many use-cases that can be configured to help save developers time and ultimately increase how quickly you can create and release software.

### Prompt for Branch Name at Build Time

Even if you [automatically create a new build](/docs/buildmaster/builds/continuous-integration/build-triggers-and-monitors) each time code is checked-in, it's still important to have a manual process to create new builds. In either case, if you're often building from multiple branches, you can configure the branch name as a user prompt (with a drop-down of branches) that will appear when you create a new build an application.

Variable prompts are configured in an application's [Release Templates](/docs/buildmaster/releases/templates). If you're not familiar with the feature, that's okay -- all applications have a "Default" release template that's used behind-the-scenes to determine which pipeline to use and which variables to prompt for at release, build, and deploy time.

After you configure a branch name variable prompt (e.g. `$BranchName`), BuildMaster will create a build-scoped variable based on whatever the user selects. You can use that variable in your OtterScript plans to get or checkout code from that branch.

### Example: Creating a Drop-down for GitHub Branches

The following steps will configure a `$BranchName` variable prompt when new builds are created that defaults to `master`.

- Navigate to the Default release template (Releases > Release Template > "Default")
- Add a Build Template Variable
	-	Type: Select "Dynamic List"
		- This will open a window where you can select the type of list
		-	Select "GitHub Branches" as the type
		-	Select the Credential you've already configured at the application
	- Name: BranchName
	- Initial value: master
	- Options: value is required

## Selecting Branch when Triggering Builds API {#api-branch-selection data-title="Branch Selection & Builds API "}
You can programmatically create new builds (i.e. trigger builds) by using the [Release & Build Deployment API](/docs/buildmaster/reference/api/release-and-build). When you want to specify the branch name to build, simply that in as a variable (e.g. `$BranchName`) when making the API request.

That will automatically create a build-scoped variable, which can then be used in your OtterScript plans to get or checkout code from that branch.

### Example: Triggering a Build against Branch Using PowerShell 

    Invoke-WebRequest http://{buildmaster-server}/api/releases/builds/create `
    
	    -ContentType "application/json" `
	    -Method POST `
	    -Body '{"applicationName": "hdars", "releaseNumber": "2.14.0", "$BranchName": "dev" }' `
	    -Headers @{"X-ApiKey" = "<api-key>"}

## Branching by Release...Sometimes {#branching-by-release data-title="Branching by Release"}

One common branching strategy is to create a branch that's unique to each release. This not only enables work on multiple releases, but it prevents changes in future releases from impacting the current release.

This pattern is very easy to implement in BuildMaster:

- Create a release in BuildMaster (e.g. 3.4)
- Create a branch in your source control with the same release number (e.g. 3.4)
- [Get or checkout](#checking-out-branches) from the `$ReleaseNumber` branch

However, a lot of teams will prefer to develop against the mainline for the _current_ release (i.e., the in-flight release schedule next), use branches for _future_ releases (i.e., ones that will become _current_ later), and eventually merge those future release branches into the mainline when it will become the _current_ release.

This workflow is also trivial to implement in BuildMaster:

- Create a pipeline named *Current* that defines a pipeline stage variable (`$BranchName`) with "trunk" or "master" as the value
- Create a pipeline named _Future_ that sets $BranchName = $ReleaseNumber instead; do not add a stage for production to this pipeline
- In the build plan, set the branch parameter to `$Eval($BranchName)`

Of course, if your team prefers to make a release branch for the _current_ release while reserving mainline for _future_ releases, you can follow the exact same strategy of using two pipelines. This also has the benefit of clearly labeling the state of a release in the UI.

## Working with Hotfix Branches {#hotfix-branches data-title="Working with Hotfix Branches"}

Sometimes, a critical production bug will necessitate an unscheduled release. Often called a "hotfix," this release will usually have minimal code changes (to minimize change risk), deploy to fewer environments (to minimize interruption of testing other releases), and have different approval requirements.

Implementing this in BuildMaster is straightforward: Just create a _Hotfix_ or _Emergency_ pipeline that models the shortened process and follows the [Branching by Releases](#branching-by-release) strategy so that you can build from a release branch.

When a hotfix is needed:

- Create a release in BuildMaster using the hotfix pipeline
- Create a branch in source control with that release number from the tag or label you used (see [Tagging / Labeling Source Code](/docs/buildmaster/builds/continuous-integration/source-control#tagging))

## Programmatically Determined at Build-time in OtterScript Using Custom Logic {#programmatic-branches data-title="Custom/Programmatic Branching"}

Sometimes, you'll want to use a combination of the techniques described above, or even implement custom logic to determine which branch to build from. Because getting and checking out from source control is handled within OtterScript, it's trivial to implement this, or any other advanced logic.

For example, you could use the following OtterScript in conjunction with any of the above techniques:

    #Set Branch Name for v9 Releases
    #We are still suporting v9 for some customers, and it uses our old conventions
    if  ($Compare($ReleaseNumberPart(major, $ReleaseNumber), >= ,9))
    {
	    if ($PipelineName == Hotfix)
	    {
		    set $BranchName = v9-hotfixes/$ReleaseNumber;
	    }
	    else
	    {
		    set $BranchName = v9;
	    }  
	    Set-BuildVariable BranchName  
	    (  
	    Value: $BranchName  
	    );  
    }

The last statement (`Set-BuildVariable`) is necessary if you want the new `$BranchName` value to be persisted on the build.

## Preventing Builds from Feature Branches from Being Deployed {#blocking-feature-branches data-title="Blocking Feature Branches"}

One of the biggest risks with introducing feature branches into your deployment pipeline is that a build might get accidentally deployed. You don't have to worry about this with BuildMaster.

The simplest solution is to restrict deployments to a single branch:

- Use a `$BranchName` variable in any of the ways described above
- Create a Promotion Requirement that `$Branchname = master`

Of course, you can override this by forcing a promotion to Production. But you may have different or more complex rules, which can just as easily be implemented in OtterScript.

For example, if want to restrict promotion to `master` or `hotfix-` branches, you can use an OtterScript expression instead: 

    $ListIndexOf(@(master,hotfix-$ReleaseNumber), $BranchName) != -1

You could take this one step further, and define the permitted branches at the application or system level, like this:

    $ListIndexOf(@AllowedBranches, $BranchName) != -1