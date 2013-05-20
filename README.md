GitPushy - A Simple Deployment Framework
========

GitPushy is a siplified framework to aid in scripting [continuous deployment] [cd] and [continuous integration] [ci].
At it's core, GitPushy is simply a Bash framework for [CD] [cd] and/or [CI] [ci] without a lot of needless fluff.

[CD]: http://en.wikipedia.org/wiki/Continuous_deployment "Wikipedia: Continuous Deployment"
[CI]: http://en.wikipedia.org/wiki/Continuous_integration "Wikipedia: Continuous Integration"

Deployments should be simple. Send a git commit, deploy updates to intended servers. Different branches represent different stages of application development.
Git branch access determins deployment access. Commiting to a certain branch triggers deployment on that branch. Deployments happen from multiple servers to
multiple servers. Using git hooks allows us to capitalize on the distributed nature of git to enhance deployment. Manual deployments use bash from the command
line so we might as well write the entire deployment engine in bash to avoid duplicating the effort.


Instalation
--------
GitPushy is triggered from the git *post_receive* hook. It uses this hook to trigger it's own deployment process, stage and deploy source code to a working
application environment. To install GitPushy simply copy the *gitpushy-common* to the git hooks directory and copy the contents of *post-receive* to your git
hooks folder.


Anatomy of GitPushy Deployment
--------

Each GitPushy deployment are devided into four sections: *config*, *build*, *stage*, *deploy*. A fifth *custom* section can be added in place of the *build*,
*stage* and *deploy* sections to empower complex deployment processes. Initially GitPushy triggers the main deployment. You can trigger other deployments by
adding them to the **$PUSHY_TRIGGERS** variable or by calling **gitpushy_trigger** directly.

When GitPushy is triggered everything starts with the main deployment. The following files are run if they exist:

    .gitpushy-config
    .gitpushy-main-config
    .gitpushy-main-custom*
    .gitpushy-build
    .gitpushy-main-build
    .gitpushy-stage
    .gitpushy-main-stage
    .gitpushy-deploy
    .gitpushy-main-deploy

NOTE: If the file .gitpushy-main-custom exists none of the other stages are called unless the custom script calls them specifically.

There are both general and deployment-specific parts to each stage. The general part of the stage is always run first followed by the deployment-specific
level code. If the config section specifies more deployments in the **$PUSHY_TRIGGERS** variable these will be run after the main deploymenthas finished
it's execution.

GitPushy Config Script
--------
The Config stage is intended to set key variables necesary for making a pushy deployment possible. The most simple version of this script consists of
setting the **$PUSHY_DEPLOY_DIR** variable to the place where your final code will reside. Without specyfying this directory nothing will be done to deploy
unless you have a *.gitpushy-{trigger}-custom* script for your GitPushy deployment. There are many other variables that can be set in this section. See
GitPushy Scripting Variables for more info.


GitPushy Build Script
--------
In the build section GitPushy creates a clean clone of a git repository and deposits it in the specified **$PUSHY_BUILD_DIR**. The clone is set to the
branch being deployed to it's latest commits. Then the build script is run to do any of the heavy lifting necessary in the bundling of code.

If this process is rather strenuous and time consuming it may be better to run these detatched from a *.gitpushy-{trigger}-custom* script to avoid
monopolizing the git commit process. See more on GitPushy Custom Script.

In most cases where GitPushy is used to deploy websites this script will be used to move files to their proper location and possibly delete files
that do not belong in a particular deployment. This may also be useful to trigger a database sync from production back to development.

Since the build directory is a valid git clone you can script a push back to the git repository. There are special GitPushy commands to push code back
and avoid double triggering of GitPushy. See Scripting GitPushy for more information on git pushy commands.


GitPushy Stage Script
--------
After GitPushy builds a deployment it copies files through rsync to the local or remote **$PUSHY_STAGE_DIR**. GitPushy then runs the commands in the stage
section before completing deployment. Unlike the config and build stages the stage section happens in a directory which is not a git repository.
Furthermore, it hapens outside of the GitPushy script environment and possibly on a remote server.

The state section is intended for times when you wish to check a deployment before actually deploying it. This allows you to run unit tests or simple
litmus tests to prove the applcation is stable and ready to be deployed. If at any time you cause the stage section to exit with a value other than zero
GitPushy will abort at the end of that section and display the error code to the committer in the git commit scrollback. If you need to force immediate
abortion you can use the normal bash exit command providing a non-zero integer as the exit code.

NOTE: Aborting at the deploy section does not roll back code deployment. (Since the script is not called until after the code is copied.) Therefore, it is
preferable to do all of the heavy testing for success in the stage section to insure sucess when deploy is called.


GitPushy Deploy Script
--------
After stage returns successful GitPushy replaces code in the **$PUSHY_DEPLOY_DIR** and then executes the GitPushy Deploy Script.

This section is intended for any finalization and cleanup that must happen before a deployment can be called complete. This is where a database dump might
be loaded or where daemons might be restarted to accept new configuration files.

In a typical GitPushy Deployment this is concidered the end of a GitPushy script.


GitPushy Custom Script
--------
In some cases of more complex deployments you may wish to have more constrol over when and how deployments happen. In these case, GitPushy will replace
the normal execution of build, stage and deploy with a single section to be scripted by you.

This can be useful when multiple repositories are required or when you wish to change the sequence of events for a deployment. This script executes in the
cloned git repository located at **$PUSHY_BUILD_DIR**.

There are many scenarious that lend itself to custom scripting. Suppose a Makefile needs to be run for a long period, you want to add lock checking to
your script to insure that consecutive commits are handled properly while the a build is in progress. Or perhaps you want to push and pull from other
repositories and attempt some complicated git scripting.


GitPushy Scripting Variables
--------
Git Pushy comes with a short list of configuration variables to make it easy to setup a deployment.

 * **$PUSHY_REPO** - [String] Repository being pushed
 * **$PUSHY_BRANCH** - [String] Specifies the branch of the current commit
 * **$PUSHY_BUILD_DIR** - [String] Specifies the directory where build activity should take place 
 * **$PUSHY_STAGE_DIR** - [String] Specifies the directory where staging activity should take place
 * **$PUSHY_DEPLOY_DIR** - [String] Specifies the directory where deployment activity should take place
 * **$PUSHY_DEPLOYMENT** - [String] Current process being run
 * **$PUSHY_TRIGGERS** - [Array] All triggers to be run
 * **$PUSHY_PORT** - [Number] Remote port to connect in order to stage and deploy files
 * **$PUSHY_REMOTE** - [Boolean] whether or not deployment is to a remote server
 * **$PUSHY_SERVER** - [String] Server hostname to use when staging and deploying
 * **$PUSHY_STATUS** - [Boolean] Return code for current running process
 * **$PUSHY_USER** - [String] Remote user to use to connect
 * **$PUSHY_VERBOSE** - [Boolean] Whether to print verbose debug information to the console


GitPushy Scripting Commands
--------
When writing advanced deployments the following commands can be useful to scult your deployment process. They are only available in the config and
custom sections of deployment. In most cases you will want to avoid using these in the config section and only use them in the
*.gitpushy-{trigger}-custom* script. However, certain commands like **gitpushy-server-is-local** may be useful even in your *.gitpushy-{trigger}-config*
file.

* **gitpushy-trigger {deployment}** - Trigger a new GitPushy deployment. This should only really be used inside of a *.gitpushy-{trigger}-custom* script
since the usual way to call extra deployments is through the **$PUSHY_TRIGGERS** variable. This call will wipe and repopulate the **$PUSHY_BUILD_DIR**.
If you wish to keep the same build directory it is better to use the sectional calls **gitpushy-build {deployment}**, **gitpushy-stage {deployment}** and
**gitpushy-deploy {deployment}**.
* **gitpushy-build {deployment}** - Calls the *.gitpushy-build* and *.gitpushy-{trigger}-build* scripts. This can be used in custom scripting or where a
conditional build process may be necessary. For example:

        [ passed_error_checks ] && gitpushy-build uat

* **gitpushy-stage {deployment}** - Calls the *.gitpushy-stage* and *.gitpushy-{trigger}-stage*. This is really only useful when called from within
custom scripting where it is necessary to change the call order of the scripts:

        gitpushy-stage $PUSHY_DEPLOYMENT && gitpushy-stage unit-testing

* **gitpushy-deploy {deployment}** - Calls the *.gitpushy-deploy* and *.gitpushy-{trigger}-deploy* scripts. This is really only useful when called from
within custom scripting where it is necessary to change the call order of the scripts.

        gitpushy-deploy $PUSHY_DEPLOYMENT && gitpushy-deploy backup

* **gitpushy-push-branch {branch}** - Calls for the GitPushy deployment of the specified branch. This is useful where commits to one branch control
deployments of another. This is used internally to deploy the commited branches like normal. This will trigger a fresh deployment process of the branch.
NOTE: If you call **gitpushy-push-branch $PUSHY_BRANCH** without changing the variable from your current branch you will end up in an endless loop.

* **gitpushy-push-replicate {server} {server ...}** - This is ment to be run in the build stage where each replication is put in its own branch with the
same name as the server who it pushed to and replicated from. Note that replication may trigger gitpushy on the remote server if GitPushy is installed
there too. You can prevent continous replication by testing:

        [ "$(gitpushy-committer)" = "GitPushy" ] && return 0

* **gitpushy-show {file} {file}** - This works like cat inside of gitpushy allowing you to create variables from the contents of files. This is used
internally by GitPushy to check for GitPushy script files. An empty string will be returned if the file does not exist.
* **gitpushy-committer** - Show the last comitter for the current branch.
* **gitpushy-branch-name** - Show the branch name of the current selected branch. This is for cases where multiple branches are used in the
**$PUSHY_BUILD_DIR** to make sure that the right one is selected.
* **gitpushy-server-ip** - Show the resoved IP of a server name
* **gitpushy-local-ips** - Show a list of IPs that reference the current machine. 
* **gitpushy-in-array {string} {array}** - Checks if a certain string is in a specified array.
* **gitpushy-server-is-local {server}** - Checks if a server is local or remote. The following code is similar to the code in
**gitpushy-push-replicate**:

        [ gitpushy-server-is-local $SERVER ] && return 0


GitPushy Branch Hooks
--------
When *.gitpushy-{trigger}-custom* is not desired or when you wish to avoid shipping the custom script with the repsitory for some reason you can put
your custom script in the git hooks directory as a file *gitpushy-{branch}-branch*. This will be called like a custom script but without running the
config files first. This hook is run in the **$PUSHY_BUILD_DIR** in a fresh git clone in place of the traditional _.gitpushy-*_ files. Since this file
is not versioned it should be managed from another repository somewhere else. Also, you should be careful about installing this in *.gitolite/hooks/common*
since it will run based on branch name.
