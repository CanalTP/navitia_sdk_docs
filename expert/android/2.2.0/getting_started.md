---
layout: main
title: Getting started
parent: Expert Android
grand_parent: Expert
nav_order: 1
permalink: /expert/android/getting-started
---

# Getting Started
{: .no_toc }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🧰  Requirements

Minimum Android SDK target: `21`

## 💻  Setup

The access to `Expert` module require valid credentials to our private artifactory. Add the following maven repository in the `build.gradle` of your project. Replace `USERNAME` and `PASSWORD` with your credentials:

```ruby
repositories {
    ...
    maven {
        credentials {
            username = USERNAME
            password = PASSWORD
        }
        url("https://kisiodigital.jfrog.io/kisiodigital/android-release")
    }
}
```
 
Add the following dependency in the `build.gradle` file of your application:

```ruby
dependencies {
    ...
    implementation("com.kisio.navitia.sdk.data:expert:2.2.0")
}
```

## 👨‍💻  Implementation

This module is set up by simply instanciate 2 objects: 
- `NavitiaConfiguration` will hold your Navitia token
- `NavitiaSDK` will hold `NavitiaConfiguration`

```kotlin
val navitiaSdk = NavitiaSDK(NavitiaConfiguration(token))
```

## 🚀  Launching

You can now call any endpoint from `navitiaSdk` and its variety of builders that will help you request Navitia. As an example:

```kotlin
navitiaSdk.physicalModesApi.newCoverageRegionPhysicalModesRequestBuilder()
    .withRegion("YOUR_COVERAGE")
    .get(object : ApiCallback<T> {
        override fun onSuccess(
            result: T,
            statusCode: Int,
            responseHeaders: MutableMap<String, MutableList<String>>?
        ) {
            // Success result
        }

        override fun onFailure(
            e: ApiException?,
            statusCode: Int,
            responseHeaders: MutableMap<String, MutableList<String>>?
        ) {
            // Failure result
        }

        override fun onUploadProgress(bytesWritten: Long, contentLength: Long, done: Boolean) {
            // Progress upload
        }

        override fun onDownloadProgress(bytesRead: Long, contentLength: Long, done: Boolean) {
            // Progress download
        }
    })
```

You can use this folliwing utility function to benefit from the advantages of coroutines

```kotlin
@InternalCoroutinesApi
@ExperimentalCoroutinesApi
private suspend fun <T> awaitApiCallback(block: (ApiCallback<T>) -> Unit): Result<T> =
    suspendCancellableCoroutine { cont ->
        block(object : ApiCallback<T> {

            override fun onSuccess(
                result: T,
                statusCode: Int,
                responseHeaders: MutableMap<String, MutableList<String>>?
            ) {
                cont.resume(Result.success(result)) {}
            }

            override fun onFailure(
                e: ApiException?,
                statusCode: Int,
                responseHeaders: MutableMap<String, MutableList<String>>?
            ) {
                cont.resume(Result.failure<ApiException>(e as ApiException)) {}
            }

            override fun onUploadProgress(bytesWritten: Long, contentLength: Long, done: Boolean) {
            }

            override fun onDownloadProgress(bytesRead: Long, contentLength: Long, done: Boolean) {
            }
        })
    }
```
and use it like so

```kotlin
val result = awaitApiCallback<PhysicalModes> {
    navitiaSDK.physicalModesApi.newCoverageRegionPhysicalModesRequestBuilder()
        .withRegion("YOUR_COVERAGE")
        .get(it)
}
```
