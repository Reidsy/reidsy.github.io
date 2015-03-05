---
layout:	post
title:	"Locking down Arch"
date:	2015-03-04
categories: sysadmin
---
# Disable root login

	passwd -l root
    
# Disable password ssh login

Find the lines

	# To disable tunneled clear text passwords, change to no here!
	#PasswordAuthentication yes 
	#PermitEmptyPasswords no

and change to

	# To disable tunneled clear text passwords, change to no here!
	PasswordAuthentication no 
	PermitEmptyPasswords no
