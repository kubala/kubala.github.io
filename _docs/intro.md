---
title: Intorduction to Goma
description: Short introduction to this document
next-link: /docs/prerequisites
next-name: Prerequisites
prev-link: /
prev-name: Start Page
---

# Introduction

### What is Goma

Goma is a tool that lets you delegate your local compiler job to a remote build system. Depending on the capacity of the remote system the jobs may be run remotely in parallel dramatically speeding up the build. Also, Goma provides access to a shared cache of build outputs so you don't have to run that same tasks all over again - you can fetch outputs from the cache without running an actual build.

### Why Goma

Goma is not a new concept. There are other tools like ccache, distcc, etc... that can do the same thing. What is special about Goma is that:

- It combines distributed build and cache into one service - no more problems with shared ccache storage, slow NFS, etc
- It was designed for scalability and flexibility. Hundreds of requests? Thousands of workers? Not a big deal.
- The client works out of the box on three major platforms (Linux, Windows and OSX)

### Is Goma Suitable For Everyone?

That depends. distcc deployment is usually completely free of charge. In most cases you install it on existing developers' machines using their computing power while they're idling. Goma comes with a hidden cost. It is designed to be deployed to a dedicated cluster therefore you will need to have or build one. Also, Goma is CPU hungry - you'll need hundreds (if not thousands) of CPUs to have Goma spread its wings.
