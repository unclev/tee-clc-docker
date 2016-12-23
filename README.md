Docker container with Cross-platform Command-line Client for Team Foundation Server (Team Explorer Everywhere)
===
# Licensing
* The [Dockerfile](Dockerfile) is issued by the [MIT license](LICENSE.txt).
* The image contains (at build time it downloads) [Microsoft/team-explorer-everywhere release 14.111.1](https://github.com/Microsoft/team-explorer-everywhere/releases/tag/v14.111.1)  
  __Team Explorer Everywhere Plug-in for Eclipse__ from [Microsoft/team-explorer-everywhere](Microsoft/team-explorer-everywhere) is issued by the [MIT license](https://github.com/Microsoft/team-explorer-everywhere/blob/master/LICENSE.txt).
* For using __tf__ - *Cross-platform Command-line Client for Team Foundation Server* you MUST accept [MICROSOFT SOFTWARE LICENSE TERMS - MICROSOFT TEAM EXPLORER EVERYWHERE license agreement](https://github.com/Microsoft/team-explorer-everywhere/blob/master/source/com.microsoft.tfs.client.clc/license.html).  
  The image contais a tweak to have this EULA accepted. If you do not accept [MICROSOFT TEAM EXPLORER EVERYWHERE license agreement](https://github.com/Microsoft/team-explorer-everywhere/blob/master/source/com.microsoft.tfs.client.clc/license.html) then you MUST NOT use this image containers.

# Using __tf__
Usage of the Cross-platform Command-line Client for TFS is described on the Internet, for eg.:
* at [msdn.microsoft.com](https://msdn.microsoft.com/en-us/library/hh873092(v=vs.120).aspx); [commands](https://msdn.microsoft.com/en-us/library/z51z7zy0(v=vs.100).aspx)
* at [blogs.msdn.microsoft.com](https://blogs.msdn.microsoft.com/tfssetup/2014/09/23/install-the-cross-platform-command-line-client-for-team-foundation-server/) - Install the Cross-platform Command-line Client for Team Foundation Server
* [Team Foundation Version Control and the Linux Command Line](http://holymonkey.com/team-foundation-version-control-client-in-linux.html)

There are quite a few differences when using the container.
Important notes:
* If your collection (or URL itself) contains spaces, - enclose the URL in doble quotes. Do not use URL encoding like %20. The URL is encoded in the application after accepting parameters.
* If your password contains `!`, escape it with `\` . For eg.: `My\!password`

## TEE License Agreement

Please read and agree with [TEE License Agreement](https://github.com/Microsoft/team-explorer-everywhere/blob/master/source/com.microsoft.tfs.client.clc/license.html) (EULA) before using this container.

The image contains the tweak to have this EULA accepted. Else you would get the following output from the console client:
```
victor@unclev:/srv/storage/Programming/tfs$ tf workspace -new VICTOR -collection:https://mytfshost.net/tfs/My%20Collection
Error: You must accept the End User License Agreement for this product
Run 'tf eula' to accept the End User License Agreement.
```
The image (or container) with EULA accepted returns:
```
victor@unclev:~$ tf eula
You have already accepted the terms of this End User License Agreement.
```

## Volumes
* __/home/tf/projects__  
  This folder is added specifically as a placeholder of the __workfold__ in thems of the TF client.
  Any folder mapped to it in the *tee-clc* environment is considered as the /home/tf/projects work folder. That is *tee-clc-docker* is designed to work with the only work folder.  
  Leaving this directory not mapped does not make any sense.
* __/home/tf/.microsoft/Team Foundation/4.0/Cache__
  The folder where *tee-clc* stores relevant data, - so called *Cache*. For eg., __workspace__.
  You MUST map this directory to a persistent location for the __tf__ client working correctly.
* __/home/tf/.microsoft/Team Foundation/4.0/Configuration__
  The folder where *tee-clc* stores data.  
  You may want having the __Configuration__ folder mapped to persistent storage (for storing settings and data between __tf__ executions).
* __/home/tf/.microsoft/Team Foundation/4.0/Logs__
  The folder where *tee-clc* stores execution logs.  
  You may want having the __Logs__ folder mapped to persistent storage for debugging, inspecting *tee-clc* executions.

## Bash alias
Lets assume the container is started and *not* re-used. The minimum required command would be as follows:
```bash
alias tf='docker run --rm -v "$(pwd):/home/tf/projects"  -v "/srv/microsoft/tf/cache:/home/tf/.microsoft/Team Foundation/4.0/Cache" unclev/tf tf'
```

### Hostname
It is not strictly necessary, but it is highly recommended that you spsefify the same hostname of the container on each start.
Normal set up includes specifying workspace on the tf client host. The TF saves workspaces against the tf client hostname.

### Specifying credentials
The above approaches often cause the error like follows
```
victor@unclev:/srv/storage/Programming/tfs$ tf workspace -new VICTOR -collection:"https://mytfshost.net/tfs/My Collection"
Default credentials are unavailable because no Kerberos ticket or other authentication token is available.
Username: A client error occurred: Authentication credentials were not explicitly provided and could not be determined from any workspace, or argument paths provided.  Use the workspace or login option (if the noprompt option is specified, the login option value must contain the username, domain, and password).

```
Usually the Bash alias has to be modified so that it define credentials. With hostnames specified it would be something like:
```Bash
alias tf='docker run --rm --hostname LINUX -v "$(pwd):/home/tf/projects" -v "/srv/microsoft/tf/cache:/home/tf/.microsoft/Team Foundation/4.0/Cache" unclev/tf tf -login:MYDOMAIN\\myusername,my\!pass -collection:"https://mytfshost.net/tfs/My Collection"'
```
The above you work with the only collection "`https://mytfshost.net/tfs/My Collection`". If this is not suitable to you, exclude the -collection option form the alias.

## Commands
* Available commands and their options:
```bash
tf | more
```

## Examples
With the alias:
```bash
alias tf='docker run --rm --hostname LINUX -v "$(pwd):/home/tf/projects"  -v "/srv/microsoft/tf/cache:/home/tf/.microsoft/Team Foundation/4.0/Cache" unclev/tf tf -login:MYDOMAIN\\myusername,my\!pass -collection:"https://mytfshost.net/tfs/My Collection"'
```
List all directories in "MY.Project":

```
$ tf dir $/MY.Project
$/MY.Project:
$BuildProcessTemplates
$PRJ40DEV
$CURRENT
$FUTURE
$N02Fix
current_change_log.txt
future_change_log.txt

7 item(s).
```
Create a new workspace:
```
victor@unclev:/srv/storage/Programming/tfs$ tf workspace -new VICTOR
Workspace 'VICTOR' created.
```
List cashed workspaces:
```
victor@unclev:/srv/storage/Programming/tfs$ tf workspaces
Collection: https://mytfshost.net/tfs/my collection/
Workspace Owner               Computer Comment
--------- ------------------- -------- ----------------------------------------
VICTOR    Kulitchenko, Victor LINUX    
```

Find specific version(s) by label
```
victor@unclev:/srv/storage/Programming/tfs$ tf labels -format:brief | grep v1.20
v1.20.30                           Kulitchenko, Victor Jan 30, 2013 4:51:23 AM
```
Create a Mapping Between a Local Directory and Source Control (Note: this maps the current folder `v1.20.30` to its internal representation `/home/tf/projects`)
```
victor@unclev:/srv/storage/Programming/tfs$ mkdir v1.20.30
victor@unclev:/srv/storage/Programming/tfs$ cd v1.20.30
victor@unclev:/srv/storage/Programming/tfs/v1.20.30$ tf workfold -map $/MY.Project /home/tf/projects -workspace:VICTOR
```
Get the specified version (by label)
```Bash
victor@unclev:/srv/storage/Programming/tfs/v1.20.30$ tf get -version:Lv1.20.30
/home/tf/
Getting Fix/fixtimeinsf.prg
[...]
Getting wincrypt.h
victor@unclev:/srv/storage/Programming/tfs/v1.20.30$ ls -l
drwxr-xr-x 2 victor victor    4096 янв  2 13:37 Fix
[...]
-rw-r--r-- 1 victor victor   10667 янв  2 13:38 wincrypt.h
```
Get the latest version
```
victor@unclev:/srv/storage/Programming/tfs/v1.20.30$ mkdir ../latest && cd ../latest
victor@unclev:/srv/storage/Programming/tfs/latest$ tf get -version:T
[...]
Deleting /home/tf/projects/Fix
ls -l
drwxr-xr-x  2 victor victor 4096 янв  2 13:55 BuildProcessTemplates
drwxr-xr-x  3 victor victor 4096 янв  2 13:55 prj40dev
drwxr-xr-x 13 victor victor 4096 янв  2 13:56 PRJ40DEV
drwxr-xr-x  4 victor victor 4096 янв  2 13:56 current
drwxr-xr-x 12 victor victor 4096 янв  2 13:57 CURRENT
-rw-r--r--  1 victor victor 4056 янв  2 13:57 current_change_log.txt
drwxr-xr-x  5 victor victor 4096 янв  2 13:57 future
drwxr-xr-x 12 victor victor 4096 янв  2 13:58 FUTURE
-rw-r--r--  1 victor victor  612 янв  2 13:58 future_change_log.txt
drwxr-xr-x  4 victor victor 4096 янв  2 13:58 N02ReclassFix
```
__NB! It gets directories (eg. PRJ40DEV and prj40dev) as they appear in the item paths. As the linux filesystem is case sensitive, it stores them twice even though the Windows version treats them as the same directory__.

Create another project mapping.
```
victor@unclev:/srv/storage/Programming/tfs/latest$ mkdir -p ../DEV.Shared/SomeCoolLib && cd ../DEV.Shared/SomeCoolLib
victor@unclev:/srv/storage/Programming/tfs/DEV.Shared/SomeCoolLib$ DEV.Shared/SomeCoolLib$ tf workfold -map $/DEV.Shared/SomeCoolLib /home/tf/projects -workspace:VICTOR
victor@unclev:/srv/storage/Programming/tfs/DEV.Shared/SomeCoolLib$ tf workfold
===============================================================================
Workspace:  VICTOR
Collection: https://mytfshost.net/tfs/my collection/
 $/DEV.Shared/SomeCoolLib: /home/tf/projects
```
From the __tf__ point this is the folder of the previous project, doing __get__ it deletes the previously mapped project files from the mapped folder (even though the actual folder is another folder and the previos project has never existed there). This may case unexpected behaviour.

### Persistent projects (working with several projects at a time)
If you work with several projects in a parallel then consider mapping your projects folder to `/home/tf/projects`:
```bash
alias tf='docker run --rm --hostname LINUX -v "/full_path_to_your_projects_workdir:/home/tf/projects"  -v "/srv/microsoft/tf/cache:/home/tf/.microsoft/Team Foundation/4.0/Cache" unclev/tf tf -login:MYDOMAIN\\myusername,my\!pass -collection:"https://mytfshost.net/tfs/My Collection"'
```
