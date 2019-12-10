---
layout: post
title: "ssh tunneling"
date: 2019-11-22
categories: network
tags: ssh
---

# ssh tunneling

An SSH client connects to a Secure Shell server, which allows you to run terminal commands as if you were sitting in front of another computer. But an SSH client also allows you to “tunnel” a port between your local system and a remote SSH server.

There are three different types of SSH tunneling, and they’re all used for different purposes. Each involves using an SSH server to redirect traffic from one network port to another. The traffic is sent over the encrypted SSH connection, so it can’t be monitored or modified in transit.

You can do this with the ssh command included on Linux, macOS, and other UNIX-like operating systems. On Windows, which doesn’t include a built-in ssh command, we recommend the free tool PuTTY to connect to SSH servers. It supports SSH tunneling, too.

SSH client 는 Secure Shell Server에 접속해서 마치 다른 컴퓨터 앞에 앉아서 커맨드를 입력하듯이 만들어준다. 하지만 SSH client는 local 시스템과 remote ssh server와의 

https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/

https://medium.com/serverlessguru/what-is-a-ssh-tunnel-aws-ec2-8ebc394b8208

~~~bash
ssh -N -L 5432:mls-rdb.cabsm2hsndfe.ap-northeast-2.rds.amazonaws.com:5432 ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com -i /Users/nayoon/Downloads/mnodt-dev-key.pem
~~~