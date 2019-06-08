---
layout: post
title: Moving my CI/CD builds from Appveyor to DevOps
category: post
---
I’ve been using Appveyor for my CI builds for quite a while, but with Azure DevOps offering a really generous free tier, I figured it was worth checking out how the big guys operate.

If you haven’t already, setting up a DevOps account (free!) is as easy as hitting up https://azure.microsoft.com/en-us/services/devops/ and connecting with your GitHub account (side note: remember the concern when MS bought GH? So far they’re only doing amazing things).

The free tier includes 1,800 monthly minutes for CI/CD work on a DevOps-hosted pipeline, and unlimited minutes for self-hosted. That’s plenty generous for the amount of CI I’m planning on doing – a single build currently runs around 4 minutes, increasing as I add more tests – so even a 10-minute build would still mean 180 a month, which is a lot.

While there’s a heap of DevOps features in addition to, and included in, the free tier, the scope of this article is to look at what I was doing on Appveyor, and how I shifted that to DevOps with a few super useful additions.

I’ve used Appveyor (and still do) for several years to manage CI builds for my Umbraco projects. With reasonably simple configuration, Appveyor will grab my latest commit, build the solution, test it (assuming I’ve provided a test suite) and compile the Umbraco package for the package repository and Nuget. 

Finally, it pushes a tagged release back to GitHub. Automation is amazing, and a huge timesaver.

However, when I started work on porting Plumber from Umbraco 7 to 8, I decided I’d introduce two paid options, in addition to a basic free version. To make that work, for the short term at least, the source code needed to be private. 

Appveyor’s free tier doesn’t permit private repositories, but DevOps does. Hence partial motivation for the shift.

## CI/CD with Appveyor

First, let’s look at the lay of the Appveyor land – how my previous builds worked, and which parts of the process were platform-specific and which were regular old MSBuild tasks.

Appveyor’s builds can be fully configured online via the GUI, or by adding a YAML file to the repository (same goes for DevOps). I use the latter, and it looks something like this:

/// yaml

Is it perfect? Hell no. Does it work? Hell yes. This could likely be improved a heap, and can definitely be expanded on, but for what I need to do, it’s fine.

YAML is reasonably self-documenting, which is nice. For Appveyor, my builds run like this:

-	Declare the build number, which matches the assembly version of my compiled code
-	Declare the machine image I want to build on
-	Set up some cache locations for Nuget and NPM stuff
-	Declare some stuff to run on install (more on this soon)
-	I only want to build on commits to the master and variants branches
-	Don’t test. Naughty
-	Let Appveyor know where to find my build artifacts for publishing
-	Deploy to Nuget and Github, assuming the commit was tagged and the branch is master

In the `install` step, running `build.bat` kicks off the entire Umbraco build – this is a build script I’ve been using for years after [Jeavon’s super-helpful 24days.in post way back in 2014](https://24days.in/umbraco-cms/2014/packaging-with-appveyor/). 

When something ain’t broke, don’t fix it. It’s a really solid process and manages all the steps for creating a distributable Umbraco package.

Taking a step back, Appveyor isn’t really doing the heavy lifting, it’s kicking off a process in which all the work takes place. 

With that in mind, let’s look at how to replicate the same build on DevOps.

## CI/CD with DevOps

After creating an account, you’ll need a project to work from. On the DevOps dashboard, over on the right, click the Create Project button. Give it a name, maybe a description, and click Create. Congratulations, you’re using DevOps.

Once the new project is created, head on over to the Pipelines section, and choose New Pipeline. This is where the fun starts. 

DevOps will helpfully offer some templates as starting points, based on where your code lives. There’s no need to host your repository in DevOps, but that’s absolutely an option.

Choose your repository, accept the suggest pipeline YAML and hit Run. This will trigger a CI build, which may or may not complete, based on what actually needs to happen to build your solution. That’s fine, we’re about the customise the guts out of it anyway.

While the build config is stored as a YAML file in the root of your repository, it can be modified from the DevOps GUI, simply by selecting tasks from an extensive list and updating a few fields. Magic.

So how does that look compared to my Appveyor build? It’s more complex, no doubt, but it’s also doing a lot more. Because I still wanted to leverage the awesomeness that came with `build.bat`, some of steps are a bit messy and can probably be improved. Time for that later.

/// yaml

The first three sections are pretty clear – the build is triggered only by pushes to master or tagged commits, will run on the latest Windows VM, and requires a set of variables (`buildVersion` here serves the same purpose as `build` in the Appveyor YAML).

The `steps` section is where we start the interesting bits.

First, we set some environment variables using a bash task. This could be done with a file reference, but I’m doing it inline. 

These variables are then available through the build, while the ones declared earlier are only available within the build configuration (I think!).

Next, we install all the NPM dependencies required to build my package. This uses a built-in NPM task, which makes for simpler configuration. Could do the same with a bash task, but it would be wordier.

Once the NPM job is done, we run the default Grunt task to build the client-side part of the package. Grunt is a bit old-school, but thanks to [Tom Fulton’s grunt-umbraco-package module](https://www.npmjs.com/package/grunt-umbraco-package), Grunt can automate all the package creation steps we’d usually be doing in the Backoffice. 

Happy. Days. 

I don’t include the compiled DLLs in this step, because they don’t exist yet.

To generate the DLLs, we need to build the Umbraco solution. That happens in the next three tasks, which install the required Nuget tooling, pulls the Nuget dependencies, then builds the solution.

The solution build task uses a built-in MSBuild job. All it needs is the location of the solution or project file to build. 

Remember the `build.bat` steps from the Appveyor build? We don’t need it anymore – the steps we’ve completed up until here essentially recreate what was happening in `build.bat`. We could have chosen to run it as part of the pipeline, but I preferred breaking it out into individual tasks since that adds visibility into the build in case of errors.

Instead of `build.bat` we now build the BuildPackage project. This is the same project file as I used in the Appveyor build, with a few modifications to make it work on the DevOps platform. 

The build sets up some more variables, extracts the version from the provided build variable, updates assembly versions, builds Umbraco, builds Plumber (Core, Web, Testing libraries), copies those to the same location as the earlier Grunt task put the client-side assets, then updates the package.xml files for the Umbraco package and the Nuget version, builds the packages and outputs them to the artifacts directory.

It’s a thing of beauty and it Just Works TM.

Next up, testing the build.

While DevOps provides a built-in testing task using Visual Studio Test Runner, this doesn’t quite meet my need since it doesn’t output a coverage report. It automatically generates a pretty test results dashboard, but not coverage. Instead, I use a Powershell task to execute the tests and create reports using Coverlet and ReportGenerator.

The YAML config for testing looks simple, it runs a PS file at the given location, but there’s a bit going on in that file.

<script src="https://gist.github.com/nathanwoulfe/b44dfe0654b5fb3cc4db3d06279247f4.js"></script>

The PS magic does the following:

-	Installs ReportGenerator and Coverlet.
-	Creates an output directory
-	Finds the compiled testing library and the Coverlet executable
-	Runs Coverlet with arguments set to output in Cobertura format, and only generate coverage for Plumber.Core and Plumber.Web (otherwise it covers all assemblies in the solution)
-	Tells ReportGenerator where to find the coverage output, then outputs the HTML reports to the output directory created earlier

It would be possible to run the entire CI build from a PS script. It would be a large, complex PS script, but it would work.

It would though lose the visibility into the build that comes with using individual tasks. I can see build bottlenecks at a glance, can see failed steps at a glance, and if I need to explore the output from a given task, it’s a lot more manageable than having everything in a single log. 

It’s also considerably more pleasant to build out tasks from the GUI than in a script. To each their own, do what works for you.

Once the test suite has run, the next two tasks publish the results and coverage report for DevOps to use in the dashboards. Easy config for these two, just providing file locations.

Almost done.

The final two steps copy the build artifacts (being the Plumber package for the Umbraco repository and Nuget) from the BuildPackage/artifacts directory to DevOps’ artifacts staging directory, then from there publishes the same files as pipeline artifacts so they can be used in a release.

Fin. Done. The end.

It’s definitely more verbose than the Appveyor equivalent, but that’s largely due to task previously completed by running `build.bat` now existing as explicit steps in the build. Like I said earlier, I prefer the additional visibility.

There’s a whole additional stage for CD and creating and publishing releases, which is a tale for another day.




