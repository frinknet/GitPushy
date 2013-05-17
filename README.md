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

If the config file specifies more hooks in the _PUSHY_HOOKS_ variable these will be run after the main hook has finished it's run.

There are both general and hook-specific parts to each stage. The general part of the stage is always run first followed by the mor specific hook level code.

NOTE: If the file .gitpushy-main-custom exists none of the other stages are called unless the custom script calls them specifically.


GitPushy Hook Config Section
--------
The Config stage is intended to set key variables necesary for making a pushy deployment possible. The most simple version of this section consists of setting the _PUSHY_DEPLOY_DIR_ variable to the place where your final code will reside. Without specyfying this directory nothing will be done to deploy unless you have a custom section for your GitPushy hook. There are many other variables that can be set in this section. See GitPushy Config Variables for mor info.


GitPushy Hook Build Section
--------
In the build section GitPushy creates a clean clone of a git repository and deposits it in the specified _PUSHY_BUILD_DIR_. The clone is set to the branch being deployed to it's latest commits. Then the build hook is run to do any of the heavy lifting necessary in the bundling of code. If this process is rather strenuous and time consuming it may be better to run these detatched from a _.gitpushy-hook-custom_ script to avoid monopolizing the git commit process. If a Makefile needs to be run you may also want to add lock checking to your scrpt to insure that consecutive commits are handled properly when a build may be in progress. See more on GitPushy Hooks Custom Section.

In most cases where gitpushy is used to deploy websites this hook will only be used to move files to their proper location and possibly delete files that 


GitPushy Hook Stage Section
--------


GitPushy Hook Deploy Section
--------


GitPushy Hook Custom Section
--------


Each replication is put in its own branch with the same name as the server who it pushed and replicated from. Note that replication may trigger gitpushy on the remote server if GitPushy is installed there too.

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


GitPushy Default Hooks
--------
Default hooks are hooks that are run every time GitPushy come to that section of the delployment process

**.gitpushy-config** This is where you set the common variables used by all your git pushy hooks

**.gitpushy-build** This is where you put all code common to the build portion of your hooks

**.gitpushy-stage** This is where you put all code common to the staging portion of your hooks

**.gitpushy-deploy** This is where you put all code common 


GitPushy Build Hooks
--------
**.gitpushy-{hook}-config**

**.gitpushy-{hook}-build**

**.gitpushy-{hook}-stage**

**.gitpushy-{hook}-deploy**


GitPushy Custom Hooks
--------
**.gitpushy-{hook}-custom**

