# Understanding Docker Internals with Linux Namespaces

This repository is a hands-on exploration of Docker’s inner workings by diving into the Linux technologies that power it. By breaking down the foundational concepts like **Linux namespaces**, **filesystems**, and **networking**, this project bridges the gap between Docker’s high-level abstractions and the underlying kernel features.

Through this lab practice, I have:
- **Inspected Docker images**, delving into extracted layers and `manifest.json` files to understand how Docker stores and manages its filesystems.
- **Simulated Docker-like environments**, creating isolated namespaces for filesystems and networking.
- **Appreciated Docker’s automation**, learning firsthand the complexity that Docker abstracts away.

Each experiment is documented with **step-by-step guides** in markdown files, designed for developers, engineers, and learners who want to:
- **Demystify Docker** by understanding its relationship with Linux namespaces.
- **Build their own "mini containers"** using raw namespaces.
- **Appreciate containerization** by replicating Docker functionality manually.

---

## **Table of Contents**
- **Inspecting and Understanding Docker Filesystem Inside Docker Images**  
  [Inspect Docker Image Layers](study_docker/docker_inspect_filesystem.md)

- **Create a Minimal Container with an Isolated Filesystem**  
  [Namespace Filesystem](namespaces/00_namespaces_filesystem.md)

- **Enable Networking Between Host and Namespaced Processes**  
  [Namespace Networking](namespaces/10_namespace-networking.md)

---

Whether you're curious about the magic of Docker or eager to understand how it’s built on top of Linux kernel features, this repository offers a refreshing, practical learning experience. Dive in and start your journey toward mastering the internals of containerization!

I hope you find this repository as refreshing and enlightening as I did while working on it!
