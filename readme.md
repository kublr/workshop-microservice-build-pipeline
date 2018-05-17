# 1. Introduction

While containers can help to manage complex systems and processes, the tools to
manage them come with their own set of challenges. This session will walk
through the steps IT and development teams should take to setup a standard
DevOps and cluster orchestration platform based on a Kubernetes cluster in AWS.
The ultimate goal of such a platform is to allow developers to further improve
the efficiency of their work, repurpose work to save time, and focus on the
unique business requirements of each project.

This presentation will help developers better understand how modern open source
technologies such as Kubernetes, Jenkins, and Nexus may be used to build a
DevOps stack as reliable as other packaged options. The goal of the session is
to encourage developers, and the companies they work for, to use open source
products and participate more in their upstream development.

# 2. Session plan

The hands-on session will follow through the next general steps:

1.  Setup Kubernetes cluster in AWS

2.  Setup AWS EFS for build pipeline data

3.  Deploy and configure Nexus repository in K8S cluster

4.  Deploy and configure Jenkins in K8S cluster

5.  Setup Jenkins to run dynamically allocated workers as pods in K8S cluster

6.  Setup a Jenkins job to build Docker image in K8S cluster

7.  Setup CI for a service project to automatically deploy to K8S

8.  Discuss possible further improvements: security, environment sharing between
    multiple teams and projects, etc

Detailed step-by-step instructions for the session are available in
[docs/session.md](docs/session.md)
