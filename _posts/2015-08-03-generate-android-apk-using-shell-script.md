---
layout:     post
title:      Generate Android apk using shell script.
date:       2015-08-03 20:38:19
author:     Franklin
summary:    An article about how to generate apk using shell script.
categories: android shell
#thumbnail:  heart
tags:
 - Shell script
 - Android
 - git
---

Recently i was working on an android application. This project includes two [modules (libraries)][1]. Lets name them as module A and module B. I was working module A and my colleague was working on module B. We use the application to test both modules. We use [git][2] and [submodules][3] to maintain the project. Module A is a submodule of project and Module B is another submodule of the project.

For development we use mac and ubuntu machines and for testing we use Windows. We have two server environments one for test/staging and the other for production. Module A takes an user input to set the environment, but module B does not. The developer has to give a test env build and a production env build to qc to complete the testing process. Here comes the dependency. Qc had to depend on dev to create a new build.

Generally once the Qc sign off the build, Qc release the version. Here comes another dependency. The developers need to share the required build files with the QC to release files. We share the source code of module A. In case of module B we share aar file for Android studio users, res and jar files for eclipse users. So Qc needs classes.jar, res, AndroidManifest.xml, moduleB.aar files to make a release.

##This is how we resolved the problems.

I always like to write shell scripts. I know we could have installed android studio in their systems, they can pull the latest code and can generate the required files, but as they wanted a simple script which gives the required files and i am very much interested in shell script, i took this as an opportunity to learn shell script and made an attempt to solve the problem.

##This is how we divided the problem into small steps.

  1. Get the current git branch and the debug mode (true/false) as input.
  2. Pull the latest code and checkout to the given branch.
  3. Read the config file and re write debug value from the input
  4. Remove the app/build directory and ModuleB/build directory.
  5. Build the project using gradle command.
  6. Copy the required files to output folder.
  7. Exit.

_<ins>1. Reading user input.</ins>_

{% highlight sh %}
echo -n "Enter the CB branch name > "
read cb_branch_name
echo -n "Enter CB DEBUG Mode (true/false) > "
read cb_debug_mode
{% endhighlight %}

_read cb_branch_name_ stores the user input (branch name) in cb_branch_name variable.

_read cb_debug_mode_ stores the debug mode (true-> test/ false->prod) in cb_debug_mode variable.

Click here to learn more about [keyboard inputs][4]

_<ins>2. Update ModuelB using git</ins>_

{% highlight sh %}
cd ModuleB/
git checkout .
git pull --all
git checkout $cb_branch_name
{% endhighlight %}

_git checkout ._ will reset all the change in all the files.

_git pull --all_ will pull the all the branches from the remote repo.

_git checkout $cb_branch_name_ will checkout to the given branch.

please refer [this][5] to know more about git commands.

_<ins>3. Change the config files's debug mode.</ins>_

{% highlight sh %}
sed -e "s/DEBUG = [a-z]*/DEBUG = $cb_debug_mode/g" src/com/test/Config.java > src/com/test/Config1.java
rm src/com/test/Config.java
mv src/com/test/Config1.java src/com/test/Config.java
cd ..
{% endhighlight %}

sed - [Stream Editor][6] helps to find a string from the stream and edit. It supports [regex pattern][7] to search the stream.

we write the output of sed to a new file called Config1.java and remove the current Config.java file.

then rename Config1 file as Config

_<ins>4. Cleaning the build.</ins>_

{% highlight sh %}
if [ -d app/build ]
  then rm -fr app/build/
fi

if [ -d ModuleB/build ]
  then rm -fr ModuleB/build/
fi
{% endhighlight %}

Removing app/build and ModuleB/build folders to make sure we always regenerate a new files.
we using [shell control flow][8] to check the availability of the files.  

_<ins>5. Gradle build</ins>_

{% highlight sh %}
./gradlew assembleDebug
{% endhighlight %}

[./ gradlew assembleDebug][9] build the project and generate apk, aar files.

_<ins>6. copy required files and place it in outputs.</ins>_

{% highlight sh %}
if [ -d outputs ]
  then rm -fr outputs/*
else
  mkdir outputs
fi
# lets copy the required files.
cp $APK_OUTPUT_PATH outputs/
cp $CB_AAR_FILE_PATH outputs/
cp $CB_CLASS_FILE_PATH outputs/
cp $CB_MANIFEST_PATH outputs/
cp -r $CB_RESOURCE_PATH outputs/
{% endhighlight %}

Removing the existing files in outputs directory using rm -r command.

Then copy the required files to outputs directory using cp command.

_<ins>7. Exit the script with success</ins>_

  {% highlight sh %}
  exit 0
  {% endhighlight %}

Exiting the project with [Code 0][10]

##Finally the script looks something like this.

{% highlight sh %}
#!/bin/bash
# Constants.

APK_OUTPUT_PATH="app/build/outputs/apk/app-debug.apk"
B_AAR_FILE_PATH="ModuleB/build/outputs/aar/ModuleB-release.aar"
B_CLASS_FILE_PATH="app/build/intermediates/exploded-aar/android-test-project/ModuleB/unspecified/classes.jar"
B_MANIFEST_PATH="app/build/intermediates/exploded-aar/android-test-project/ModuleB/unspecified/AndroidManifest.xml"
B_RESOURCE_PATH="app/build/intermediates/exploded-aar/android-test-project/ModuleB/unspecified/res"

# get user inputs .

echo -n "Enter the CB branch name > "
read cb_branch_name
echo -n "Enter CB DEBUG Mode (true/false) > "
read cb_debug_mode

# lets print the data now:

echo -n "
CB Branch : $cb_branch_name
CB Debug  : $cb_debug_mode
Do you want to continue (yes / no) :"
read user_confirmation

if [ "$user_confirmation" = "yes" ]
  # then echo $cb_debug_mode
  # echo $cb_branch_name
  then
  cd ModuleB/
  git checkout .
  git pull --all
  git checkout $cb_branch_name
  # lets change the Config.java file debug mode according to the userput.
  sed -e "s/DEBUG = [a-z]*/DEBUG = $cb_debug_mode/g" src/com/payu/ModuleB/Config.java > src/com/payu/ModuleB/Config1.java
  echo "$(pwd)"
  rm src/com/payu/ModuleB/Config.java
  mv src/com/payu/ModuleB/Config1.java src/com/payu/ModuleB/Config.java
  cd ..
  # we are good to go.

  # lets remove the old builds.
  if [ -d app/build ]
    then rm -fr app/build/
  fi

  if [ -d ModuleB/build ]
    then rm -fr ModuleB/build/
  fi

  # lets try to build the project using .gradle

  ./gradlew assembleDebug

  building the gradle project using [./gradlew command][9]


  # lets remove the old apk and copy the new apk into outputs folder.
  if [ -d outputs ]
    then rm -fr outputs/*
  else
    mkdir outputs
  fi

  # lets copy the required files.
  cp $APK_OUTPUT_PATH outputs/
  cp $B_AAR_FILE_PATH outputs/
  cp $B_CLASS_FILE_PATH outputs/
  cp $B_MANIFEST_PATH outputs/
  cp -r $B_RESOURCE_PATH outputs/

fi
exit 0
{% endhighlight %}

[1]: https://developer.android.com/sdk/installing/create-project.html#CreatingAModule
[2]: https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository
[3]: http://git-scm.com/docs/git-submodule
[4]: http://linuxcommand.org/lc3_wss0100.php
[5]: https://www.atlassian.com/git/tutorials/viewing-old-commits
[6]: http://www.gnu.org/software/sed/manual/sed.html
[7]: http://www.tldp.org/LDP/abs/html/x17129.html
[8]: http://linuxcommand.org/lc3_wss0080.php
[9]: http://developer.android.com/tools/building/building-cmdline.html
[10]: http://linuxcommand.org/wss0150.php
