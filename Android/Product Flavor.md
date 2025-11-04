Merupakan fitur dari Gradle (build system android)  yang memungkinkan membuat beberapa varian aplikasi dari code base yang sama.

## Usecase
Product Flavor biasanya digunakan untuk menghasilakan beberapa versi aplikasi seperti
- versi gratis dan versi berbayar,
- versi untuk staging dan versi untuk production,
- atau versi dengan branding berbeda (misalnya aplikasi untuk klien A dan klien B).

## Contoh
```groovy
flavorDimensions += "environment"

productFlavors {
    create("local") {
        dimension = "environment"
        applicationIdSuffix = ".local"
        versionNameSuffix = "-local"

        buildConfigField("String", "BASE_URL", "\"http://192.168.0.100:3000\"")
        buildConfigField("String", "ENV_TYPE", "\"local\"")
    }

    create("dev") {
        dimension = "environment"
        applicationIdSuffix = ".dev"
        versionNameSuffix = "-dev"

        buildConfigField("String", "BASE_URL", "\"https://api-dev.example.com\"")
        buildConfigField("String", "ENV_TYPE", "\"dev\"")
    }

    create("uat") {
        dimension = "environment"
        applicationIdSuffix = ".uat"
        versionNameSuffix = "-uat"

        buildConfigField("String", "BASE_URL", "\"https://api-uat.example.com\"")
        buildConfigField("String", "ENV_TYPE", "\"uat\"")
    }

    create("prd") {
        dimension = "environment"
        applicationIdSuffix = ".prd"
        versionNameSuffix = "-prd"

        buildConfigField("String", "BASE_URL", "\"https://api.example.com\"")
        buildConfigField("String", "ENV_TYPE", "\"prd\"")
    }
}

```

Dari configurasi di atas kita dapat menghasilkan 4 APK
- `com.example.myapp.local`
- `com.example.myapp.dev`
- `com.example.myapp.uat`
- `com.example.myapp.prd`

