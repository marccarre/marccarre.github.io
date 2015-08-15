---
layout: post
status: publish
published: true
title: Install and configure Tomcat 7 under MacOSX (or Linux)
author:
  display_name: zhong
  login: Zh0nG
  email: zh0nggu0.1337@gmail.com
  url: http://www.zh0ng.net
author_login: Zh0nG
author_email: zh0nggu0.1337@gmail.com
author_url: http://www.zh0ng.net
wordpress_id: 160
wordpress_url: http://www.zh0ng.net/?p=160
date: '2011-09-25 08:56:00 +0100'
date_gmt: '2011-09-25 07:56:00 +0100'
categories:
- IT
- Java
- Web
tags: []
comments: []
---
<p>This article explains how to manually install and configure Tomcat 7 under MacOSX &#47; Linux:</p>
<ol>
<li><a href="#prerequisites">Prerequisites<&#47;a><&#47;li>
<li><a href="#downloadAndInstall">Download and install Tomcat 7<&#47;a><&#47;li>
<li><a href="#configureFilePermissions">Configuration: setting the file permissions<&#47;a><&#47;li>
<li><a href="#configureTomcat">Configuration: Tomcat 7<&#47;a><&#47;li>
<li><a href="#startTomcat">Start Tomcat<&#47;a><&#47;li><br />
<&#47;ol></p>
<p><u>Note<&#47;u>: installing under Linux is roughly the same, except in step #3, where you may need to change the users&#47;groups to the appropriate values.</p>
<ol>
<hr&#47;>
<li><a name="prerequisites"><b><u>Prerequisites<&#47;u><&#47;b><&#47;a><&#47;li><br />
<br&#47;></p>
<p>
Ensure you have Java 6 installed as it is required for Tomcat 7:<br />
[sourcecode language="bash"]<br />
$ java -version<br />
[&#47;sourcecode]<br />
[sourcecode language="bash" gutter="false"]<br />
java version "1.6.0_26"<br />
Java(TM) SE Runtime Environment (build 1.6.0_26-b03-384-10M3425)<br />
Java HotSpot(TM) 64-Bit Server VM (build 20.1-b02-384, mixed mode)<br />
[&#47;sourcecode]<br />
<&#47;p></p>
<hr&#47;>
<li><a name="downloadAndInstall"><b><u>Download and install Tomcat 7<&#47;u><&#47;b><&#47;a><&#47;li><br />
<br&#47;></p>
<p>
Download the Tomcat 7 archive:<br />
[sourcecode language="bash"]<br />
$ cd &#47;usr&#47;share<br />
$ sudo mkdir tomcat<br />
$ cd tomcat<br />
$ sudo wget http:&#47;&#47;www.apache.org&#47;dist&#47;tomcat&#47;tomcat-7&#47;v7.0.21&#47;bin&#47;apache-tomcat-7.0.21.tar.gz<br />
[&#47;sourcecode]</p>
<p>Unpack it and create a link to the "current" version of Tomcat:<br />
[sourcecode language="bash"]<br />
$ sudo tar xvzf apache-tomcat-7.0.21.tar.gz<br />
$ sudo rm apache-tomcat-7.0.21.tar.gz<br />
$ sudo ln -s apache-tomcat-7.0.21 current<br />
[&#47;sourcecode]</p>
<p>Ensure that the link has correctly been created:<br />
[sourcecode language="bash"]<br />
$ ls -altr<br />
[&#47;sourcecode]<br />
[sourcecode language="bash" gutter="false"]<br />
total 8<br />
drwxr-xr-x&nbsp; 75 root&nbsp; wheel&nbsp; 2550 25 sep 08:55 ..<br />
drwxr-xr-x&nbsp; 13 root&nbsp; wheel&nbsp;&nbsp; 442 25 sep 08:56 apache-tomcat-7.0.21<br />
lrwxr-xr-x&nbsp;&nbsp; 1 root&nbsp; wheel&nbsp;&nbsp;&nbsp; 20 25 sep 09:00 current -> apache-tomcat-7.0.21<br />
drwxr-xr-x&nbsp;&nbsp; 4 root&nbsp; wheel&nbsp;&nbsp; 136 25 sep 09:00 .<br />
[&#47;sourcecode]<br />
<&#47;p></p>
<hr&#47;>
<li><a name="configureFilePermissions"><b><u>Configuration: setting the file permissions<&#47;u><&#47;b><&#47;a><&#47;li><br />
<br&#47;></p>
<p>
When running the below commands, replace <User> by the name of the user who should own Tomcat, or alternatively whatever the command whoami returns:<br />
[sourcecode language="bash"]<br />
$ sudo chown -R root:wheel .<br />
$ sudo chmod 644 conf&#47;*<br />
$ sudo chown :staff conf&#47;tomcat-users.xml<br />
$ sudo chmod 640 conf&#47;tomcat-users.xml<br />
$ sudo chown :staff logs temp webapps work<br />
$ sudo chmod 2770 logs temp webapps work<br />
[&#47;sourcecode]</p>
<p>A few comments about the above commands:</p>
<ul>
<li>1) All files now belong to root user.<&#47;li>
<li>2) Files in conf can be edited only by owner (here, root), but can be read by anyone.<&#47;li>
<li>3) tomcat-users.xml (to configures roles in Tomcat, etc.) now belongs to <User>.<&#47;li>
<li>4) Owner (here, <User>) can edit&#47;read tomcat-users.xml. Users in the same group can only read it. Others can't do anything with it.<&#47;li>
<li>5) logs, temp, webapps and work now belong to <User>.<&#47;li>
<li>6) When a new file is created in these directories, it will inherit the group of the directory (and not of the user who created the file).<&#47;li><br />
<&#47;ul><br />
<br&#47;><br />
Ensure you understand what these commands mean and what security flaws they can potentially introduce in <span style="text-decoration: underline;">your<&#47;span> specific environment.<br />
<br&#47;><br />
More details can be found on <a title="Linux file permissions" href="http:&#47;&#47;www.tuxfiles.org&#47;linuxhelp&#47;filepermissions.html" target="_blank">these<&#47;a> <a title="Linux file owners" href="http:&#47;&#47;www.tuxfiles.org&#47;linuxhelp&#47;fileowner.html" target="_blank">three<&#47;a> <a title="Linux file permissions: SETUID, SETGID, etc." href="http:&#47;&#47;www.zzee.com&#47;solutions&#47;linux-permissions.shtml#setuid" target="_blank">pages<&#47;a>.</p>
<hr&#47;>
<li><a name="configureTomcat"><b><u>Configuration: Tomcat 7<&#47;u><&#47;b><&#47;a><&#47;li><br />
<br&#47;></p>
<p>
Create bin&#47;setenv.sh and put your environment variables in it:<br />
[sourcecode language="bash"]<br />
$ cd bin<br />
$ sudo touch setenv.sh<br />
$ cd ..<br />
[&#47;sourcecode]</p>
<p>Ensure your connector encodes URLs using UTF-8:<br />
[sourcecode language="bash"]<br />
$ cd conf<br />
$ sudo nano server.xml<br />
[&#47;sourcecode]<br />
Look for your connector and ensure it has, among others, the URIEncoding attribute set to the correct value:<br />
[sourcecode language="bash" gutter="false"]<br />
<Connector port="8080" URIEncoding="UTF-8"&#47;><br />
[&#47;sourcecode]</p>
<p>[sourcecode language="bash"]<br />
$ cd ..<br />
[&#47;sourcecode]</p>
<hr&#47;>
<li><a name="startTomcat"><b><u>Start Tomcat<&#47;u><&#47;b><&#47;a><&#47;li><br />
<br&#47;></p>
<p>
Check whether port 8080 is free or if some processes are already using it:<br />
[sourcecode language="bash"]$ lsof -ni tcp:8080[&#47;sourcecode]or[sourcecode language="bash"]$ netstat -ln | grep 8080[&#47;sourcecode]</p>
<p>Start Tomcat:<br />
[sourcecode language="bash"]<br />
$ cd bin<br />
$ .&#47;startup.sh<br />
[&#47;sourcecode]</p>
<p>Open <a title="http:&#47;&#47;localhost:8080" href="http:&#47;&#47;localhost:8080" target="_blank">http:&#47;&#47;localhost:8080<&#47;a> in your favourite web browser and enjoy!<br />
<a href="http:&#47;&#47;www.zh0ng.net&#47;zh0ng&#47;wp-content&#47;uploads&#47;2011&#47;09&#47;01_tomcat_running.png"><img src="http:&#47;&#47;www.zh0ng.net&#47;zh0ng&#47;wp-content&#47;uploads&#47;2011&#47;09&#47;01_tomcat_running.png" alt="Screenshot of Tomcat, up and running" title="Tomcat up and running" width="846" height="513" class="alignnone size-full wp-image-179" &#47;><&#47;a><br />
<&#47;ol></p>
