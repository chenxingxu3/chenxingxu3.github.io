---
layout:     post
title:      Oracle Database Learning Notes (I)
subtitle:   Build server-side runtime environment
date:       2020-05-20
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - Oracle
---

# Overview

The operating system environment I use to build Oracle database is Windows XP SP3, and the virtual machine used is VMware Workstation.

# Preliminary Preparation

1. VMware Workstation （https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html）
2. Windows XP SP3 ISO file （https://msdn.itellyou.cn/）

# Building process

1、File -> New Virtual Machine. 

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_160914.134](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_160914.134.jpg)

2、After the "New Virtual Machine Wizard" starts, just follow the step-by-step settings in the following diagram.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_160919.298](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_160919.298.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_160922.052](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_160922.052.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_160932.356](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_160932.356.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161036.328](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161036.328.jpg)

The settings below can be left as default, or you can set them according to your own preferences. I set the settings according to my own preferences.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161046.224](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161046.224.jpg)

Click on "Customize Hardware".

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161050.420](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161050.420.jpg)

Select "New CD/DVD (IDE)" on the left, click "Use ISO image file" on the right, click "Browse" and select the image file for Windows XP. Then click "Close" at the bottom to close this window.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161112.490](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161112.490.jpg)

Just click "Finish".

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161127.867](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161127.867.jpg)

3. After the virtual machine is newly created, follow the steps in the figure below to install the operating system in turn.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161133.321](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161133.321.jpg)

Press the C key to partition the disc.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161201.009](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161201.009.jpg)

For development and learning purposes, just divide it into one section and enter it directly.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161207.320](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161207.320.jpg)

Enter directly to start the system installation.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161211.721](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161211.721.jpg)

Because it is a virtual machine, not real hardware, you can format it by selecting the option with the word "fast" as shown below.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161219.179](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161219.179.jpg)

Wait patiently for the installation to complete.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161239.648](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161239.648.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161258.402](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161258.402.jpg)

4、After the system installation is complete, follow the steps in the diagram below to install VMware Tools. Find the virtual machine you have built, right-click on it, and select "Install VMware Tools".

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161506.012](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161506.012.jpg)

Open "My Computer" and you will see VMware Tools as shown below.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161512.860](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161512.860.jpg)

Keep the default, Next->Next->... until the end.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161517.012](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161517.012.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161520.624](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161520.624.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161527.636](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161527.636.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161548.280](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161548.280.jpg)

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161555.468](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161555.468.jpg)

When the installation is complete, select "Yes" and restart the virtual machine.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161610.257](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161610.257.jpg)

# Performance optimization (optional)

You can refer to the following practices to improve the operation efficiency of Windows XP in the virtual machine.

1、Click the right mouse button on the desktop and select "Properties".

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161657.751](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161657.751.jpg)

2、Select the Windows Classic theme. Click the "OK" button below to complete the settings.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161708.272](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161708.272.jpg)

3、Right click on "My Computer" and select "Properties".

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161717.874](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161717.874.jpg)

4、Go to the "Advanced" tab and click the "Settings" button in Performance.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161726.146](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161726.146.jpg)

5、Click on "Adjust for best performance". Click the "OK" button below to complete the settings.

![bandicam 2020-05-20 11-48-03-277.mp4_20200520_161729.010](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 11-48-03-277.mp4_20200520_161729.010.jpg)

# Turn off the firewall

To facilitate development, it is recommended that the firewall be turned off.

![bandicam 2020-05-20 17-47-08-523.mp4_20200520_201425.164](/img-post/2020-05-20-learning-notes-01/bandicam 2020-05-20 17-47-08-523.mp4_20200520_201425.164.jpg)

This completes the work of building the server-side runtime environment.

