---
title: RPM ref. sheet
layout: page
resource: true
date:   2016-02-20 16:02:09 +0000
---

This reference sheet describes how to quickly setup a rpm build env and start building packages.

1. Create a user/group makerpm:makerpm
2. (Optional) Start by installing *rpmdevtools* package [fedora desc.](https://fedoraproject.org/wiki/Rpmdevtools)
3. (Optional) Use *rpmdev-setuptree* command to build rpm file tree

The last two steps allows you to easily build the rpm tree structure. But depending on your configuration
you may want to manage manually. This is what a build directory tree looks like:

path | details | alias
--- | --- | ---
~/rpmbuild/SPECS	| RPM specifications (.spec) files 						| %_specdir
~/rpmbuild/SOURCES	| Pristine source package (e.g. tarballs) and patches 				| %_sourcedir
~/rpmbuild/BUILD	| Source files are unpacked and compiled in a subdirectory underneath this. 	| %_builddir
~/rpmbuild/BUILDROOT	| Files are installed under here during the %install stage. 			| %_buildrootdir
~/rpmbuild/RPMS		| Binary RPMs are created and stored under here. 				| %_rpmdir
~/rpmbuild/SRPMS	| Source RPMs are created and stored here. 					| %_srcrpmdir


### RPM env

RPM is a package manager that allows you to build RH* compatible packages. To achieve this, all you need to do
is to write a package descriptor file called a SPEC file. It describes the process of building your package
from the sources to the final .rpm file.

RPM comes with a lot of macros and variables. You can refer to them in your package descriptor file
using the % symbol. You can also define your own build related variables and even override system default ones.
Defining a variable is usually done with the %define directive at the top of your spec file.

To get the value of a specific build variable:

```
$ rpm --eval '%{_topdir}'
/home/makerpm/rpmbuild
```

To list RPM macros, just run:

```
rpm --showrc
```

A word about the rpm buildroot, while building a package rpm uses a dir as a root for the upcoming packaged files.
It can be overriden with the *Buildroot:* instruction to allow other users than root to build packages.

During installation, you can access the buildroot path with %_buildrootdir or $BUILD_ROOT_DIR env variables.

### Start writing a spec file :

```
# RPM SPEC FILE SKELETON

%define app         <yourappname>
%define version     <version>

# on start actions
%(echo 'you can run bash functions here')

Summary:    <your app description>
Name:       %{app}
Version:    %{version}
BuildArch:  <targeted arch>
Release:    <build release number>
License:    <License>
URL:        <your website>
Requires:   <comma separated list of dependencies>
BuildRequires: <build required dependencies>
Buildroot:  <user build dir>

%description
# complete description of the package comes here

%prep
# The following line extracts your sources from %{_sourcedir} and put them in the %{_builddir}
setup -q

%build
# this is where you actually build your sources, for instance call ./configure and make

%install
# this is the install scripting part, here move your files to the appropriate location in the buildroot

%clean
# cleaning instructions go there

%pre
# script run before package installation

%post
# script run after package installation

%preun
# script run before package removal

%postun
# script run after package removal

%files
%defattr(-,root,root)
# This is where you list all the files installed by your package and edit permissions on them
# Adding *%doc* or *%config* to the line mentions whether a file is part of documentation or configuration.

```

If you want to have a better idea of what you can put in this file, have a look at an existing spec file
over the internet.

### Build your package:

A spec file is divided into several sections, *%description*, *%prep*, *%build*, *%install* and more.
Each of these sections can be executed through the build process as a separate build step.

To build your package run the following command:

```
rpmbuild -v -ba --clean ~/rpmbuild/SPECS/package.spec
```

shortcut | Role
--- | ---
p	  | rpmbuild -bp executes %prep
c	  | rpmbuild -bc executes %prep, %build
i	  | rpmbuild -bi executes %prep, %build, %install, %check
b	  | rpmbuild -bb executes %prep, %build, %install, %check, package (bin)	
a	  | rpmbuild -ba executes %prep, %build, %install, %check, package (bin, src)
l	  | rpmbuild -bl checks %files list

### Time to check and install your package

To verify your package, run:

```
rpm -V <yourpackage> 
```

See what your package is about to perform with:

```
rpm -ivh --test <packagename>
```

Review package metadata and content:

```
rpm -qilp <packagename>
```

To install your newly built package, just run:

```
rpm -i <yourpackage> or rpm -Uvh <yourpackage>
```

To remove a package:

```
rpm -e <your package>
```

### Resources

Complete user guides can be found here: 

- [fedora guide](https://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/)
- [maximum rpm book](http://www.rpm.org/max-rpm-snapshot/)


