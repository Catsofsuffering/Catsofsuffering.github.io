---
title: Jenkins的搭建与部署
tags:
---
## 安装 Jenkins

```
pvcreate /dev/sdc 
vgcreate jenkinsvg /dev/sdc
lvcreate -l 100%FREE -n jenkinslv jenkinsvg
mkfs.ext4 /dev/jenkinsvg/jenkinslv

mkdir -p /opt/jenkins
mount /dev/jenkinsvg/jenkinslv /opt/jenkins

cat << EOF >> /etc/fstab
/dev/mapper/sjenkinsvg-jenkinslv /opt/jenkins/ ext4 defaults 0 0
EOF

mv /tmp/jenkins.war /opt/jenkins/
groupadd jenkins
useradd -m -r jenkins -s /bin/bash -g jenkins
chown -R jenkins:jenkins /opt/jenkins/
echo JENKINS_HOME=/opt/jenkins/ >> 
su - jenkins -c "nohup java -jar /opt/jenkins/jenkins.war > /opt/jenkins/jenkins.log 2>&1 &"
```

1. jenkinsUrl/safeRestart - Let you to wait for running JOBS to get complete and do a RESTART.
2. jenkinsUrl/restart - Do a restart immediately without waiting for the jobs which are running currently.
3. jenkinsUrl/exit - It stops/shutdown the JENKINS services
4. jenkinsUrl/reload - To reload the configuration changes.
