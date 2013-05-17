GitPushy - A Simple Deployment Framework
========

Deployments should be simple. Access controls to a git repository should determin who has access. Deployment should be as simple as commiting to a certain brance of the git repository. Deployment scripts are almost always writen in bash anyway so why not write the entire deployment engine in bash?

GitPushy works as a trigger from git's *post_receive* hook. It uses this to trigger it's own hooks and build, stage and deploy source code to a working application environment.


GitPushy Configuration Variables
--------
Git Pushy comes with a long list of configuration variables to make it easy to setup a deployment.

PUSHY_USER

PUSHY_SERVER

PUSHY_PORT

POSHY_HOOKS


GitPushy Default Hooks
--------
Default hooks are hooks that are run every time GitPushy come to that section of the delployment process

**.gitpushy-config**
**.gitpushy-build**
**.gitpushy-stage**
**.gitpushy-deploy**


GitPushy Build Hooks
--------
**.gitpushy-{hook}-config**
**.gitpushy-{hook}-build**
**.gitpushy-{hook}-stage**
**.gitpushy-{hook}-deploy**


GitPushy Custom Hooks
--------
**.gitpushy-{hook}-custom**

