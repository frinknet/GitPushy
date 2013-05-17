GitPushy - A Simple Deployment Framework
========

Deployments should be simple. Access controls to a git repository should determin who has access. Deployment should be as simple as commiting to a certain brance of the git repository. Deployment scripts are almost always writen in bash anyway so why not write the entire deployment engine in bash?


GitPushy Hooks
--------
GitPushy works as a trigger from git's *post_receive* hook. It uses this to trigger it's own hooks and build, stage and deploy source code to a working application environment.

A hook is devided into four section: _config_, _build_, _stage_, _deploy_ - each of these sections are further devided into a general and a specific part of the script. In adition to these four stages a fifth _custom_ stage can be added in place of the _build_, _stage_ and _deploy_ sections to aid in complex deployment proceedures.


Anatomy of a GitPushy Hook
--------
When GitPushy is triggered everything starts with the main hook. The following files are run if they exist:

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

There are both general and hook-specific parts to each stage. The general part of the stage is always run first followed by the mor specific hook level code. If the config section specifies hooks in the _PUSHY_HOOKS_ variable these will be run after the main hook has finished it's execution.

GitPushy Hook Config Section
--------
The Config stage is intended to set key variables necesary for making a pushy deployment possible. The most simple version of this section consists of setting the _PUSHY_DEPLOY_DIR_ variable to the place where your final code will reside. Without specyfying this directory nothing will be done to deploy unless you have a custom section for your GitPushy hook. There are many other variables that can be set in this section. See GitPushy Config Variables for mor info.


GitPushy Hook Build Section
--------
In the build section GitPushy creates a clean clone of a git repository and deposits it in the specified _PUSHY_BUILD_DIR_. The clone is set to the branch being deployed to it's latest commits. Then the build hook is run to do any of the heavy lifting necessary in the bundling of code.

If this process is rather strenuous and time consuming it may be better to run these detatched from a _.gitpushy-hook-custom_ script to avoid monopolizing the git commit process. See more on GitPushy Hooks Custom Section.

In most cases where GitPushy is used to deploy websites this hook will only be used to move files to their proper location and possibly delete files that do not belong in a particular deployment. This may also be useful to trigger a database sync from production back to development.

Since the build directory is a valid git clone you can script a push back to the git repository. There are special GitPushy commands to push code back and avoid double triggering of GitPushy. See Scripting GitPushy for more information on git pushy commands.


GitPushy Hook Stage Section
--------
After GitPushy builds a deployment it copies files through rsync to the local or remote _PUSHY_STAGE_DIR_. GitPushy then runs the commands in the stage section before completing deployment. Unlike the config and build stages the stage section happens in a directory which is not a git repository. Furthermore, it hapens outside of the GitPushy script environment and possibly on a remote server.

The state section is intended for times when you wish to check a deployment before actually deploying it. This allows you to run unit tests or simple litmus tests to prove the applcation is stable and ready to be deployed. If at any time you cause the stage section to exit with a value other than 0 GitPushy will abort and display the error code to the committer in the git commit scrollback.


GitPushy Hook Deploy Section
--------
After stage returns successful GitPushy replaces code in the _PUSHY_DEPLOY_DIR_ and then executes the deploy section of a GitPushy Hook.

This section is intended for any finalization and cleanup that must happen before a deployment can be called complete. This is where a database dump might be loaded or where daemons might be restarted to accept new configuration files.

In a typical GitPushy Deployment this is concidered the end of a GitPushy script.


GitPushy Hook Custom Section
--------
In some cases of more complex deployments you may wish to have more constrol over when and how deployments happen. In these case, GitPushy will replace the normal execution of build, stage and deploy with a single section to be scripted by you.

This can be useful when multiple repositories are required or when you wish to change the sequence of events for a deployment. This script executes in the cloned git repository located at _PUSHY_BUILD_DIR_.

There are many scenarious that lend itself to custom hooks. Suppose a Makefile needs to be run for a long period, you want to add lock checking to your script to insure that consecutive commits are handled properly while the a build is in progress. Or perhaps you want to push and pull from other repositories and attempt some complicated git scripting.


GitPushy Configuration Variables
--------
Git Pushy comes with a short list of configuration variables to make it easy to setup a deployment.

 * PUSHY_REPO - Repository being pushed
 * PUSHY_BRANCH - Specifies the branch of the current commit
 * PUSHY_BUILD_DIR - Specifies the directory where build activity should take place 
 * PUSHY_STAGE_DIR - Specifies the directory where staging activity should take place
 * PUSHY_DEPLOY_DIR - Specifies the directory where deployment activity should take place
 * PUSHY_HOOK - Current hook being run
 * PUSHY_HOOKS - All hooks to be run
 * PUSHY_PORT - Remote port to connect in order to stage and deploy files
 * PUSHY_REMOTE - Boolean whether or not we
 * PUSHY_SERVER - Server hostname to push to
 * PUSHY_STATUS - Return code for current running hook
 * PUSHY_USER - remote user to connect as
 * PUSHY_VERBOSE - Whether to print verbose debug information to the console


GitPushy Scripting
--------

* __gitpushy-hook {hook}__ - Call another GitPushy hook. This should only really be used inside of a gitpushy-hook-custom script since the usual way to call extra hooks is through the _PUSHY_HOOKS_ variable. Calling this will wipe the _PUSHY_BUILD_DIR_ and start again. It is therefore preferable to call a section of the hook rather than the whole hook.
* __gitpushy-build {hook}__ - Calls the build section of a GitPushy hook. This can be used in custom hooks or when a conditional build process may be necessary. For example:

        [ condition ] && gitpushy-build otherhook

* __gitpushy-stage {hook}__ - Calls the stage section of a GitPushy hook. This is really only useful when called from within a custom hook where it is necessary to change the call order of the sections within a custom hook:

        gitpushy-stage $PUSHY_HOOK && gitpushy-stage otherhook

* __gitpushy-deploy {hook}__ - Calls the deploy section of a GitPushy hook. This is really only useful when called from within a custom hook where it is necessary to change the call order of the sections within a custom hook.

        gitpushy-deploy $PUSHY_HOOK && gitpushy-deploy otherhook

* __gitpushy-push-branch {branch}__ - Calls for the GitPushy deployment of the specified branch. This is useful where commits to one branch control deployments of another. This is used internally to deploy the commited branches like normal. This will trigger a fresh deployment process of the branch. If you call gitpushy-push-branch $PUSHY_BRANCH without changing the variable from your current branch you will end up in an endless loop.

* __gitpushy-push-replicate {server} {server ...}__ - This is ment to be run in the build stage where each replication is put in its own branch with the same name as the server who it pushed to and replicated from. Note that replication may trigger gitpushy on the remote server if GitPushy is installed there too. You can prevent continous replication by testing:

        [ "$(gitpushy-committer)" = "GitPushy" ] && return 0

* __gitpushy-show {file} {file}__ - This works like cat inside of gitpushy allowing you to create variables from the contents of files. This is used internally by GitPushy to check for hooks. An empty string will be returned if the file does not exist.
* __gitpushy-committer__ - Show the last comitter for the current branch.
* __gitpushy-branch-name__ - Show the branch name of the current selected branch. This is for cases where multiple branches are used in the $PUSHY_BUILD_DIR to make sure that the right one is selected.
* __gitpushy-server-ip__ - Show the resoved IP of a server name
* __gitpushy-local-ips__ - Show a list of IPs that reference the current machine. 
* __gitpushy-in-array {string} {array}__ - Checks if a certain string is in a specified array.
* __gitpushy-server-is-local {server}__ - Checks if a server is local or remote. The following code is similar to the code in gitpushy-push-replicate:

        [ gitpushy-server-is-local $SERVER ] && return 0



