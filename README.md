# Advanced Hive Cache System

## Overview

This document provides a detailed explanation of the advanced caching system I developed for the **Stronger Kiddos** Flutter application. The system is designed to significantly improve performance, reduce Firebase API calls, and enable full offline functionality. It also includes data validation layers, migration handling, and performance optimization.

---

## Code Sample

Below is a real piece of code I wrote from the caching layer:

```dart
class HiveCacheService {
  static HiveCacheService? _instance;
  static HiveCacheService get instance => _instance ??= HiveCacheService._();
  HiveCacheService._();

  late Box _programsBox;
  late Box _metadataBox;

  Future<void> initialize() async {
    _programsBox = await Hive.openBox('programs');
    _metadataBox = await Hive.openBox('metadata');
    await _performVersionMigration();
  }

  Future<T?> get<T>(String key, {required String cacheType}) async {
    final box = _getBox(cacheType);
    final raw = box.get(key);

    if (raw == null) return null;

    final version = raw['version'] as int?;
    if (version == null || version < CacheConfig.currentCacheVersion) {
      await box.delete(key);
      return null;
    }

    final cachedAt = raw['cachedAt'] as int?;
    if (cachedAt == null || !_isValidTTL(cachedAt)) {
      await box.delete(key);
      return null;
    }

    return raw['data'] as T;
  }

  bool _isValidTTL(int timestamp) {
    final now = DateTime.now().millisecondsSinceEpoch;
    final diff = (now - timestamp) / 1000;
    return diff < CacheConfig.defaultTTL.inSeconds;
  }
}
```

---

## What This Code Does

This code is a core part of the **HiveCacheService**. It performs the following:

* Manages data caching locally using Hive.
* Validates cached data before returning it to ensure it is still valid.
* Applies TTL (time-to-live) checks to remove expired cache.
* Ensures compatibility using cache versioning.
* Automatically removes old, corrupted, or invalid data.
* Improves performance by reducing network calls and enabling offline mode.

It effectively serves as an **Offline-First Data Layer** for the application.

---

## Key Concepts Used

### 1. Singleton Pattern

Ensures there is only one instance of the caching service across the app.

### 2. OOP

Encapsulation of cache logic in a dedicated service class.

### 3. Generics & Type Safety

Used in `get<T>` and `put<T>` to guarantee data type consistency.

### 4. Asynchronous Programming

All read/write operations are asynchronous to avoid blocking the UI.

### 5. Multi-Layer Validation System

Data is validated through several layers:

* Existence Check
* Metadata Check
* Version Validation
* TTL Validation
* Corruption Check

### 6. Migration Handling

Ensures structural changes in models do not break existing cached data.

---

## Difficult Problem Solved

One of the main challenges was that Hive returns maps as **Map<dynamic, dynamic>**, which caused type errors.

To fix this, I wrote a deep conversion function that converts all maps and lists into clean `Map<String, dynamic>` objects:

```dart
dynamic _convertHiveData(dynamic raw) {
  if (raw is Map && raw is not Map<String, dynamic>) {
    final result = <String, dynamic>{};
    for (final e in raw.entries) {
      result[e.key.toString()] = _convertHiveData(e.value);
    }
    return result;
  }
  if (raw is List) {
    return raw.map((x) => _convertHiveData(x)).toList();
  }
  return raw;
}
```

This eliminated all corruption issues and made cached data fully stable.

---

## GitHub Link

You can find the full implementation here:

[https://github.com/engAhmedSami/Stronger-Kiddos-main/blob/main/lib/core/cache/hive_cache_service.dart](https://github.com/engAhmedSami/Stronger-Kiddos-main/blob/main/lib/core/cache/hive_cache_service.dart)

---

If you need a shorter version, English version, or a portfolio-ready version, I can generate it instantly.
