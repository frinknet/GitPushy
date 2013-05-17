GitPushy - A Simple Deployment Framework
========

Deployments should be simple. Access controls to a git repository should determin who has access. Deployment should be as simple as commiting to a certain brance of the git repository. Deployment scripts are almost always writen in bash anyway so why not write the entire deployment engine in bash?


GitPushy Hooks
--------
GitPushy works as a trigger from git's *post_receive* hook. It uses this to trigger it's own hooks and build, stage and deploy source code to a working application environment.

A hook is devided into four parts: _config_, _build_, _stage_, _deploy_ - each of these sections are further devided into a general and a specific part of the script. In adition to these four stages a fifth _custom_ stage can be added in place of the _build_, _stage_ and _deploy_ sections to aid in complex deployment proceedures.


Anatomy of the Main Hook
--------
When GitPushy is triggered everything starts with the main hook. The following files are run if they exist

    .gitpushy-config
    .gitpushy-main-config
    .gitpushy-main-custom*
    .gitpushy-build
    .gitpushy-main-build
    .gitpushy-stage
    .gitpushy-main-stage
    .gitpushy-deploy
    .gitpushy-main-deploy

As you can see there are both general and specific stages that can be run for each hook.


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

