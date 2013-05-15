GitPushy - The Simple Build Deployment Framework
========

Historically code deployment has been a mess. It is either handled by a Merge Master who inadvertently forgets a step and causes the whole system to go down or it is handed by a Continuous Integration System wich takes a degree in quanotum physics to understand and configure.

FRINKnet thinks deployments should be simple. Access controls to a git repository should determin who has access. Deployment should be as simple as commiting to a certain brance of the git repository. Deployment scripts are almost always writen in bash anyway so why not write the entire deployment engine in this wonderful scripting language?

GitPushy works as a trigger from git's __post_receive__ hook. It uses this to trigger it's own hooks and build, stage and deploy source code to a working application environment.


GitPushy Default Hooks
--------

.gitpushy-config
.gitpushy-build
.gitpushy-stage
.gitpushy-deploy


GitPushy Build System Hooks
--------

.gitpushy-{hook}-config
.gitpushy-{hook}-build
.gitpushy-{hook}-stage
.gitpushy-{hook}-deploy


GitPushy Custom Hooks
--------

.gitpushy-{hook}-custom

