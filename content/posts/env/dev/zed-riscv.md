---
title: 'Building the Zed editor for Risc-V'
date: 2025-02-17T17:52:40Z
draft: false
tags: ["Risc-V","Rust","Linux","Zed"]
---

I got access to a Linux Risc-V development board made by [DeepComputing](https://store.deepcomputing.io/products/dc-roma-risc-v-mainboard-for-framework-laptop-13) and started exploring if I could use it for development.

My idea was to run a code editor that can remote into another more powerful machine. So I can benefit from the low battery consumption of the system and still use my preferred keyboard layout (that is not supported when I use VSCode in a browser window).

I wanted to use either Zed or Visual Studio Code, both don't have a Risc-V build available. I first started to look at VSCode but it heavily depends on Electron, that doesn't have Risc-V support yet, this looked like it needs a lot of work. Zed on the other hand is built using the Rust ecosystem, where usually the crates can be compile for different architectures without any changes.

## Installing Rust

Installing the rust toolchain is pretty easy even on Risc-V, the installation can be done using one shell command (see the [getting started](https://www.rust-lang.org/learn/get-started) page).

## Building Zed

I first tried to clone the source code of Zed from their [GitHub Repo](https://github.com/zed-industries/zed) and building it using `cargo build`. The build goes well for some time until it tries to build [webrtc-sys](https://github.com/zed-industries/livekit-rust-sdks/tree/zed-patches/webrtc-sys), part of the [livekit](https://github.com/livekit) project that is used for realtime communication between two users for collaboration.

### Building libwebrtc

The webrtc-sys packages uses C bindings to the webrtc code in the [Chromium project](https://chromium.googlesource.com/chromium/src.git). Instead of downloading the whole Chromium project when building webrtc-sys, an archive containing prebuilt objects is downloaded.
