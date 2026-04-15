# CypienSDK for Android

[![Android](https://img.shields.io/badge/Android-API%2021%2B-green?logo=android)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9%2B-purple?logo=kotlin)](https://kotlinlang.org)
[![Version](https://img.shields.io/badge/version-1.0.2-orange)](https://github.com/Cypien-AI/cypien-android-sdk)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

Behavioral analytics and personalized content delivery SDK for Android. CypienSDK enables real-time user behavior tracking, AI-driven product personalization, and A/B testing for e-commerce applications — with a single, lightweight integration.

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Setup](#setup)
- [Page Integration](#page-integration)
  - [Home Screen](#home-screen)
  - [Category / List Screen](#category--list-screen)
  - [Product Detail Screen](#product-detail-screen)
  - [Cart Screen](#cart-screen)
  - [Checkout Screen](#checkout-screen)
  - [Purchase / Order Confirmation](#purchase--order-confirmation)
  - [Search Screen](#search-screen)
- [User Management](#user-management)
- [Debug & Testing](#debug--testing)
- [Privacy & GDPR](#privacy--gdpr)
- [Configuration Reference](#configuration-reference)
- [API Reference](#api-reference)
- [License](#license)

---

## Requirements

- Android API 21+ (Android 5.0 Lollipop)
- Kotlin 1.9+

---

## Installation

### 1. Add the Maven repository

In your project-level `settings.gradle` (or `settings.gradle.kts`), add the Cypien Maven repository inside `dependencyResolutionManagement`:

```gradle
// settings.gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://raw.githubusercontent.com/Cypien-AI/cypien-android-sdk/main/")
        }
    }
}
```

### 2. Add the dependencies

In your app-level `app/build.gradle`:

```gradle
// app/build.gradle
dependencies {
    implementation 'io.cypien:sdk-android:1.0.2'

    // Required transitive dependencies
    implementation 'androidx.security:security-crypto:1.1.0-alpha06'
    implementation 'androidx.work:work-runtime-ktx:2.9.0'
    implementation 'androidx.lifecycle:lifecycle-process:2.7.0'
}
```

> **Note:** The three `androidx` dependencies are required transitive dependencies that must be declared explicitly because the SDK is distributed as an AAR. If your project already declares compatible versions of these libraries, you do not need to add them again.

---

## Setup

### 1. Create an Application class

Initialize the SDK once, at application startup, inside your `Application` subclass. Obtain your **Workspace ID** and **API Key** from the [Cypien Dashboard](https://app.cypien.ai).

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

### 2. Register in AndroidManifest.xml

Tell Android to use your `Application` class:

```xml
<application
    android:name=".MyApplication"
    android:label="@string/app_name"
    ...>
```

> **Security tip:** Do not hardcode credentials in source code that is committed to version control. Use `BuildConfig` fields injected from `local.properties`, or retrieve them from your backend at runtime.

---

## Page Integration

The SDK tracks user behavior by recording the screens a user visits and the commerce events they trigger. Call `CypienSDK.trackScreen` at the start of each screen and fire the appropriate commerce event as the user interacts.

### Home Screen

```kotlin
import io.cypien.sdk.core.CypienSDK

@Composable
fun HomeScreen() {
    LaunchedEffect(Unit) {
        CypienSDK.trackScreen(screenPath = "/home", screenName = "Home")
    }

    // ... your composable content
}
```

### Category / List Screen

```kotlin
@Composable
fun CategoryScreen(categoryName: String) {
    LaunchedEffect(categoryName) {
        CypienSDK.trackScreen(
            screenPath = "/category/$categoryName",
            screenName = categoryName
        )
    }

    // ... your composable content
}
```

### Product Detail Screen

The Product Detail screen is where personalization is most impactful. Use `fetchDescription` and `fetchProductImage` to retrieve AI-personalized content for the current user.

```kotlin
import io.cypien.sdk.core.CypienSDK
import io.cypien.sdk.model.CypienContentResult
import io.cypien.sdk.model.CypienItem

@Composable
fun ProductDetailScreen(product: Product) {
    var personalizedDescription by remember { mutableStateOf(product.defaultDescription) }
    var personalizedImageUrl by remember { mutableStateOf(product.defaultImageUrl) }

    LaunchedEffect(product.sku) {
        // Track the screen view
        CypienSDK.trackScreen(
            screenPath = "/products/${product.sku}",
            screenName = "Product Detail"
        )

        // Request personalized description
        CypienSDK.content.fetchDescription(
            itemId = product.sku,
            slotId = "product_description",
            category = product.category
        )

        // Request personalized product image
        CypienSDK.content.fetchProductImage(
            itemId = product.sku,
            slotId = "product_image",
            category = product.category
        )

        // Handle personalized description result
        CypienSDK.content.onContentReady("product_description") { result ->
            if (result is CypienContentResult.Description) {
                personalizedDescription = result.text
            }
        }

        // Handle personalized image result
        CypienSDK.content.onContentReady("product_image") { result ->
            if (result is CypienContentResult.Image) {
                personalizedImageUrl = result.url
            }
        }
    }

    // Track view_item commerce event
    LaunchedEffect(product.sku) {
        val item = CypienItem(
            itemId = product.sku,
            itemName = product.name,
            itemCategory = product.category,
            price = product.price
        )
        CypienSDK.commerce.viewItem(
            item = item,
            currency = "USD",
            value = product.price
        )
    }

    // ... render product UI with personalizedDescription and personalizedImageUrl
}
```

### Cart Screen

```kotlin
@Composable
fun CartScreen(cartItems: List<CartItem>) {
    LaunchedEffect(Unit) {
        CypienSDK.trackScreen(screenPath = "/cart", screenName = "Cart")
    }

    // ... your composable content
}

// Call this when the user taps "Add to Cart" on any screen
fun onAddToCartClicked(product: Product) {
    val item = CypienItem(
        itemId = product.sku,
        itemName = product.name,
        itemCategory = product.category,
        price = product.price
    )
    CypienSDK.commerce.addToCart(
        item = item,
        currency = "USD",
        value = product.price
    )
}
```

### Checkout Screen

```kotlin
@Composable
fun CheckoutScreen(cartItems: List<CartItem>, cartTotal: Double) {
    LaunchedEffect(Unit) {
        CypienSDK.trackScreen(screenPath = "/checkout", screenName = "Checkout")

        val items = cartItems.map { cartItem ->
            CypienItem(
                itemId = cartItem.sku,
                itemName = cartItem.name,
                itemCategory = cartItem.category,
                price = cartItem.price,
                quantity = cartItem.quantity
            )
        }

        CypienSDK.commerce.beginCheckout(
            items = items,
            currency = "USD",
            value = cartTotal
        )
    }

    // ... your composable content
}
```

### Purchase / Order Confirmation

```kotlin
@Composable
fun OrderConfirmationScreen(order: Order) {
    LaunchedEffect(order.transactionId) {
        CypienSDK.trackScreen(
            screenPath = "/order-confirmation",
            screenName = "Order Confirmation"
        )

        val items = order.items.map { orderItem ->
            CypienItem(
                itemId = orderItem.sku,
                itemName = orderItem.name,
                itemCategory = orderItem.category,
                price = orderItem.price,
                quantity = orderItem.quantity
            )
        }

        CypienSDK.commerce.purchase(
            transactionId = order.transactionId,
            items = items,
            currency = order.currency,
            value = order.totalAmount
        )
    }

    // ... your composable content
}
```

### Search Screen

```kotlin
@Composable
fun SearchScreen() {
    LaunchedEffect(Unit) {
        CypienSDK.trackScreen(screenPath = "/search", screenName = "Search")
    }

    // ... your composable content
}

// Call this when the user submits a search query
fun onSearch(query: String) {
    CypienSDK.commerce.search(searchTerm = query)
}
```

---

## User Management

### Identify a user

Call `setUserId` as soon as the user authenticates. This links all subsequent events to the identified user profile in the Cypien Dashboard.

```kotlin
CypienSDK.setUserId("user-12345")
```

### Set user properties

Attach additional attributes to the user profile for segmentation and personalization:

```kotlin
CypienSDK.setUserProperties(
    mapOf(
        "plan" to "premium",
        "country" to "US",
        "age_group" to "25-34"
    )
)
```

### Reset user

Call `resetUser` on logout to disassociate future events from the previous user identity:

```kotlin
CypienSDK.resetUser()
```

---

## Debug & Testing

### Enable debug mode

Enable verbose Logcat output during development by setting `debugMode = true` in your config:

```kotlin
CypienSDK.initialize(
    application = this,
    config = CypienConfig(
        workspaceId = "YOUR_WORKSPACE_ID",
        apiKey = "YOUR_API_KEY",
        debugMode = true
    )
)
```

With debug mode active, the SDK logs every tracked event, network request, and personalization result to Logcat under the `CypienSDK` tag.

### Override interest profile

During QA and manual testing, you can force a specific interest profile without waiting for the engine to compute one organically:

```kotlin
// Override the interest profile (debug builds only)
CypienSDK.debug.setInterestOverride(
    mapOf(
        "electronics" to 0.9,
        "fashion" to 0.3
    )
)
```

### Force-emit events

Flush the event queue immediately, bypassing the configured `emitInterval`. Useful when testing event delivery end-to-end:

```kotlin
CypienSDK.debug.forceEmit()
```

> **Warning:** `debug.setInterestOverride` and `debug.forceEmit` are intended exclusively for development and QA. Wrap calls in a `BuildConfig.DEBUG` check to ensure they are never active in production builds.

```kotlin
if (BuildConfig.DEBUG) {
    CypienSDK.debug.setInterestOverride(mapOf("electronics" to 0.9))
}
```

---

## Privacy & GDPR

CypienSDK provides first-class support for privacy regulations including GDPR and CCPA. All opt-out and data deletion APIs take effect immediately and persist across app restarts.

### Opt out

Stop all event collection and personalization for the current user. No data is sent to Cypien servers while the user is opted out:

```kotlin
CypienSDK.optOut()
```

### Opt in

Resume event collection after a previous opt-out:

```kotlin
CypienSDK.optIn()
```

### Delete user data

Submit a deletion request for all data associated with the current user. This is the programmatic equivalent of a GDPR "right to erasure" request:

```kotlin
CypienSDK.deleteUserData()
```

> **Recommended pattern:** Present a privacy consent screen on first launch and call `optOut()` by default until the user grants consent. Once the user accepts, call `optIn()` to begin collection.

---

## Configuration Reference

All parameters are set once via `CypienConfig` at initialization time.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `workspaceId` | `String` | — | **Required.** Your workspace identifier, found in the Cypien Dashboard. |
| `apiKey` | `String` | — | **Required.** Your secret API key, found in the Cypien Dashboard. |
| `debugMode` | `Boolean` | `false` | When `true`, enables verbose Logcat output for all SDK activity. |
| `emitInterval` | `Long` | `15L` | Interval in seconds between automatic event batch flushes. |
| `emitMode` | `EmitMode` | `DUAL` | Controls event routing. `DUAL` sends to both Cypien backend and GA4. `GA4_ONLY` skips the Cypien backend. `BACKEND_ONLY` skips GA4. |

---

## API Reference

### Core

| Method | Signature | Description |
|--------|-----------|-------------|
| `initialize` | `(application: Application, config: CypienConfig)` | Initializes the SDK. Must be called once in `Application.onCreate()`. |
| `trackScreen` | `(screenPath: String, screenName: String)` | Records a screen view event. |
| `setUserId` | `(userId: String)` | Associates all subsequent events with the given user ID. |
| `resetUser` | `()` | Clears the current user identity. Call on logout. |
| `setUserProperties` | `(properties: Map<String, Any>)` | Attaches key-value attributes to the user profile. |
| `optOut` | `()` | Disables all event collection immediately. |
| `optIn` | `()` | Re-enables event collection after a previous opt-out. |
| `deleteUserData` | `()` | Submits a data deletion request for the current user. |

### Commerce (`CypienSDK.commerce`)

| Method | Signature | Description |
|--------|-----------|-------------|
| `viewItem` | `(item: CypienItem, currency: String, value: Double)` | Tracks a product detail page view. |
| `addToCart` | `(item: CypienItem, currency: String, value: Double)` | Tracks an add-to-cart action. |
| `beginCheckout` | `(items: List<CypienItem>, currency: String, value: Double)` | Tracks the start of the checkout flow. |
| `purchase` | `(transactionId: String, items: List<CypienItem>, currency: String, value: Double)` | Tracks a completed purchase. |
| `search` | `(searchTerm: String)` | Tracks a search query. |

### Content (`CypienSDK.content`)

| Method | Signature | Description |
|--------|-----------|-------------|
| `fetchDescription` | `(itemId: String, slotId: String, category: String)` | Requests a personalized text description for the given product and slot. Result is delivered via `onContentReady`. |
| `fetchProductImage` | `(itemId: String, slotId: String, category: String)` | Requests a personalized product image URL for the given product and slot. Result is delivered via `onContentReady`. |
| `fetchImage` | `(slotId: String, category: String)` | Requests a personalized image for a non-product slot (e.g. a banner). Result is delivered via `onContentReady`. |
| `onContentReady` | `(slotId: String, callback: (CypienContentResult) -> Unit)` | Registers a callback that fires when personalized content for `slotId` is available. `CypienContentResult` is either `Description(text: String)` or `Image(url: String)`. |

### Debug (`CypienSDK.debug`)

| Method | Signature | Description |
|--------|-----------|-------------|
| `setInterestOverride` | `(interests: Map<String, Double>)` | Forces a specific interest profile for testing personalization. Values are weights in the range `[0.0, 1.0]`. |
| `forceEmit` | `()` | Immediately flushes the event queue, bypassing `emitInterval`. |

---

## License

MIT © [Cypien AI](https://cypien.ai)
