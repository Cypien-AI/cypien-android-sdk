# CypienSDK for Android

[![Android](https://img.shields.io/badge/Android-API%2021%2B-green?logo=android)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9%2B-purple?logo=kotlin)](https://kotlinlang.org)
[![Version](https://img.shields.io/badge/version-1.0.0-orange)](https://github.com/Cypien-AI/cypien-android-sdk)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

Behavioral analytics and personalized content delivery SDK for Android.

## Requirements

- Android API 21+ (Android 5.0)
- Kotlin 1.9+

## Installation

### 1. Add Maven repository

In `settings.gradle` (project level):

```gradle
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://raw.githubusercontent.com/Cypien-AI/cypien-android-sdk/main/")
        }
    }
}
```

### 2. Add dependency

In `app/build.gradle`:

```gradle
dependencies {
    implementation 'io.cypien:sdk-android:1.0.2'

    // Required dependencies
    implementation 'androidx.security:security-crypto:1.1.0-alpha06'
    implementation 'androidx.work:work-runtime-ktx:2.9.0'
    implementation 'androidx.lifecycle:lifecycle-process:2.7.0'
}
```

## Setup

Initialize the SDK in your `Application` class with your **Workspace ID** and **API Key** from the Cypien Dashboard.

```kotlin
import android.app.Application
import io.cypien.sdk.core.CypienConfig
import io.cypien.sdk.core.CypienSDK

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        CypienSDK.initialize(
            application = this,
            config = CypienConfig(
                workspaceId = "YOUR_WORKSPACE_ID",
                apiKey = "YOUR_API_KEY"
            )
        )
    }
}
```

Register in `AndroidManifest.xml`:

```xml
<application android:name=".MyApplication" ...>
```

## Basic Usage

```kotlin
// Track screens
CypienSDK.trackScreen("/products/sku-123", "Product Detail")

// Personalized description
LaunchedEffect(product.sku) {
    CypienSDK.content.fetchDescription(product.sku, "product_description", product.category)
    CypienSDK.content.onContentReady("product_description") { result ->
        if (result is CypienContentResult.Description) label.text = result.text
    }
}

// Personalized product image
LaunchedEffect(product.sku) {
    CypienSDK.content.fetchProductImage(product.sku, "product_image", product.category)
    CypienSDK.content.onContentReady("product_image") { result ->
        if (result is CypienContentResult.Image) imageView.load(result.url)
    }
}

// Commerce events
CypienSDK.commerce.viewItem(item, currency = "USD", value = 49.99)
CypienSDK.commerce.addToCart(item, currency = "USD", value = 49.99)
CypienSDK.commerce.purchase(transactionId = "TXN-001", items = items, currency = "USD", value = 99.99)
```

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `workspaceId` | — | **Required** — from Cypien Dashboard |
| `apiKey` | — | **Required** — from Cypien Dashboard |
| `debugMode` | `false` | Verbose Logcat logging |
| `emitInterval` | `15L` | Seconds between event batch sends |
| `emitMode` | `DUAL` | `DUAL`, `GA4_ONLY`, `BACKEND_ONLY` |

## License

MIT © [Cypien AI](https://cypien.ai)
