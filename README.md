# Strive SDK

## What is the Strive SDK?

The Strive SDK are native bindings for Android and iOS to enable WebRTC-based live streaming CDN offloading.

The SDK itself is written in Go and cross compiled to ARM.
The API is available in Java for Android and Swift/ObjC for iOS.

## Which players are supported ?

The Strive SDK is compatible with all players as long as HLS or MPEG-DASH is used for the transport level.
To achieve this the SDK acts as a software proxy and intercepts player requests to offload approx. 70%-80% of the required traffic on average to other vieweres.

## What do I need to get started?

Checkout [releases](https://github.com/chunkedswarm/strive-sdk/releases) and select a matching package to download.

For Android there is a single **AAR** file and for iOS there is a single **Framework** folder.
