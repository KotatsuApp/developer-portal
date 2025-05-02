# Configuration System

The Configuration System in Kotatsu Parsers provides a type-safe mechanism for configuring manga parsers with site-specific options. This system enables parsers to define their configuration requirements and allows the application to provide user interfaces for customizing parser behavior.

For information about search filtering capabilities, see [Search and Filter System](search-and-filter-system.md).

## Configuration Key System
The foundation of the configuration system is the ConfigKey class, which defines different types of configuration options with their associated types and default values.

Each configuration key has:

* A string identifier (key)
* A generic type parameter T for type safety
* A default value (defaultValue)

### Standard Configuration Key Types
| Key Type | Purpose | Data Type | Example |
| --------- | ------------- | ---------- | ----------- |
| Domain | Website domain for HTTP requests | String | `"mangadex.org"` |
| UserAgent | User agent for HTTP requests | String | `"Mozilla/5.0..."` |
| ShowSuspiciousContent | Toggle for content filtering | Boolean | `false` |
| SplitByTranslations | Group chapters by translation | Boolean | `true` |
| PreferredImageServer | Select image server | String? | `"server1"` |

## Key Integration Process
The configuration system follows these integration steps:

1. **Define Configuration Properties**: Parsers define configuration keys as properties
```
override val configKeyDomain = ConfigKey.Domain("webtoons.com")
override val userAgentKey = ConfigKey.UserAgent("nApps (Android 12;; linewebtoon; 3.1.0)")
```
2. **Register Keys**: Parsers register these keys in the `onCreateConfig` method
``` kotlin
override fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>) {
    super.onCreateConfig(keys)
    keys.add(userAgentKey)
}
```
3. **Access Values**: Parsers access configuration values using the `config` property
``` kotlin
if (config[suspiciousContentKey]) {
    url.addQueryParameter("f_sh", "on")
}
```

## Domain Configuration
The domain configuration is mandatory for all parsers and determines which domain to use for HTTP requests. It's accessed via the `domain` property in parsers.

### Simple Domain Configuration

``` kotlin
override val configKeyDomain = ConfigKey.Domain("hitomi.la")
```

### Multiple Domain Options

``` kotlin
override val configKeyDomain = ConfigKey.Domain("example.com", "alt.example.com")
```

The first domain is used as the default value.

### Dynamic Domain Configuration
Some parsers define domains dynamically based on state:

``` kotlin
override val configKeyDomain: ConfigKey.Domain
    get() = ConfigKey.Domain(
        if (isAuthorized) DOMAIN_AUTHORIZED else DOMAIN_UNAUTHORIZED,
        if (isAuthorized) DOMAIN_UNAUTHORIZED else DOMAIN_AUTHORIZED,
    )
```

## Real-World Examples
### WebtoonsParser Example

``` kotlin
internal abstract class WebtoonsParser(
    context: MangaLoaderContext,
    source: MangaParserSource,
) : LegacyMangaParser(context, source) {
    // Domain configuration - static
    override val configKeyDomain = ConfigKey.Domain("webtoons.com")
    
    // User agent configuration
    override val userAgentKey = ConfigKey.UserAgent("nApps (Android 12;; linewebtoon; 3.1.0)")
    
    // Register the user agent key
    override fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>) {
        super.onCreateConfig(keys)
        keys.add(userAgentKey)
    }
    
    // Usage in network requests is handled internally
}
```

### ExHentaiParser Example

``` kotlin
internal class ExHentaiParser(
    context: MangaLoaderContext,
) : LegacyPagedMangaParser(context, MangaParserSource.EXHENTAI, pageSize = 25), 
    MangaParserAuthProvider, Interceptor {
    
    // Domain configuration - dynamic based on auth state
    override val configKeyDomain: ConfigKey.Domain
        get() = ConfigKey.Domain(
            if (isAuthorized) DOMAIN_AUTHORIZED else DOMAIN_UNAUTHORIZED,
            if (isAuthorized) DOMAIN_UNAUTHORIZED else DOMAIN_AUTHORIZED,
        )
    
    // Additional configuration for content filtering
    private val suspiciousContentKey = ConfigKey.ShowSuspiciousContent(false)
    
    // Register configuration keys
    override fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>) {
        super.onCreateConfig(keys)
        keys.add(userAgentKey)
        keys.add(suspiciousContentKey)
    }
    
    // Using the configuration value
    private suspend fun getListPage(...): List<Manga> {
        // ...
        if (config[suspiciousContentKey]) {
            url.addQueryParameter("f_sh", "on")
        }
        // ...
    }
}
```

## Extending the Configuration System
To add new configuration options to a parser:

1. Define a configuration key property with a default value:
``` kotlin
private val myCustomKey = ConfigKey.YourKeyType(defaultValue)
```
2. Register the key in `onCreateConfig`:
``` kotlin
override fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>) {
    super.onCreateConfig(keys)
    keys.add(myCustomKey)
}
```
3. Use the configuration value in your parser:
``` kotlin
val configValue = config[myCustomKey]
// Use configValue to customize parser behavior
```

## Summary
The Configuration System provides a robust mechanism for configuring manga parsers in the Kotatsu application. It offers type safety through generic configuration keys, supports different value types, and enables a clean separation of configuration from parser logic. This system is essential for adapting parsers to different websites and user preferences, providing flexibility while maintaining code quality.