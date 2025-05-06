---
layout: post
title: "Building Linux From Scratch: A Journey into the Heart of Linux"
date: 2025-05-06 10:05 +0200
author: borko
description: "Building Linux From Scratch: A Journey into the Heart of Linux"
categories: [linux]
tags: linux
image: "/assets/img/posts/2025-05-06-lfs/lfs.png"
---

![LFS](/assets/img/posts/2025-05-06-lfs/lfs.png){: .shadow style="max-width: 80%" }

## Introduction

After over a decade of using Linux, primarily Ubuntu, I felt it was time to delve deeper into the system's internals. My goal was to understand how Linux operates beneath the surface, beyond the convenience of package managers and pre-configured environments. This curiosity led me to embark on the Linux From Scratch (LFS) project, a challenging yet rewarding experience that offers a comprehensive understanding of Linux's core components.

![LFS neofetch](/assets/img/posts/2025-05-06-lfs/neofetch.png){: .shadow style="max-width: 90%" }

## Motivation and Background

Having used distributions like `Ubuntu`, `Debian`, and `Fedora`, I was comfortable with Linux's user-facing aspects. However, I realized that to truly grasp the system's workings, I needed to build it from the ground up. LFS presented itself as the perfect opportunity to achieve this, providing a step-by-step guide to constructing a Linux system entirely from source code.

## Preparation and Setup

Initially, I was hesitant about starting LFS, questioning whether it was the right path. However, after reading positive reviews and experiences online, I decided to proceed. I followed the official LFS book closely and occasionally consulted Reddit for solutions to specific issues.

For the build environment, I used `VirtualBox` with a `Debian` host system. Setting up the virtual machine posed challenges, particularly with partitioning. I encountered issues that led to 2 failed attempts before successfully configuring the partitions correctly.

## Building the System

I opted for LFS version 12.3 with systemd ([link to latest stable systemd LFS](https://www.linuxfromscratch.org/lfs/view/stable-systemd/)). The build process spanned approximately three weeks, as I worked on it intermittently. Following the book meticulously, I encountered difficulties, especially during the kernel compilation phase. To safeguard my progress, I took snapshots of the virtual machine, allowing me to revert to stable states when necessary.

An invaluable resource during this phase was the [Kernotex YouTube channel](https://www.youtube.com/@Kernotex). The creator's detailed explanations and walkthroughs provided clarity on complex steps, supplementing the information in the LFS book.

## Post-Build Configuration

Upon completing the base LFS system, I refrained from adding additional software immediately. My focus was on ensuring the system booted correctly and functioned as intended. A notable hiccup occurred when I misconfigured GRUB, causing it to boot the kernel from the Debian host instead of the custom-compiled one. This misconfiguration led to unexpected behavior until I rectified the GRUB settings, after which the system operated smoothly.

## Reflections and Insights

The most significant lesson from this experience was the importance of thoroughly reading documentation and understanding each command's purpose. Merely copying and pasting commands without comprehension diminishes the learning experience. This project deepened my appreciation for the complexity and elegance of Linux, highlighting the immense effort invested in its development over the years.

While I found the LFS journey enlightening, I recognize that it's not for everyone. The process is time-consuming and can be overwhelming, especially for those without prior Linux experience. However, for individuals passionate about understanding Linux at a fundamental level, LFS offers unparalleled insights.

## Conclusion

Embarking on the LFS project was a transformative experience that enriched my understanding of Linux. It challenged me to engage with the system's core components, fostering a deeper appreciation for its architecture and design. For those seeking to explore Linux beyond the surface, LFS serves as both a rigorous challenge and a rewarding educational endeavor.
