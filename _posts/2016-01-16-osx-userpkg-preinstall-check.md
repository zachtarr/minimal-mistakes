---
title:  "osx-userpkg-preinstall-check"
date:   2016-01-16
blog: true
---
One goal we have at the office is to setup consistent local admin accounts across every Mac at each of our clients offices. I created a some packages with CreateUserPkg to accomplish this, and I have been installing them on every new Mac I setup. Going forward, my plan was to deploy them using Munki. However, I ran into one problem:

Because many of these Macs were around before we started using the CreateUserPkg package, they already have a local admin account that was created manually. This account may conflict with the package in some way.

I wrote a very simple shell script to solve this problem. It first checks to see if an account with the given shortname exists. If it doesn't, then it exits with code 0. If it does, it compares a given UID and UUID to those of the local account on that Mac. It either reports back a conflict and exits with code 1, or it gives you the go-ahead to install the package and exits with code 0.

It can be run locally or via ARD, but I plan on using it as a [Preinstall Script](https://github.com/munki/munki/wiki/Pre-And-Postinstall-Scripts) in Munki.

I put this up on [GitHub](https://github.com/zachtarr/osx-userpkg-check)