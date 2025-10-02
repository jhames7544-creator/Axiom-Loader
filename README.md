// FILE: Axiom/build.gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '8.9.1' apply false
    id 'com.android.library' version '8.9.1' apply false
         
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

// FILE: Axiom/gradle.properties
#AndroidIDE: enforce UTF-8 & locale for Gradle daemon
#Thu Oct 02 04:51:09 GMT 2025
android.nonTransitiveRClass=true
kotlin.code.style=official
systemProp.user.language=en
systemProp.user.country=US
systemProp.sun.jnu.encoding=UTF-8
systemProp.file.encoding=UTF-8
org.gradle.jvmargs=-Xmx512m -Dfile.encoding\=UTF-8
android.useAndroidX=true


// FILE: Axiom/gradlew.bat
@rem
@rem Copyright 2015 the original author or authors.
@rem
@rem Licensed under the Apache License, Version 2.0 (the "License");
@rem you may not use this file except in compliance with the License.
@rem You may obtain a copy of the License at
@rem
@rem      https://www.apache.org/licenses/LICENSE-2.0
@rem
@rem Unless required by applicable law or agreed to in writing, software
@rem distributed under the License is distributed on an "AS IS" BASIS,
@rem WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
@rem See the License for the specific language governing permissions and
@rem limitations under the License.
@rem
@rem SPDX-License-Identifier: Apache-2.0
@rem

@if "%DEBUG%"=="" @echo off
@rem ##########################################################################
@rem
@rem  Gradle startup script for Windows
@rem
@rem ##########################################################################

@rem Set local scope for the variables with windows NT shell
if "%OS%"=="Windows_NT" setlocal

set DIRNAME=%~dp0
if "%DIRNAME%"=="" set DIRNAME=.
@rem This is normally unused
set APP_BASE_NAME=%~n0
set APP_HOME=%DIRNAME%

@rem Resolve any "." and ".." in APP_HOME to make it shorter.
for %%i in ("%APP_HOME%") do set APP_HOME=%%~fi

@rem Add default JVM options here. You can also use JAVA_OPTS and GRADLE_OPTS to pass JVM options to this script.
set DEFAULT_JVM_OPTS="-Xmx64m" "-Xms64m"

@rem Find java.exe
if defined JAVA_HOME goto findJavaFromJavaHome

set JAVA_EXE=java.exe
%JAVA_EXE% -version >NUL 2>&1
if %ERRORLEVEL% equ 0 goto execute

echo. 1>&2
echo ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH. 1>&2
echo. 1>&2
echo Please set the JAVA_HOME variable in your environment to match the 1>&2
echo location of your Java installation. 1>&2

goto fail

:findJavaFromJavaHome
set JAVA_HOME=%JAVA_HOME:"=%
set JAVA_EXE=%JAVA_HOME%/bin/java.exe

if exist "%JAVA_EXE%" goto execute

echo. 1>&2
echo ERROR: JAVA_HOME is set to an invalid directory: %JAVA_HOME% 1>&2
echo. 1>&2
echo Please set the JAVA_HOME variable in your environment to match the 1>&2
echo location of your Java installation. 1>&2

goto fail

:execute
@rem Setup the command line

set CLASSPATH=%APP_HOME%\gradle\wrapper\gradle-wrapper.jar


@rem Execute Gradle
"%JAVA_EXE%" %DEFAULT_JVM_OPTS% %JAVA_OPTS% %GRADLE_OPTS% "-Dorg.gradle.appname=%APP_BASE_NAME%" -classpath "%CLASSPATH%" org.gradle.wrapper.GradleWrapperMain %*

:end
@rem End local scope for the variables with windows NT shell
if %ERRORLEVEL% equ 0 goto mainEnd

:fail
rem Set variable GRADLE_EXIT_CONSOLE if you need the _script_ return code instead of
rem the _cmd.exe /c_ return code!
set EXIT_CODE=%ERRORLEVEL%
if %EXIT_CODE% equ 0 set EXIT_CODE=1
if not ""=="%GRADLE_EXIT_CONSOLE%" exit %EXIT_CODE%
exit /b %EXIT_CODE%

:mainEnd
if "%OS%"=="Windows_NT" endlocal

:omega


// FILE: Axiom/settings.gradle
pluginManagement {
  repositories {
    gradlePluginPortal()
    google()
    mavenCentral()
  }
}

dependencyResolutionManagement {
  repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
  repositories {
    google()
    mavenCentral()
  }
}

rootProject.name = "Axiom"

include(":app")

// FILE: Axiom/app/build.gradle
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.axiomloader'
    compileSdk 35
    ndkVersion '28.2.13676358'
    
    defaultConfig {
        applicationId "com.axiomloader"
        minSdk 21
        targetSdk 35
        versionCode 1
        versionName "1.0"
        
        vectorDrawables { 
            useSupportLibrary true
        }
        
        externalNativeBuild {
            ndkBuild {
                abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86_64', 'x86'
            }
        }
        
        // Plugin system configuration
        buildConfigField "String", "PLUGIN_DIR", "\"axiom_plugins\""
        buildConfigField "String", "PLUGIN_MANIFEST", "\"plugin.json\""
        buildConfigField "int", "PLUGIN_API_VERSION", "1"
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled false
            debuggable true
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    
    externalNativeBuild {
        ndkBuild {
            path file('src/main/jni/Android.mk')
        }
    }
    
    buildFeatures {
        viewBinding true
        buildConfig true
    }
    
    // Allow reflection for plugin system
    packagingOptions {
        pickFirst '**/libc++_shared.so'
        pickFirst '**/libjsc.so'
    }
}

dependencies {
    implementation("androidx.appcompat:appcompat:1.7.1")
    implementation("androidx.constraintlayout:constraintlayout:2.2.1")
    implementation("com.google.android.material:material:1.12.0")
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    implementation("androidx.cardview:cardview:1.0.0")
    implementation("androidx.fragment:fragment:1.8.5")
    
    // Timber for logging
    implementation 'com.jakewharton.timber:timber:5.0.1'
    
    // File operations and JSON
    implementation 'com.google.code.gson:gson:2.10.1'
    
    // Date/Time utilities
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    
    // Plugin System Dependencies
    implementation 'commons-io:commons-io:2.11.0'
    implementation 'org.apache.commons:commons-compress:1.21'
    implementation 'org.jetbrains:annotations:24.0.1'
    
    // For DEX file handling
    implementation 'androidx.multidex:multidex:2.0.1'
    
    // For ZIP operations (plugin packages)
    implementation 'net.lingala.zip4j:zip4j:2.11.5'
    
    // For reflection and dynamic loading
    implementation 'org.reflections:reflections:0.10.2'
    
    // For HTTP downloads (plugin updates)
    implementation 'com.squareup.okhttp3:okhttp:4.11.0'
    
    // For XML parsing (plugin manifests)
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.15.2'
    implementation 'com.fasterxml.jackson.core:jackson-core:2.15.2'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.2'
}

// FILE: Axiom/app/proguard-rules.pro
# Add project specific ProGuard rules here.
# You can control the set of applied configuration files using the
# proguardFiles setting in build.gradle.

# Keep plugin system classes
-keep class com.axiomloader.modding.** { *; }
-keep interface com.axiomloader.modding.PluginInterface { *; }
-keep class * implements com.axiomloader.modding.PluginInterface { *; }

# Keep plugin manifest classes for Gson
-keepclassmembers class com.axiomloader.modding.PluginManifest { *; }
-keepclassmembers class com.axiomloader.modding.PluginManifest$* { *; }

# Keep native methods
-keepclasseswithmembernames class * {
    native <methods>;
}

# Keep all classes that might be loaded by plugins
-keepnames class * implements com.axiomloader.modding.PluginInterface

# Preserve line numbers for debugging
-keepattributes SourceFile,LineNumberTable
-renamesourcefileattribute SourceFile

# Keep Timber logging
-dontwarn timber.log.**
-keep class timber.log.** { *; }

# Keep Gson classes
-keepattributes Signature
-keepattributes *Annotation*
-dontwarn sun.misc.**
-keep class com.google.gson.** { *; }
-keep class * implements com.google.gson.TypeAdapter
-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer

# Keep Apache Commons
-dontwarn org.apache.commons.**
-keep class org.apache.commons.** { *; }

# Keep plugin data classes
-keep class com.axiomloader.modding.PluginInfo { *; }
-keep enum com.axiomloader.modding.PluginStatus { *; }

# Preserve reflection for dynamic plugin loading
-keepattributes *Annotation*,Signature,InnerClasses,EnclosingMethod

# Keep plugin entry points
-keepclassmembers class * implements com.axiomloader.modding.PluginInterface {
    public void onLoad(android.content.Context);
    public void onUnload();
    public java.lang.String getPluginName();
    public java.lang.String getPluginVersion();
    public java.lang.String getPluginDescription();
    public java.util.Map getPluginInfo();
}

# Keep classes loaded via reflection
-keepclassmembers class * {
    @com.google.gson.annotations.SerializedName <fields>;
}

# AndroidX and Material Design
-keep class com.google.android.material.** { *; }
-dontwarn com.google.android.material.**

# ViewBinding
-keep class * implements androidx.viewbinding.ViewBinding {
    public static * inflate(android.view.LayoutInflater);
    public static * bind(android.view.View);
}

# Keep plugin class loaders
-keep class dalvik.system.DexClassLoader { *; }
-keep class dalvik.system.PathClassLoader { *; }

// FILE: Axiom/app/src/main/AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
        android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
        android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"
        android:minSdkVersion="30" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
    <uses-permission android:name="android.permission.INTERNET" />
  
    <application 
        android:name=".AxiomApplication"
        android:allowBackup="true" 
        android:icon="@mipmap/ic_launcher" 
        android:roundIcon="@mipmap/ic_launcher" 
        android:label="@string/app_name" 
        android:supportsRtl="true" 
        android:theme="@style/AppTheme"
        android:requestLegacyExternalStorage="true"
        android:extractNativeLibs="false"
        android:usesCleartextTraffic="true">
        
        <activity 
            android:name=".MainActivity" 
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name=".ui.LoggingActivity"
            android:exported="false"
            android:label="@string/logging_activity_title"
            android:parentActivityName=".MainActivity"
            android:screenOrientation="portrait" />
            
        <activity
            android:name=".modding.ModdingActivity"
            android:exported="false"
            android:label="@string/modding_activity_title"
            android:parentActivityName=".MainActivity"
            android:screenOrientation="portrait"
            android:launchMode="singleTop" />
            
        <activity
            android:name=".modding.PluginManagerActivity"
            android:exported="false"
            android:label="@string/plugin_manager_title"
            android:parentActivityName=".modding.ModdingActivity"
            android:screenOrientation="portrait" />

        <activity
            android:name=".modding.PluginDevelopmentActivity"
            android:exported="false"
            android:label="@string/plugin_development"
            android:parentActivityName=".modding.ModdingActivity"
            android:screenOrientation="portrait" />
            
        <activity
            android:name=".modding.PluginStoreActivity"
            android:exported="false"
            android:label="@string/plugin_store"
            android:parentActivityName=".modding.ModdingActivity"
            android:screenOrientation="portrait" />
            
        <activity
            android:name=".modding.PluginSettingsActivity"
            android:exported="false"
            android:label="@string/plugin_settings"
            android:parentActivityName=".modding.ModdingActivity"
            android:screenOrientation="portrait" />
        <service
            android:name=".modding.PluginLoaderService"
            android:exported="false"
            android:enabled="true" />
            
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.axiomloader.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        
        <provider
            android:name=".modding.PluginProvider"
            android:authorities="com.axiomloader.plugins"
            android:exported="false"
            android:grantUriPermissions="true" />
            
    </application>
</manifest>


// FILE: Axiom/app/src/main/java/com/axiomloader/AxiomApplication.java
package com.axiomloader;

import android.app.Application;
import android.content.Context;
import android.os.StrictMode;
import androidx.multidex.MultiDex;
import androidx.multidex.MultiDexApplication;
import com.axiomloader.utils.LogUtils;
import com.axiomloader.modding.PluginManager;
import com.axiomloader.modding.PluginSystemInitializer;
import timber.log.Timber;

public class AxiomApplication extends MultiDexApplication {
    
    private PluginManager pluginManager;
    
    @Override
    public void onCreate() {
        super.onCreate();

        // Initialize LogUtils with the application context
        LogUtils.initialize(this);

        // Check for debug build to plant the Timber tree
        if (BuildConfig.DEBUG) {
            Timber.plant(new Timber.DebugTree());
            
            // Enable StrictMode for debugging
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .build());
            
            StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .build());
        }

        // Initialize Plugin System
        initializePluginSystem();

        // Log application lifecycle events
        LogUtils.logInfo("AxiomApplication", "Application onCreate() called with Plugin System");
        LogUtils.logInfo("AxiomApplication", "Plugin Manager initialized: " + (pluginManager != null));
    }
    
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(this);
    }
    
    private void initializePluginSystem() {
        try {
            // Initialize plugin directories and system
            PluginSystemInitializer initializer = new PluginSystemInitializer(this);
            initializer.initialize();
            
            // Initialize plugin manager
            pluginManager = PluginManager.getInstance();
            pluginManager.initialize(this);
            
            // Auto-load enabled plugins
            pluginManager.loadEnabledPlugins();
            
            LogUtils.logInfo("AxiomApplication", "Plugin system initialized successfully");
            
        } catch (Exception e) {
            LogUtils.logError("AxiomApplication", "Failed to initialize plugin system", e);
            Timber.e(e, "Plugin system initialization failed");
        }
    }
    
    public PluginManager getPluginManager() {
        return pluginManager;
    }
    
    @Override
    public void onTerminate() {
        super.onTerminate();
        
        // Shutdown plugin system
        if (pluginManager != null) {
            pluginManager.shutdown();
        }
        
        LogUtils.logInfo("AxiomApplication", "Application terminated");
    }
    
    @Override
    public void onLowMemory() {
        super.onLowMemory();
        
        // Notify plugin system of low memory
        if (pluginManager != null) {
            pluginManager.onLowMemory();
        }
        
        LogUtils.logWarning("AxiomApplication", "Low memory condition detected");
    }
    
    @Override
    public void onTrimMemory(int level) {
        super.onTrimMemory(level);
        
        // Let plugin system handle memory trimming
        if (pluginManager != null) {
            pluginManager.onTrimMemory(level);
        }
        
        LogUtils.logInfo("AxiomApplication", "Memory trimmed at level: " + level);
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/MainActivity.java
package com.axiomloader;

import android.content.Intent;
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import com.axiomloader.databinding.ActivityMainBinding;
import com.axiomloader.ui.LoggingActivity;
import com.axiomloader.modding.ModdingActivity;
import timber.log.Timber;

public class MainActivity extends AppCompatActivity {
    
    private ActivityMainBinding binding;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        
        // Set up the logs button
        binding.logsButton.setOnClickListener(v -> {
            Timber.i("Opening Logging Activity from MainActivity");
            Intent intent = new Intent(this, LoggingActivity.class);
            startActivity(intent);
        });
        
        // Set up the modding button
        binding.moddingButton.setOnClickListener(v -> {
            Timber.i("Opening Modding Activity from MainActivity");
            Intent intent = new Intent(this, ModdingActivity.class);
            startActivity(intent);
        });
        
        // Log app startup
        Timber.i("MainActivity created successfully");
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        this.binding = null;
        Timber.d("MainActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/ModdingActivity.java
package com.axiomloader.modding;

import android.content.Intent;
import android.os.Bundle;
import android.view.MenuItem;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityModdingBinding;
import com.axiomloader.utils.LogUtils;
import timber.log.Timber;

public class ModdingActivity extends AppCompatActivity {
    
    private ActivityModdingBinding binding;
    private PluginManager pluginManager;
    private static final String TAG = "ModdingActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityModdingBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.modding_activity_title);
        }
        
        pluginManager = PluginManager.getInstance();
        
        setupCards();
        updatePluginStatus();
        
        LogUtils.logInfo(TAG, "ModdingActivity created");
    }
    
    private void setupCards() {
        // Plugin Manager Card
        binding.cardPluginManager.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Opening Plugin Manager");
            Intent intent = new Intent(this, PluginManagerActivity.class);
            startActivity(intent);
        });
        
        // Plugin Development Card
        binding.cardPluginDevelopment.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Opening Plugin Development");
            Intent intent = new Intent(this, PluginDevelopmentActivity.class);
            startActivity(intent);
        });
        
        // Plugin Store Card
        binding.cardPluginStore.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Opening Plugin Store");
            Intent intent = new Intent(this, PluginStoreActivity.class);
            startActivity(intent);
        });
        
        // Plugin Settings Card
        binding.cardPluginSettings.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Opening Plugin Settings");
            Intent intent = new Intent(this, PluginSettingsActivity.class);
            startActivity(intent);
        });
        
        // System Toggle Switch
        binding.switchPluginSystem.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) { // Only respond to user clicks
                togglePluginSystem(isChecked);
            }
        });
        
        // Refresh button
        binding.btnRefreshPlugins.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Refreshing plugin status");
            refreshPluginStatus();
        });
        
        // Scan plugins button
        binding.btnScanPlugins.setOnClickListener(v -> {
            LogUtils.logInfo(TAG, "Scanning for plugins");
            scanForPlugins();
        });
    }
    
    private void updatePluginStatus() {
        if (pluginManager == null) {
            pluginManager = PluginManager.getInstance();
        }
        
        new Thread(() -> {
            try {
                final boolean systemEnabled = pluginManager.isPluginSystemEnabled();
                final int totalPlugins = pluginManager.getTotalPluginCount();
                final int enabledPlugins = pluginManager.getEnabledPluginCount();
                final int loadedPlugins = pluginManager.getLoadedPluginCount();
                final String pluginDir = pluginManager.getPluginDirectory().getAbsolutePath();
                
                runOnUiThread(() -> {
                    binding.switchPluginSystem.setChecked(systemEnabled);
                    binding.tvSystemStatus.setText(systemEnabled ? 
                        getString(R.string.plugin_system_enabled) : 
                        getString(R.string.plugin_system_disabled));
                    
                    binding.tvTotalPlugins.setText("Total: " + totalPlugins);
                    binding.tvEnabledPlugins.setText("Enabled: " + enabledPlugins);
                    binding.tvLoadedPlugins.setText("Loaded: " + loadedPlugins);
                    binding.tvPluginDirectory.setText("Directory: " + pluginDir);
                    
                    // Update card states
                    updateCardStates(systemEnabled);
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Error updating plugin status", e);
                runOnUiThread(() -> {
                    Toast.makeText(this, "Error updating plugin status", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void updateCardStates(boolean systemEnabled) {
        float alpha = systemEnabled ? 1.0f : 0.5f;
        boolean enabled = systemEnabled;
        
        binding.cardPluginManager.setAlpha(alpha);
        binding.cardPluginManager.setEnabled(enabled);
        
        binding.cardPluginDevelopment.setAlpha(alpha);
        binding.cardPluginDevelopment.setEnabled(enabled);
        
        binding.cardPluginStore.setAlpha(alpha);
        binding.cardPluginStore.setEnabled(enabled);
        
        binding.cardPluginSettings.setAlpha(alpha);
        binding.cardPluginSettings.setEnabled(enabled);
    }
    
    private void togglePluginSystem(boolean enable) {
        new Thread(() -> {
            try {
                if (enable) {
                    pluginManager.enablePluginSystem();
                    LogUtils.logInfo(TAG, "Plugin system enabled");
                } else {
                    pluginManager.disablePluginSystem();
                    LogUtils.logInfo(TAG, "Plugin system disabled");
                }
                
                runOnUiThread(() -> {
                    Toast.makeText(this, enable ? 
                        getString(R.string.plugin_system_enabled) : 
                        getString(R.string.plugin_system_disabled), 
                        Toast.LENGTH_SHORT).show();
                    
                    updatePluginStatus();
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Error toggling plugin system", e);
                runOnUiThread(() -> {
                    binding.switchPluginSystem.setChecked(!enable);
                    Toast.makeText(this, "Error toggling plugin system", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void refreshPluginStatus() {
        binding.btnRefreshPlugins.setEnabled(false);
        binding.btnRefreshPlugins.setText("Refreshing...");
        
        new Thread(() -> {
            try {
                pluginManager.refreshPlugins();
                Thread.sleep(500); // Brief delay for user feedback
                
                runOnUiThread(() -> {
                    updatePluginStatus();
                    binding.btnRefreshPlugins.setEnabled(true);
                    binding.btnRefreshPlugins.setText("Refresh Status");
                    Toast.makeText(this, "Plugin status refreshed", Toast.LENGTH_SHORT).show();
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Error refreshing plugins", e);
                runOnUiThread(() -> {
                    binding.btnRefreshPlugins.setEnabled(true);
                    binding.btnRefreshPlugins.setText("Refresh Status");
                    Toast.makeText(this, "Error refreshing plugins", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void scanForPlugins() {
        binding.btnScanPlugins.setEnabled(false);
        binding.btnScanPlugins.setText("Scanning...");
        
        new Thread(() -> {
            try {
                int foundPlugins = pluginManager.scanForNewPlugins();
                
                runOnUiThread(() -> {
                    binding.btnScanPlugins.setEnabled(true);
                    binding.btnScanPlugins.setText("Scan for Plugins");
                    
                    String message = foundPlugins > 0 ? 
                        "Found " + foundPlugins + " new plugins" : 
                        "No new plugins found";
                    
                    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                    
                    if (foundPlugins > 0) {
                        updatePluginStatus();
                    }
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Error scanning for plugins", e);
                runOnUiThread(() -> {
                    binding.btnScanPlugins.setEnabled(true);
                    binding.btnScanPlugins.setText("Scan for Plugins");
                    Toast.makeText(this, "Error scanning for plugins", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        updatePluginStatus();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "ModdingActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginAdapter.java
package com.axiomloader.modding;

import android.content.Context;
import android.graphics.Color;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.Switch;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.axiomloader.R;
import com.google.android.material.card.MaterialCardView;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;

public class PluginAdapter extends RecyclerView.Adapter<PluginAdapter.PluginViewHolder> {
    
    private final Context context;
    private final List<PluginInfo> plugins;
    private final PluginActionListener actionListener;
    private final SimpleDateFormat dateFormat;
    private final PluginManager pluginManager;
    
    // Status colors
    private static final int COLOR_LOADED = Color.parseColor("#4CAF50");    // Green
    private static final int COLOR_ENABLED = Color.parseColor("#2196F3");   // Blue  
    private static final int COLOR_DISABLED = Color.parseColor("#757575");  // Gray
    private static final int COLOR_ERROR = Color.parseColor("#F44336");     // Red
    
    public interface PluginActionListener {
        void onPluginClick(PluginInfo plugin);
        void onEnableToggle(PluginInfo plugin, boolean enabled);
        void onPluginSettings(PluginInfo plugin);
        void onPluginUninstall(PluginInfo plugin);
    }
    
    public PluginAdapter(Context context, List<PluginInfo> plugins, PluginActionListener actionListener) {
        this.context = context;
        this.plugins = plugins;
        this.actionListener = actionListener;
        this.dateFormat = new SimpleDateFormat("MMM dd, yyyy", Locale.getDefault());
        this.pluginManager = PluginManager.getInstance();
    }
    
    @NonNull
    @Override
    public PluginViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_plugin_entry, parent, false);
        return new PluginViewHolder(view);
    }
    
    @Override
    public void onBindViewHolder(@NonNull PluginViewHolder holder, int position) {
        PluginInfo plugin = plugins.get(position);
        holder.bind(plugin);
    }
    
    @Override
    public int getItemCount() {
        return plugins.size();
    }
    
    class PluginViewHolder extends RecyclerView.ViewHolder {
        
        private final MaterialCardView cardView;
        private final TextView tvPluginName;
        private final TextView tvPluginVersion;
        private final TextView tvPluginAuthor;
        private final TextView tvPluginDescription;
        private final TextView tvPluginStatus;
        private final TextView tvPluginSize;
        private final TextView tvLastModified;
        private final Switch switchEnabled;
        private final ImageView ivPluginIcon;
        private final View statusIndicator;
        private final View btnSettings;
        private final View btnUninstall;
        
        public PluginViewHolder(@NonNull View itemView) {
            super(itemView);
            
            cardView = itemView.findViewById(R.id.card_plugin);
            tvPluginName = itemView.findViewById(R.id.tv_plugin_name);
            tvPluginVersion = itemView.findViewById(R.id.tv_plugin_version);
            tvPluginAuthor = itemView.findViewById(R.id.tv_plugin_author);
            tvPluginDescription = itemView.findViewById(R.id.tv_plugin_description);
            tvPluginStatus = itemView.findViewById(R.id.tv_plugin_status);
            tvPluginSize = itemView.findViewById(R.id.tv_plugin_size);
            tvLastModified = itemView.findViewById(R.id.tv_last_modified);
            switchEnabled = itemView.findViewById(R.id.switch_enabled);
            ivPluginIcon = itemView.findViewById(R.id.iv_plugin_icon);
            statusIndicator = itemView.findViewById(R.id.status_indicator);
            btnSettings = itemView.findViewById(R.id.btn_settings);
            btnUninstall = itemView.findViewById(R.id.btn_uninstall);
        }
        
        public void bind(PluginInfo plugin) {
            // Basic plugin info
            tvPluginName.setText(plugin.getDisplayName());
            tvPluginVersion.setText("v" + plugin.version);
            tvPluginAuthor.setText("by " + (plugin.author != null ? plugin.author : "Unknown"));
            
            // Description
            if (plugin.description != null && !plugin.description.isEmpty()) {
                tvPluginDescription.setText(plugin.description);
                tvPluginDescription.setVisibility(View.VISIBLE);
            } else {
                tvPluginDescription.setVisibility(View.GONE);
            }
            
            // Plugin size
            if (plugin.size > 0) {
                tvPluginSize.setText(formatFileSize(plugin.size));
            } else {
                tvPluginSize.setText("Unknown size");
            }
            
            // Last modified date
            if (plugin.lastModified > 0) {
                tvLastModified.setText("Modified: " + dateFormat.format(new Date(plugin.lastModified)));
            } else {
                tvLastModified.setText("Modified: Unknown");
            }
            
            // Status and colors
            updatePluginStatus(plugin);
            
            // FIXED: Enable/disable switch - properly check THIS specific plugin
            boolean isEnabled = pluginManager != null && pluginManager.isPluginEnabled(plugin.id);
            switchEnabled.setOnCheckedChangeListener(null); // Clear listener first
            switchEnabled.setChecked(isEnabled);
            switchEnabled.setOnCheckedChangeListener((buttonView, isChecked) -> {
                if (buttonView.isPressed()) { // Only respond to user interactions
                    actionListener.onEnableToggle(plugin, isChecked);
                }
            });
            
            // Plugin icon (placeholder for now)
            setPluginIcon(plugin);
            
            // Click listeners
            cardView.setOnClickListener(v -> actionListener.onPluginClick(plugin));
            
            btnSettings.setOnClickListener(v -> actionListener.onPluginSettings(plugin));
            
            btnUninstall.setOnClickListener(v -> actionListener.onPluginUninstall(plugin));
            
            // Card styling based on status
            updateCardStyling(plugin);
        }
        
        private void updatePluginStatus(PluginInfo plugin) {
            String statusText;
            int statusColor;
            
            switch (plugin.status) {
                case LOADED:
                    statusText = "Loaded";
                    statusColor = COLOR_LOADED;
                    break;
                case LOADING:
                    statusText = "Loading...";
                    statusColor = COLOR_ENABLED;
                    break;
                case ERROR:
                    statusText = "Error";
                    statusColor = COLOR_ERROR;
                    break;
                case DISABLED:
                    statusText = "Disabled";
                    statusColor = COLOR_DISABLED;
                    break;
                case DISCOVERED:
                    statusText = "Available";
                    statusColor = COLOR_ENABLED;
                    break;
                default:
                    statusText = "Unknown";
                    statusColor = COLOR_DISABLED;
                    break;
            }
            
            tvPluginStatus.setText(statusText);
            tvPluginStatus.setTextColor(statusColor);
            statusIndicator.setBackgroundColor(statusColor);
        }
        
        private void setPluginIcon(PluginInfo plugin) {
            // For now, use a default icon based on plugin type/category
            // In a real implementation, this could load custom icons from plugin assets
            
            if (plugin.hasErrors()) {
                ivPluginIcon.setImageResource(android.R.drawable.ic_dialog_alert);
            } else if (plugin.isLoaded()) {
                ivPluginIcon.setImageResource(android.R.drawable.ic_menu_manage);
            } else {
                ivPluginIcon.setImageResource(android.R.drawable.ic_menu_preferences);
            }
        }
        
        private void updateCardStyling(PluginInfo plugin) {
            if (plugin.hasErrors()) {
                // Error state - red tint
                cardView.setCardBackgroundColor(Color.parseColor("#FFEBEE"));
                cardView.setStrokeColor(COLOR_ERROR);
                cardView.setStrokeWidth(2);
            } else if (plugin.isLoaded()) {
                // Loaded state - green tint
                cardView.setCardBackgroundColor(Color.parseColor("#E8F5E8"));
                cardView.setStrokeColor(COLOR_LOADED);
                cardView.setStrokeWidth(2);
            } else {
                // Default state
                cardView.setCardBackgroundColor(Color.WHITE);
                cardView.setStrokeColor(Color.TRANSPARENT);
                cardView.setStrokeWidth(0);
            }
        }
        
        private String formatFileSize(long sizeBytes) {
            if (sizeBytes < 1024) {
                return sizeBytes + " B";
            } else if (sizeBytes < 1024 * 1024) {
                return String.format(Locale.getDefault(), "%.1f KB", sizeBytes / 1024.0);
            } else {
                return String.format(Locale.getDefault(), "%.1f MB", sizeBytes / (1024.0 * 1024.0));
            }
        }
    }
    
    /**
     * Update a specific plugin in the list
     */
    public void updatePlugin(PluginInfo plugin) {
        for (int i = 0; i < plugins.size(); i++) {
            if (plugins.get(i).id.equals(plugin.id)) {
                plugins.set(i, plugin);
                notifyItemChanged(i);
                break;
            }
        }
    }
    
    /**
     * Add a new plugin to the list
     */
    public void addPlugin(PluginInfo plugin) {
        plugins.add(plugin);
        notifyItemInserted(plugins.size() - 1);
    }
    
    /**
     * Remove a plugin from the list
     */
    public void removePlugin(String pluginId) {
        for (int i = 0; i < plugins.size(); i++) {
            if (plugins.get(i).id.equals(pluginId)) {
                plugins.remove(i);
                notifyItemRemoved(i);
                break;
            }
        }
    }
    
    /**
     * Filter plugins by status
     */
    public void filterByStatus(PluginStatus status) {
        // Implementation for filtering - would require additional logic
        notifyDataSetChanged();
    }
    
    /**
     * Sort plugins by name, date, etc.
     */
    public void sortPlugins(SortType sortType) {
        switch (sortType) {
            case NAME:
                plugins.sort((p1, p2) -> p1.getDisplayName().compareToIgnoreCase(p2.getDisplayName()));
                break;
            case DATE:
                plugins.sort((p1, p2) -> Long.compare(p2.lastModified, p1.lastModified));
                break;
            case SIZE:
                plugins.sort((p1, p2) -> Long.compare(p2.size, p1.size));
                break;
            case STATUS:
                plugins.sort((p1, p2) -> p1.status.compareTo(p2.status));
                break;
        }
        notifyDataSetChanged();
    }
    
    public enum SortType {
        NAME, DATE, SIZE, STATUS
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginDevelopmentActivity.java
package com.axiomloader.modding;

import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.OpenableColumns;
import android.view.MenuItem;
import android.widget.EditText;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityPluginDevelopmentBinding;
import com.axiomloader.utils.LogUtils;
import java.io.File;
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

public class PluginDevelopmentActivity extends AppCompatActivity {
    
    private ActivityPluginDevelopmentBinding binding;
    private static final String TAG = "PluginDevelopmentActivity";
    private SimpleDateFormat dateFormat;
    
    private static final int REQUEST_CODE_IMPORT_FILE = 1001;
    private static final int REQUEST_CODE_IMPORT_DIRECTORY = 1002;
    private static final int REQUEST_CODE_IMPORT_TEMPLATE = 1003;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityPluginDevelopmentBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.plugin_development);
        }
        
        dateFormat = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault());
        setupCards();
        
        LogUtils.logInfo(TAG, "PluginDevelopmentActivity created");
    }
    
    private void setupCards() {
        binding.cardCreateTemplate.setOnClickListener(v -> showTemplateOptions());
        binding.cardTestPlugin.setOnClickListener(v -> showTestingOptions());
        binding.cardDocumentation.setOnClickListener(v -> showDocumentation());
        binding.cardDevTools.setOnClickListener(v -> showDeveloperTools());
        binding.cardHotReload.setOnClickListener(v -> toggleHotReload());
        binding.cardConsole.setOnClickListener(v -> openPluginConsole());
        
        // Setup quick action buttons
        if (binding.btnNewProject != null) {
            binding.btnNewProject.setOnClickListener(v -> showTemplateOptions());
        }
        if (binding.btnImportProject != null) {
            binding.btnImportProject.setOnClickListener(v -> importPlugin());
        }
    }
    
    private void showTemplateOptions() {
        String[] templates = {
            "Basic Plugin Template",
            "UI Modification Plugin", 
            "Log Analyzer Plugin",
            "Network Monitor Plugin",
            "Theme Plugin",
            "Advanced Plugin Template"
        };
        
        new AlertDialog.Builder(this)
            .setTitle("Create Plugin Template")
            .setItems(templates, (dialog, which) -> createTemplate(which))
            .show();
    }
    
    private void createTemplate(int templateType) {
        new Thread(() -> {
            try {
                String templateName = getTemplateName(templateType);
                File projectDir = generateTemplate(templateType, templateName);
                
                runOnUiThread(() -> {
                    if (projectDir != null && projectDir.exists()) {
                        Toast.makeText(this, "Template created: " + templateName + "\n" + projectDir.getAbsolutePath(), Toast.LENGTH_LONG).show();
                        offerToShareTemplate(projectDir);
                    } else {
                        Toast.makeText(this, "Failed to create template", Toast.LENGTH_SHORT).show();
                    }
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to create template", e);
                runOnUiThread(() -> Toast.makeText(this, "Template creation failed: " + e.getMessage(), Toast.LENGTH_SHORT).show());
            }
        }).start();
    }
    
    private String getTemplateName(int templateType) {
        switch (templateType) {
            case 0: return "BasicPlugin";
            case 1: return "UIModificationPlugin";
            case 2: return "LogAnalyzerPlugin";
            case 3: return "NetworkMonitorPlugin";
            case 4: return "ThemePlugin";
            case 5: return "AdvancedPlugin";
            default: return "CustomPlugin";
        }
    }
    
    private File generateTemplate(int templateType, String templateName) throws IOException {
        File templateDir = new File(getExternalFilesDir(null), "axiom_plugins/development");
        if (!templateDir.exists()) {
            templateDir.mkdirs();
        }
        
        String timestamp = dateFormat.format(new Date());
        File projectDir = new File(templateDir, templateName + "_" + timestamp);
        if (!projectDir.exists()) {
            projectDir.mkdirs();
        }
        
        // Generate Java source file
        File javaFile = new File(projectDir, templateName + ".java");
        try (FileWriter writer = new FileWriter(javaFile)) {
            writer.write(generateJavaTemplate(templateType, templateName));
        }
        
        // Generate manifest file
        File manifestFile = new File(projectDir, "plugin.json");
        try (FileWriter writer = new FileWriter(manifestFile)) {
            writer.write(generateManifestTemplate(templateType, templateName));
        }
        
        // Generate README
        File readmeFile = new File(projectDir, "README.md");
        try (FileWriter writer = new FileWriter(readmeFile)) {
            writer.write(generateReadmeTemplate(templateType, templateName));
        }
        
        // Generate build script
        File buildScript = new File(projectDir, "build.sh");
        try (FileWriter writer = new FileWriter(buildScript)) {
            writer.write(generateBuildScript(templateName));
        }
        buildScript.setExecutable(true);
        
        LogUtils.logInfo(TAG, "Generated template: " + templateName);
        return projectDir;
    }
    
    private String generateJavaTemplate(int templateType, String templateName) {
        StringBuilder sb = new StringBuilder();
        
        sb.append("package com.axiom.plugins;\n\n");
        sb.append("import android.content.Context;\n");
        sb.append("import com.axiomloader.modding.PluginInterface;\n");
        sb.append("import java.util.HashMap;\n");
        sb.append("import java.util.Map;\n\n");
        
        sb.append("/**\n");
        sb.append(" * ").append(templateName).append(" - Generated plugin template\n");
        sb.append(" * Auto-generated by Axiom Plugin Development System\n");
        sb.append(" */\n");
        sb.append("public class ").append(templateName).append(" implements PluginInterface {\n\n");
        
        sb.append("    private Context context;\n");
        sb.append("    private boolean isInitialized = false;\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public void onLoad(Context context) {\n");
        sb.append("        this.context = context;\n");
        sb.append("        initialize();\n");
        sb.append("        isInitialized = true;\n");
        sb.append("    }\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public void onUnload() {\n");
        sb.append("        cleanup();\n");
        sb.append("        isInitialized = false;\n");
        sb.append("    }\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public Map<String, Object> getPluginInfo() {\n");
        sb.append("        Map<String, Object> info = new HashMap<>();\n");
        sb.append("        info.put(\"name\", \"").append(templateName).append("\");\n");
        sb.append("        info.put(\"version\", \"1.0.0\");\n");
        sb.append("        info.put(\"author\", \"Your Name\");\n");
        sb.append("        info.put(\"initialized\", isInitialized);\n");
        sb.append("        return info;\n");
        sb.append("    }\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public String getPluginName() {\n");
        sb.append("        return \"").append(templateName).append("\";\n");
        sb.append("    }\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public String getPluginVersion() {\n");
        sb.append("        return \"1.0.0\";\n");
        sb.append("    }\n\n");
        
        sb.append("    @Override\n");
        sb.append("    public String getPluginDescription() {\n");
        sb.append("        return \"").append(getTemplateDescription(templateType)).append("\";\n");
        sb.append("    }\n\n");
        
        sb.append(generateTemplateSpecificMethods(templateType));
        
        sb.append("    private void initialize() {\n");
        sb.append("        // Initialize plugin resources\n");
        sb.append("    }\n\n");
        
        sb.append("    private void cleanup() {\n");
        sb.append("        // Clean up plugin resources\n");
        sb.append("    }\n");
        
        sb.append("}\n");
        
        return sb.toString();
    }
    
    private String getTemplateDescription(int templateType) {
        switch (templateType) {
            case 0: return "A basic plugin template with minimal functionality";
            case 1: return "Plugin for modifying user interface elements";
            case 2: return "Plugin for analyzing and processing log data";
            case 3: return "Plugin for monitoring network activity";
            case 4: return "Plugin for customizing app themes and appearance";
            case 5: return "Advanced plugin template with extended capabilities";
            default: return "Custom plugin template";
        }
    }
    
    private String generateTemplateSpecificMethods(int templateType) {
        switch (templateType) {
            case 1:
                return "    public void modifyUI() {\n" +
                       "        // Add UI modification code here\n" +
                       "    }\n\n";
            case 2:
                return "    public void analyzeLog(String logEntry) {\n" +
                       "        // Add log analysis code here\n" +
                       "    }\n\n" +
                       "    public void generateReport() {\n" +
                       "        // Generate analysis report\n" +
                       "    }\n\n";
            case 3:
                return "    public void startNetworkMonitoring() {\n" +
                       "        // Start monitoring network activity\n" +
                       "    }\n\n" +
                       "    public void stopNetworkMonitoring() {\n" +
                       "        // Stop monitoring network activity\n" +
                       "    }\n\n";
            case 4:
                return "    public void applyTheme() {\n" +
                       "        // Apply custom theme\n" +
                       "    }\n\n" +
                       "    public void resetTheme() {\n" +
                       "        // Reset to default theme\n" +
                       "    }\n\n";
            default:
                return "    public void executeCustomAction() {\n" +
                       "        // Add your custom functionality here\n" +
                       "    }\n\n";
        }
    }
    
    private String generateManifestTemplate(int templateType, String templateName) {
        return "{\n" +
               "  \"id\": \"com.axiom.plugins." + templateName.toLowerCase() + "\",\n" +
               "  \"name\": \"" + templateName + "\",\n" +
               "  \"version\": \"1.0.0\",\n" +
               "  \"author\": \"Your Name\",\n" +
               "  \"description\": \"" + getTemplateDescription(templateType) + "\",\n" +
               "  \"main_class\": \"com.axiom.plugins." + templateName + "\",\n" +
               "  \"api_version\": 1,\n" +
               "  \"min_android_version\": 21,\n" +
               "  \"permissions\": [\n" +
               "    " + getTemplatePermissions(templateType) + "\n" +
               "  ],\n" +
               "  \"features\": [\n" +
               "    \"" + getTemplateFeatures(templateType) + "\"\n" +
               "  ],\n" +
               "  \"category\": \"" + getTemplateCategory(templateType) + "\",\n" +
               "  \"tags\": [\"template\", \"generated\"]\n" +
               "}";
    }
    
    private String getTemplatePermissions(int templateType) {
        switch (templateType) {
            case 1: return "\"axiom.permission.MODIFY_UI\"";
            case 2: return "\"axiom.permission.ACCESS_LOGS\"";
            case 3: return "\"axiom.permission.NETWORK_ACCESS\"";
            case 4: return "\"axiom.permission.MODIFY_UI\"";
            default: return "\"axiom.permission.BASIC_ACCESS\"";
        }
    }
    
    private String getTemplateFeatures(int templateType) {
        switch (templateType) {
            case 1: return "ui_modification";
            case 2: return "log_analysis";
            case 3: return "network_monitoring";
            case 4: return "theme_support";
            default: return "basic_plugin";
        }
    }
    
    private String getTemplateCategory(int templateType) {
        switch (templateType) {
            case 1: return "ui";
            case 2: return "analysis";
            case 3: return "monitoring";
            case 4: return "customization";
            default: return "utility";
        }
    }
    
    private String generateReadmeTemplate(int templateType, String templateName) {
        return "# " + templateName + "\n\n" +
               getTemplateDescription(templateType) + "\n\n" +
               "## Building\n\n" +
               "```bash\n" +
               "# Compile Java to class files\n" +
               "javac -cp /path/to/android.jar " + templateName + ".java\n\n" +
               "# Convert to DEX format\n" +
               "d8 --output . " + templateName + ".class\n" +
               "```\n\n" +
               "## Installation\n\n" +
               "1. Copy the DEX file and plugin.json to the plugin directory\n" +
               "2. Enable the plugin in Axiom Plugin Manager\n\n" +
               "## Development\n\n" +
               "Modify the methods to implement your desired functionality.\n\n" +
               "## API Reference\n\n" +
               "See the main documentation for available APIs and methods.\n";
    }
    
    private String generateBuildScript(String templateName) {
        return "#!/bin/bash\n\n" +
               "# Build script for " + templateName + "\n\n" +
               "echo \"Building " + templateName + "...\"\n\n" +
               "ANDROID_HOME=${ANDROID_HOME:-$HOME/Android/Sdk}\n" +
               "ANDROID_JAR=$ANDROID_HOME/platforms/android-33/android.jar\n\n" +
               "if [ ! -f \"$ANDROID_JAR\" ]; then\n" +
               "    echo \"Error: Android SDK not found\"\n" +
               "    exit 1\n" +
               "fi\n\n" +
               "javac -cp \"$ANDROID_JAR\" -d . " + templateName + ".java\n\n" +
               "d8 --lib \"$ANDROID_JAR\" --output " + templateName + ".dex " + templateName + ".class\n\n" +
               "echo \"Build complete: " + templateName + ".dex\"\n";
    }
    
    private void offerToShareTemplate(File projectDir) {
        new AlertDialog.Builder(this)
            .setTitle("Template Created")
            .setMessage("Template created at:\n" + projectDir.getAbsolutePath() + "\n\nWould you like to open the folder?")
            .setPositiveButton("Open Folder", (dialog, which) -> openFolder(projectDir))
            .setNegativeButton("Close", null)
            .show();
    }
    
    private void openFolder(File folder) {
        try {
            Intent intent = new Intent(Intent.ACTION_VIEW);
            Uri uri = FileProvider.getUriForFile(this, "com.axiomloader.fileprovider", folder);
            intent.setDataAndType(uri, "resource/folder");
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            startActivity(Intent.createChooser(intent, "Open Folder"));
        } catch (Exception e) {
            Toast.makeText(this, "Cannot open folder: " + e.getMessage(), Toast.LENGTH_SHORT).show();
        }
    }
    
    // CONTINUE TO PART 2 FOR REMAINING METHODS
    
   // PART 2 - CONTINUATION OF PluginDevelopmentActivity.java
// Add this code after Part 1

    private void showTestingOptions() {
        String[] options = {
            "Test Current Plugin",
            "Load Test Plugin",
            "Run Plugin Validation", 
            "Performance Test",
            "Memory Usage Test"
        };
        
        new AlertDialog.Builder(this)
            .setTitle("Plugin Testing")
            .setItems(options, (dialog, which) -> executeTest(which))
            .show();
    }
    
    private void executeTest(int testType) {
        String testName = getTestName(testType);
        Toast.makeText(this, "Executing " + testName + "...", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Running test: " + testName);
    }
    
    private String getTestName(int testType) {
        switch (testType) {
            case 0: return "Current Plugin Test";
            case 1: return "Load Test Plugin";
            case 2: return "Plugin Validation";
            case 3: return "Performance Test";
            case 4: return "Memory Usage Test";
            default: return "Unknown Test";
        }
    }
    
    private void showDocumentation() {
        String documentation = "# Plugin Development Documentation\n\n" +
                             "## Getting Started\n" +
                             "1. Create a plugin template\n" +
                             "2. Implement PluginInterface methods\n" +
                             "3. Create plugin.json manifest\n" +
                             "4. Build and test your plugin\n\n" +
                             "## Required Methods\n" +
                             "- onLoad(Context): Initialize plugin\n" +
                             "- onUnload(): Clean up resources\n" +
                             "- getPluginInfo(): Return metadata\n\n" +
                             "## Best Practices\n" +
                             "- Always clean up in onUnload()\n" +
                             "- Handle exceptions gracefully\n" +
                             "- Use appropriate permissions\n" +
                             "- Test thoroughly before distribution\n";
        
        new AlertDialog.Builder(this)
            .setTitle("Plugin Development Guide")
            .setMessage(documentation)
            .setPositiveButton("Close", null)
            .show();
    }
    
    private void showDeveloperTools() {
        String[] tools = {
            "Plugin Compiler",
            "Manifest Validator", 
            "DEX Inspector",
            "Log Viewer",
            "Performance Monitor"
        };
        
        new AlertDialog.Builder(this)
            .setTitle("Developer Tools")
            .setItems(tools, (dialog, which) -> launchTool(which))
            .show();
    }
    
    private void launchTool(int toolIndex) {
        String toolName = getToolName(toolIndex);
        Toast.makeText(this, "Launching " + toolName + "...", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Launching developer tool: " + toolName);
    }
    
    private String getToolName(int toolIndex) {
        switch (toolIndex) {
            case 0: return "Plugin Compiler";
            case 1: return "Manifest Validator";
            case 2: return "DEX Inspector";
            case 3: return "Log Viewer";
            case 4: return "Performance Monitor";
            default: return "Unknown Tool";
        }
    }
    
    private void toggleHotReload() {
        boolean hotReloadEnabled = getSharedPreferences("axiom_plugin_settings", MODE_PRIVATE)
            .getBoolean("hot_reload", false);
        hotReloadEnabled = !hotReloadEnabled;
        getSharedPreferences("axiom_plugin_settings", MODE_PRIVATE)
            .edit()
            .putBoolean("hot_reload", hotReloadEnabled)
            .apply();
        
        Toast.makeText(this, hotReloadEnabled ? "Hot Reload Enabled" : "Hot Reload Disabled", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Hot reload toggled: " + hotReloadEnabled);
    }
    
    private void openPluginConsole() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Plugin Console");
        
        ScrollView scrollView = new ScrollView(this);
        TextView consoleText = new TextView(this);
        consoleText.setText(generateConsoleOutput());
        consoleText.setTextSize(12);
        consoleText.setTypeface(android.graphics.Typeface.MONOSPACE);
        consoleText.setPadding(16, 16, 16, 16);
        scrollView.addView(consoleText);
        
        builder.setView(scrollView);
        builder.setPositiveButton("Close", null);
        builder.setNeutralButton("Clear", (dialog, which) -> {
            Toast.makeText(this, "Console cleared", Toast.LENGTH_SHORT).show();
        });
        builder.setNegativeButton("Export", (dialog, which) -> {
            Toast.makeText(this, "Console log exported", Toast.LENGTH_SHORT).show();
        });
        
        AlertDialog dialog = builder.create();
        dialog.show();
        
        LogUtils.logInfo(TAG, "Plugin console opened");
    }
    
    private String generateConsoleOutput() {
        StringBuilder console = new StringBuilder();
        console.append("=== AXIOM PLUGIN CONSOLE ===\n\n");
        
        PluginManager pluginManager = PluginManager.getInstance();
        
        console.append("[System] Plugin System Status: ");
        console.append(pluginManager.isPluginSystemEnabled() ? "ENABLED" : "DISABLED");
        console.append("\n");
        
        console.append("[System] Total Plugins: ").append(pluginManager.getTotalPluginCount()).append("\n");
        console.append("[System] Loaded Plugins: ").append(pluginManager.getLoadedPluginCount()).append("\n");
        console.append("[System] Plugin Directory: ").append(pluginManager.getPluginDirectory().getAbsolutePath()).append("\n\n");
        
        console.append("=== LOADED PLUGINS ===\n");
        for (PluginInfo plugin : pluginManager.getLoadedPlugins()) {
            console.append(String.format("[%s] %s v%s - Status: %s\n", 
                plugin.id, plugin.name, plugin.version, plugin.status));
        }
        
        console.append("\n=== RECENT ACTIVITY ===\n");
        console.append("[Info] Last scan: ").append(new Date(pluginManager.getLastScanTime())).append("\n");
        console.append("[Info] Total load attempts: ").append(pluginManager.getTotalLoadAttempts()).append("\n");
        console.append("[Info] Successful loads: ").append(pluginManager.getSuccessfulLoads()).append("\n");
        console.append("[Info] Failed loads: ").append(pluginManager.getFailedLoads()).append("\n");
        
        console.append("\n=== END CONSOLE ===\n");
        
        return console.toString();
    }
    
    private void importPlugin() {
        String[] importOptions = {
            "From File (.dex, .apk, .jar)",
            "From Directory",
            "From URL",
            "From Template"
        };
        
        new AlertDialog.Builder(this)
            .setTitle("Import Plugin")
            .setItems(importOptions, (dialog, which) -> {
                switch (which) {
                    case 0:
                        importFromFile();
                        break;
                    case 1:
                        importFromDirectory();
                        break;
                    case 2:
                        importFromUrl();
                        break;
                    case 3:
                        importFromTemplate();
                        break;
                }
            })
            .show();
    }
    
    private void importFromFile() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("*/*");
        intent.putExtra(Intent.EXTRA_MIME_TYPES, new String[]{
            "application/java-archive",
            "application/vnd.android.package-archive",
            "application/octet-stream"
        });
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        
        try {
            startActivityForResult(Intent.createChooser(intent, "Select Plugin File"), REQUEST_CODE_IMPORT_FILE);
            LogUtils.logInfo(TAG, "Opening file picker for plugin import");
        } catch (Exception e) {
            Toast.makeText(this, "Cannot open file picker: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            LogUtils.logError(TAG, "Failed to open file picker", e);
        }
    }
    
    private void importFromDirectory() {
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT_TREE);
        try {
            startActivityForResult(intent, REQUEST_CODE_IMPORT_DIRECTORY);
            LogUtils.logInfo(TAG, "Opening directory picker for plugin import");
        } catch (Exception e) {
            Toast.makeText(this, "Cannot open directory picker: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            LogUtils.logError(TAG, "Failed to open directory picker", e);
        }
    }
    
    private void importFromUrl() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Import from URL");
        
        final EditText input = new EditText(this);
        input.setHint("https://example.com/plugin.dex");
        input.setInputType(android.text.InputType.TYPE_TEXT_VARIATION_URI);
        builder.setView(input);
        
        builder.setPositiveButton("Import", (dialog, which) -> {
            String url = input.getText().toString().trim();
            if (!url.isEmpty()) {
                downloadAndImportPlugin(url);
            } else {
                Toast.makeText(this, "Please enter a valid URL", Toast.LENGTH_SHORT).show();
            }
        });
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }
    
    private void importFromTemplate() {
        Toast.makeText(this, "Import from template - selecting development directory", Toast.LENGTH_SHORT).show();
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT_TREE);
        try {
            startActivityForResult(intent, REQUEST_CODE_IMPORT_TEMPLATE);
        } catch (Exception e) {
            Toast.makeText(this, "Cannot open directory picker", Toast.LENGTH_SHORT).show();
        }
    }
    
    private void downloadAndImportPlugin(String url) {
        Toast.makeText(this, "Downloading plugin from URL...", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Downloading plugin from: " + url);
        
        new Thread(() -> {
            runOnUiThread(() -> Toast.makeText(this, "URL import feature coming soon!", Toast.LENGTH_LONG).show());
            LogUtils.logInfo(TAG, "URL import requested but not yet implemented");
        }).start();
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (resultCode == RESULT_OK && data != null) {
            switch (requestCode) {
                case REQUEST_CODE_IMPORT_FILE:
                    handleImportedFile(data.getData());
                    break;
                case REQUEST_CODE_IMPORT_DIRECTORY:
                    handleImportedDirectory(data.getData());
                    break;
                case REQUEST_CODE_IMPORT_TEMPLATE:
                    handleImportedTemplate(data.getData());
                    break;
            }
        }
    }
    
    private void handleImportedFile(Uri fileUri) {
        if (fileUri == null) return;
        
        new Thread(() -> {
            try {
                runOnUiThread(() -> Toast.makeText(this, "Importing plugin file...", Toast.LENGTH_SHORT).show());
                
                File pluginDir = new File(getExternalFilesDir(null), "axiom_plugins");
                if (!pluginDir.exists()) pluginDir.mkdirs();
                
                String fileName = getFileNameFromUri(fileUri);
                File destFile = new File(pluginDir, fileName);
                
                InputStream inputStream = getContentResolver().openInputStream(fileUri);
                OutputStream outputStream = new FileOutputStream(destFile);
                
                byte[] buffer = new byte[1024];
                int length;
                while ((length = inputStream.read(buffer)) > 0) {
                    outputStream.write(buffer, 0, length);
                }
                
                inputStream.close();
                outputStream.close();
                
                runOnUiThread(() -> {
                    Toast.makeText(this, "Plugin imported: " + fileName + "\nScanning...", Toast.LENGTH_LONG).show();
                    PluginManager.getInstance().scanForPlugins();
                });
                
                LogUtils.logInfo(TAG, "Plugin imported successfully: " + fileName);
                
            } catch (Exception e) {
                runOnUiThread(() -> Toast.makeText(this, "Import failed: " + e.getMessage(), Toast.LENGTH_LONG).show());
                LogUtils.logError(TAG, "Failed to import plugin file", e);
            }
        }).start();
    }
    
    private void handleImportedDirectory(Uri dirUri) {
        Toast.makeText(this, "Scanning directory for plugins...", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Importing plugins from directory: " + dirUri);
    }
    
    private void handleImportedTemplate(Uri templateUri) {
        Toast.makeText(this, "Importing template project...", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Importing template from: " + templateUri);
    }
    
    private String getFileNameFromUri(Uri uri) {
        String fileName = "imported_plugin.dex";
        
        try {
            Cursor cursor = getContentResolver().query(uri, null, null, null, null);
            if (cursor != null && cursor.moveToFirst()) {
                int nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                if (nameIndex != -1) {
                    fileName = cursor.getString(nameIndex);
                }
                cursor.close();
            }
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to get filename from URI", e);
        }
        
        if (!fileName.endsWith(".dex") && !fileName.endsWith(".apk") && !fileName.endsWith(".jar")) {
            fileName += ".dex";
        }
        
        return fileName;
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "PluginDevelopmentActivity destroyed");
    }
}

// END OF PluginDevelopmentActivity.java

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginInfo.java
package com.axiomloader.modding;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

/**
 * PluginInfo - Container class for plugin metadata and state information
 */
public class PluginInfo {
    
    // Basic plugin information
    public String id;
    public String name;
    public String version;
    public String author;
    public String description;
    public int apiVersion;
    public String mainClass;
    
    // File information
    public File pluginFile;
    public File manifestFile;
    public long size;
    public long lastModified;
    
    // Runtime state
    public PluginStatus status;
    public String errorMessage;
    public long loadTime;
    public long unloadTime;
    
    // Permissions and security
    public List<String> permissions = new ArrayList<>();
    public boolean trusted = false;
    
    // Statistics
    public int loadCount = 0;
    public long totalRunTime = 0;
    
    public PluginInfo() {
        this.status = PluginStatus.UNKNOWN;
    }
    
    public boolean isLoaded() {
        return status == PluginStatus.LOADED;
    }
    
    public boolean hasErrors() {
        return status == PluginStatus.ERROR;
    }
    
    public String getDisplayName() {
        return name != null ? name : id;
    }
    
    public String getStatusText() {
        return status != null ? status.toString() : "UNKNOWN";
    }
    
    @Override
    public String toString() {
        return String.format("PluginInfo{id='%s', name='%s', version='%s', status=%s}", 
                id, name, version, status);
    }
}

/**
 * Plugin status enumeration
 */
enum PluginStatus {
    UNKNOWN,
    DISCOVERED,
    LOADING,
    LOADED,
    UNLOADING,
    UNLOADED,
    ERROR,
    DISABLED
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginInfoActivity.java
package com.axiomloader.modding;

import android.content.Intent;
import android.os.Bundle;
import android.view.MenuItem;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityPluginInfoBinding;
import com.axiomloader.utils.LogUtils;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

public class PluginInfoActivity extends AppCompatActivity {
    
    private ActivityPluginInfoBinding binding;
    private PluginManager pluginManager;
    private PluginInfo pluginInfo;
    private String pluginId;
    private static final String TAG = "PluginInfoActivity";
    private SimpleDateFormat dateFormat;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityPluginInfoBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.plugin_info);
        }
        
        pluginManager = PluginManager.getInstance();
        dateFormat = new SimpleDateFormat("MMM dd, yyyy HH:mm", Locale.getDefault());
        
        pluginId = getIntent().getStringExtra("plugin_id");
        if (pluginId == null || pluginId.isEmpty()) {
            Toast.makeText(this, "No plugin ID provided", Toast.LENGTH_SHORT).show();
            finish();
            return;
        }
        
        loadPluginInfo();
        setupButtons();
        
        LogUtils.logInfo(TAG, "PluginInfoActivity created for plugin: " + pluginId);
    }
    
    private void loadPluginInfo() {
        pluginInfo = pluginManager.getPluginInfo(pluginId);
        if (pluginInfo == null) {
            Toast.makeText(this, "Plugin not found: " + pluginId, Toast.LENGTH_SHORT).show();
            finish();
            return;
        }
        displayPluginInfo();
    }
    
    private void displayPluginInfo() {
        binding.tvPluginName.setText(pluginInfo.getDisplayName());
        binding.tvPluginVersion.setText("Version " + (pluginInfo.version != null ? pluginInfo.version : "Unknown"));
        binding.tvPluginAuthor.setText(pluginInfo.author != null ? pluginInfo.author : "Unknown Author");
        binding.tvPluginId.setText("ID: " + pluginInfo.id);
        
        if (pluginInfo.description != null && !pluginInfo.description.isEmpty()) {
            binding.tvPluginDescription.setText(pluginInfo.description);
        } else {
            binding.tvPluginDescription.setText("No description available");
        }
        
        updateStatusDisplay();
        displayFileInfo();
        displayRuntimeInfo();
        displayPermissions();
        displayStatistics();
    }
    
    private void updateStatusDisplay() {
        String statusText = pluginInfo.getStatusText();
        binding.tvPluginStatus.setText(statusText);
        
        int statusColor = getStatusColor(pluginInfo.status);
        binding.tvPluginStatus.setTextColor(statusColor);
        binding.statusIndicator.setBackgroundColor(statusColor);
        
        boolean isLoaded = pluginInfo.isLoaded();
        binding.btnTogglePlugin.setText(isLoaded ? "Disable" : "Enable");
        binding.btnTogglePlugin.setEnabled(pluginInfo.status != PluginStatus.ERROR);
    }
    
    private int getStatusColor(PluginStatus status) {
        switch (status) {
            case LOADED:
                return getColor(android.R.color.holo_green_dark);
            case ERROR:
                return getColor(android.R.color.holo_red_dark);
            case DISABLED:
            case UNLOADED:
                return getColor(android.R.color.darker_gray);
            case LOADING:
                return getColor(android.R.color.holo_blue_dark);
            case DISCOVERED:
                return getColor(android.R.color.holo_orange_light);
            default:
                return getColor(android.R.color.darker_gray);
        }
    }
    
    private void displayFileInfo() {
        binding.tvFileSize.setText(pluginInfo.size > 0 ? formatFileSize(pluginInfo.size) : "Unknown");
        
        if (pluginInfo.lastModified > 0) {
            binding.tvLastModified.setText(dateFormat.format(new Date(pluginInfo.lastModified)));
        } else {
            binding.tvLastModified.setText("Unknown");
        }
        
        if (pluginInfo.pluginFile != null) {
            binding.tvFilePath.setText(pluginInfo.pluginFile.getAbsolutePath());
        } else {
            binding.tvFilePath.setText("Not available");
        }
        
        binding.tvApiVersion.setText(String.valueOf(pluginInfo.apiVersion));
    }
    
    private void displayRuntimeInfo() {
        binding.tvLoadCount.setText(String.valueOf(pluginInfo.loadCount));
        
        if (pluginInfo.totalRunTime > 0) {
            binding.tvTotalRuntime.setText(formatDuration(pluginInfo.totalRunTime));
        } else {
            binding.tvTotalRuntime.setText("Not available");
        }
        
        if (pluginInfo.loadTime > 0) {
            binding.tvLoadTime.setText(dateFormat.format(new Date(pluginInfo.loadTime)));
        } else {
            binding.tvLoadTime.setText("Never loaded");
        }
        
        if (pluginInfo.errorMessage != null && !pluginInfo.errorMessage.isEmpty()) {
            binding.tvErrorMessage.setText(pluginInfo.errorMessage);
            binding.cardErrorInfo.setVisibility(android.view.View.VISIBLE);
        } else {
            binding.cardErrorInfo.setVisibility(android.view.View.GONE);
        }
    }
    
    private void displayPermissions() {
        if (pluginInfo.permissions != null && !pluginInfo.permissions.isEmpty()) {
            StringBuilder permissionsText = new StringBuilder();
            for (int i = 0; i < pluginInfo.permissions.size(); i++) {
                permissionsText.append("• ").append(pluginInfo.permissions.get(i));
                if (i < pluginInfo.permissions.size() - 1) {
                    permissionsText.append("\n");
                }
            }
            binding.tvPermissions.setText(permissionsText.toString());
        } else {
            binding.tvPermissions.setText("No permissions required");
        }
    }
    
    private void displayStatistics() {
        PluginSecurityManager securityManager = new PluginSecurityManager(this);
        PluginSecurityManager.SecurityRiskLevel riskLevel = securityManager.getRiskLevel(pluginInfo);
        
        binding.tvSecurityRisk.setText(getRiskLevelText(riskLevel));
        binding.tvSecurityRisk.setTextColor(getRiskLevelColor(riskLevel));
        
        binding.tvTrustStatus.setText(pluginInfo.trusted ? "Trusted" : "Untrusted");
        binding.tvTrustStatus.setTextColor(pluginInfo.trusted ? 
            getColor(android.R.color.holo_green_dark) : 
            getColor(android.R.color.holo_orange_dark));
    }
    
    private String getRiskLevelText(PluginSecurityManager.SecurityRiskLevel level) {
        switch (level) {
            case MINIMAL: return "Minimal Risk";
            case LOW: return "Low Risk";
            case MEDIUM: return "Medium Risk";
            case HIGH: return "High Risk";
            default: return "Unknown Risk";
        }
    }
    
    private int getRiskLevelColor(PluginSecurityManager.SecurityRiskLevel level) {
        switch (level) {
            case MINIMAL: return getColor(android.R.color.holo_green_dark);
            case LOW: return getColor(android.R.color.holo_blue_dark);
            case MEDIUM: return getColor(android.R.color.holo_orange_dark);
            case HIGH: return getColor(android.R.color.holo_red_dark);
            default: return getColor(android.R.color.darker_gray);
        }
    }
    
    private void setupButtons() {
        binding.btnTogglePlugin.setOnClickListener(v -> togglePlugin());
        binding.btnUninstallPlugin.setOnClickListener(v -> showUninstallDialog());
        binding.btnPluginSettings.setOnClickListener(v -> openPluginSettings());
        binding.btnSharePlugin.setOnClickListener(v -> sharePluginInfo());
        binding.btnRefreshInfo.setOnClickListener(v -> refreshPluginInfo());
    }
    
    private void togglePlugin() {
        boolean currentlyLoaded = pluginInfo.isLoaded();
        
        binding.btnTogglePlugin.setEnabled(false);
        binding.btnTogglePlugin.setText(currentlyLoaded ? "Disabling..." : "Enabling...");
        
        new Thread(() -> {
            try {
                if (currentlyLoaded) {
                    pluginManager.disablePlugin(pluginId);
                    LogUtils.logInfo(TAG, "Plugin disabled: " + pluginId);
                } else {
                    pluginManager.enablePlugin(pluginId);
                    LogUtils.logInfo(TAG, "Plugin enabled: " + pluginId);
                }
                
                runOnUiThread(() -> {
                    refreshPluginInfo();
                    Toast.makeText(this, currentlyLoaded ? "Plugin disabled" : "Plugin enabled", Toast.LENGTH_SHORT).show();
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to toggle plugin", e);
                runOnUiThread(() -> {
                    binding.btnTogglePlugin.setEnabled(true);
                    updateStatusDisplay();
                    Toast.makeText(this, "Failed to toggle plugin: " + e.getMessage(), Toast.LENGTH_LONG).show();
                });
            }
        }).start();
    }
    
    private void showUninstallDialog() {
        new AlertDialog.Builder(this)
            .setTitle("Uninstall Plugin")
            .setMessage("Are you sure you want to uninstall " + pluginInfo.name + "?\n\nThis action cannot be undone.")
            .setPositiveButton("Uninstall", (dialog, which) -> uninstallPlugin())
            .setNegativeButton("Cancel", null)
            .setIcon(android.R.drawable.ic_dialog_alert)
            .show();
    }
    
    private void uninstallPlugin() {
        new Thread(() -> {
            try {
                if (pluginInfo.isLoaded()) {
                    pluginManager.disablePlugin(pluginId);
                }
                
                boolean filesDeleted = true;
                if (pluginInfo.pluginFile != null && pluginInfo.pluginFile.exists()) {
                    filesDeleted = pluginInfo.pluginFile.delete();
                    if (!filesDeleted) {
                        throw new Exception("Could not delete plugin file");
                    }
                }
                if (pluginInfo.manifestFile != null && pluginInfo.manifestFile.exists()) {
                    pluginInfo.manifestFile.delete();
                }
                
                pluginManager.refreshPlugins();
                
                runOnUiThread(() -> {
                    Toast.makeText(this, "Plugin uninstalled successfully", Toast.LENGTH_SHORT).show();
                    finish();
                });
                
                LogUtils.logInfo(TAG, "Plugin uninstalled: " + pluginId);
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to uninstall plugin", e);
                runOnUiThread(() -> Toast.makeText(this, "Failed to uninstall: " + e.getMessage(), Toast.LENGTH_LONG).show());
            }
        }).start();
    }
    
    private void openPluginSettings() {
        Intent intent = new Intent(this, PluginSettingsActivity.class);
        intent.putExtra("plugin_id", pluginId);
        startActivity(intent);
    }
    
    private void refreshPluginInfo() {
        pluginInfo = pluginManager.getPluginInfo(pluginId);
        if (pluginInfo != null) {
            displayPluginInfo();
        } else {
            Toast.makeText(this, "Plugin no longer available", Toast.LENGTH_SHORT).show();
            finish();
        }
    }
    
    private void sharePluginInfo() {
        StringBuilder shareText = new StringBuilder();
        shareText.append("=== AXIOM PLUGIN INFO ===\n\n");
        shareText.append("Name: ").append(pluginInfo.name).append("\n");
        shareText.append("Version: ").append(pluginInfo.version).append("\n");
        shareText.append("Author: ").append(pluginInfo.author != null ? pluginInfo.author : "Unknown").append("\n");
        shareText.append("ID: ").append(pluginInfo.id).append("\n");
        shareText.append("Status: ").append(pluginInfo.getStatusText()).append("\n");
        shareText.append("Size: ").append(formatFileSize(pluginInfo.size)).append("\n");
        shareText.append("API Version: ").append(pluginInfo.apiVersion).append("\n");
        
        if (pluginInfo.description != null && !pluginInfo.description.isEmpty()) {
            shareText.append("\nDescription:\n").append(pluginInfo.description).append("\n");
        }
        
        if (pluginInfo.permissions != null && !pluginInfo.permissions.isEmpty()) {
            shareText.append("\nPermissions:\n");
            for (String permission : pluginInfo.permissions) {
                shareText.append("• ").append(permission).append("\n");
            }
        }
        
        if (pluginInfo.errorMessage != null && !pluginInfo.errorMessage.isEmpty()) {
            shareText.append("\nError: ").append(pluginInfo.errorMessage).append("\n");
        }
        
        shareText.append("\n--- Shared from Axiom Plugin Manager ---");
        
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_TEXT, shareText.toString());
        shareIntent.putExtra(Intent.EXTRA_SUBJECT, "Plugin Info: " + pluginInfo.name);
        startActivity(Intent.createChooser(shareIntent, "Share Plugin Info"));
    }
    
    private String formatFileSize(long sizeBytes) {
        if (sizeBytes <= 0) return "0 B";
        
        if (sizeBytes < 1024) {
            return sizeBytes + " B";
        } else if (sizeBytes < 1024 * 1024) {
            return String.format(Locale.getDefault(), "%.1f KB", sizeBytes / 1024.0);
        } else if (sizeBytes < 1024 * 1024 * 1024) {
            return String.format(Locale.getDefault(), "%.1f MB", sizeBytes / (1024.0 * 1024.0));
        } else {
            return String.format(Locale.getDefault(), "%.1f GB", sizeBytes / (1024.0 * 1024.0 * 1024.0));
        }
    }
    
    private String formatDuration(long milliseconds) {
        if (milliseconds <= 0) return "0s";
        
        long seconds = milliseconds / 1000;
        long minutes = seconds / 60;
        long hours = minutes / 60;
        
        seconds = seconds % 60;
        minutes = minutes % 60;
        
        StringBuilder duration = new StringBuilder();
        if (hours > 0) {
            duration.append(hours).append("h ");
        }
        if (minutes > 0) {
            duration.append(minutes).append("m ");
        }
        if (seconds > 0 || duration.length() == 0) {
            duration.append(seconds).append("s");
        }
        
        return duration.toString().trim();
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        refreshPluginInfo();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "PluginInfoActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginInterface.java
package com.axiomloader.modding;

import android.content.Context;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import java.util.Map;

/**
 * PluginInterface - Base interface that all plugins must implement
 * Defines the contract for plugin lifecycle and interaction with the host app
 */
public interface PluginInterface {
    
    /**
     * Called when the plugin is loaded by the system
     * @param context Application context for accessing system services
     */
    void onLoad(@NonNull Context context);
    
    /**
     * Called when the plugin is being unloaded
     * Plugins should clean up resources here
     */
    void onUnload();
    
    /**
     * Get plugin metadata
     * @return Map containing plugin information
     */
    @NonNull
    Map<String, Object> getPluginInfo();
    
    /**
     * Get plugin name for display
     * @return Human-readable plugin name
     */
    @NonNull
    String getPluginName();
    
    /**
     * Get plugin version
     * @return Plugin version string
     */
    @NonNull
    String getPluginVersion();
    
    /**
     * Get plugin description
     * @return Plugin description
     */
    @Nullable
    String getPluginDescription();
    
    /**
     * Called when the host app experiences low memory
     * Plugins should free non-essential resources
     */
    default void onLowMemory() {
        // Default empty implementation
    }
    
    /**
     * Called when system requests memory trimming
     * @param level Memory trim level from ComponentCallbacks2
     */
    default void onTrimMemory(int level) {
        // Default empty implementation
    }
    
    /**
     * Called when plugin configuration changes
     * @param config New configuration parameters
     */
    default void onConfigurationChanged(@NonNull Map<String, Object> config) {
        // Default empty implementation
    }
    
    /**
     * Check if plugin is compatible with current API version
     * @param apiVersion Current API version
     * @return true if compatible, false otherwise
     */
    default boolean isCompatible(int apiVersion) {
        return true; // Default assumes compatibility
    }
    
    /**
     * Get required permissions for this plugin
     * @return Array of permission strings, or empty array if none needed
     */
    @NonNull
    default String[] getRequiredPermissions() {
        return new String[0];
    }
    
    /**
     * Called when plugin should save its state
     * @return State data to be persisted
     */
    @Nullable
    default Map<String, Object> saveState() {
        return null;
    }
    
    /**
     * Called when plugin should restore its state
     * @param state Previously saved state data
     */
    default void restoreState(@Nullable Map<String, Object> state) {
        // Default empty implementation
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginLoader.java
package com.axiomloader.modding;

import android.content.Context;
import com.axiomloader.utils.LogUtils;
import dalvik.system.DexClassLoader;
import dalvik.system.PathClassLoader;
import java.io.File;
import java.lang.ref.WeakReference;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * PluginLoader - Handles DEX file loading and class instantiation
 * Provides secure plugin loading with class isolation and validation
 * ENHANCED: Complete implementation with resource management and lifecycle tracking
 */
public class PluginLoader {
    
    private static final String TAG = "PluginLoader";
    private final Context context;
    private final Map<String, ClassLoader> classLoaders;
    private final Map<String, Long> loadTimes;
    private final Map<String, Integer> loadCounts;
    private final Map<String, List<WeakReference<Object>>> pluginResources;
    private final File optimizedDirectory;
    
    public PluginLoader(Context context) {
        this.context = context;
        this.classLoaders = new ConcurrentHashMap<>();
        this.loadTimes = new ConcurrentHashMap<>();
        this.loadCounts = new ConcurrentHashMap<>();
        this.pluginResources = new ConcurrentHashMap<>();
        this.optimizedDirectory = new File(context.getCacheDir(), "plugin_dex");
        
        if (!optimizedDirectory.exists()) {
            optimizedDirectory.mkdirs();
        }
        
        cleanupOrphanedDexFiles();
    }
    
    /**
     * Load a plugin from DEX file
     */
    public PluginLoadResult loadPlugin(PluginInfo pluginInfo) {
        try {
            LogUtils.logInfo(TAG, "Loading plugin: " + pluginInfo.name);
            
            if (!validatePluginFile(pluginInfo.pluginFile)) {
                return PluginLoadResult.failure("Invalid plugin file");
            }
            
            ClassLoader classLoader = createClassLoader(pluginInfo);
            if (classLoader == null) {
                return PluginLoadResult.failure("Failed to create class loader");
            }
            
            Class<?> mainClass = loadMainClass(classLoader, pluginInfo.mainClass);
            if (mainClass == null) {
                return PluginLoadResult.failure("Failed to load main class: " + pluginInfo.mainClass);
            }
            
            if (!PluginInterface.class.isAssignableFrom(mainClass)) {
                return PluginLoadResult.failure("Main class does not implement PluginInterface");
            }
            
            Object pluginInstance = createPluginInstance(mainClass);
            if (pluginInstance == null) {
                return PluginLoadResult.failure("Failed to create plugin instance");
            }
            
            classLoaders.put(pluginInfo.id, classLoader);
            loadTimes.put(pluginInfo.id, System.currentTimeMillis());
            loadCounts.put(pluginInfo.id, loadCounts.getOrDefault(pluginInfo.id, 0) + 1);
            
            initializePluginResources(pluginInfo.id);
            
            LogUtils.logInfo(TAG, "Plugin loaded successfully: " + pluginInfo.name);
            return PluginLoadResult.success(pluginInstance, classLoader);
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to load plugin: " + pluginInfo.name, e);
            return PluginLoadResult.failure("Load error: " + e.getMessage());
        }
    }
    
    private boolean validatePluginFile(File pluginFile) {
        if (pluginFile == null || !pluginFile.exists()) {
            LogUtils.logError(TAG, "Plugin file does not exist");
            return false;
        }
        
        if (!pluginFile.canRead()) {
            LogUtils.logError(TAG, "Plugin file is not readable");
            return false;
        }
        
        String fileName = pluginFile.getName().toLowerCase();
        if (!fileName.endsWith(".dex") && !fileName.endsWith(".apk") && !fileName.endsWith(".jar")) {
            LogUtils.logError(TAG, "Unsupported plugin file format: " + fileName);
            return false;
        }
        
        if (pluginFile.length() == 0) {
            LogUtils.logError(TAG, "Plugin file is empty");
            return false;
        }
        
        return true;
    }
    
    private ClassLoader createClassLoader(PluginInfo pluginInfo) {
        try {
            String dexPath = pluginInfo.pluginFile.getAbsolutePath();
            String optimizedDir = optimizedDirectory.getAbsolutePath();
            String librarySearchPath = null;
            ClassLoader parentClassLoader = context.getClassLoader();
            
            DexClassLoader classLoader = new DexClassLoader(
                dexPath, 
                optimizedDir, 
                librarySearchPath, 
                parentClassLoader
            );
            
            LogUtils.logDebug(TAG, "Created class loader for: " + pluginInfo.name);
            return classLoader;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to create class loader", e);
            return null;
        }
    }
    
    private Class<?> loadMainClass(ClassLoader classLoader, String className) {
        try {
            Class<?> clazz = classLoader.loadClass(className);
            LogUtils.logDebug(TAG, "Loaded main class: " + className);
            return clazz;
            
        } catch (ClassNotFoundException e) {
            LogUtils.logError(TAG, "Main class not found: " + className, e);
            return null;
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to load main class: " + className, e);
            return null;
        }
    }
    
    private Object createPluginInstance(Class<?> mainClass) {
        try {
            Object instance = mainClass.newInstance();
            LogUtils.logDebug(TAG, "Created plugin instance: " + mainClass.getSimpleName());
            return instance;
            
        } catch (InstantiationException e) {
            LogUtils.logError(TAG, "Failed to instantiate plugin class (no default constructor?)", e);
            return null;
        } catch (IllegalAccessException e) {
            LogUtils.logError(TAG, "Cannot access plugin class constructor", e);
            return null;
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to create plugin instance", e);
            return null;
        }
    }
    
    /**
     * Initialize resource tracking for a plugin
     */
    private void initializePluginResources(String pluginId) {
        pluginResources.put(pluginId, new ArrayList<>());
    }
    
    /**
     * Register a resource with a plugin for cleanup tracking
     */
    public void registerPluginResource(String pluginId, Object resource) {
        List<WeakReference<Object>> resources = pluginResources.get(pluginId);
        if (resources != null) {
            resources.add(new WeakReference<>(resource));
        }
    }
    
    /**
     * Unload a plugin and clean up its class loader
     */
    public boolean unloadPlugin(String pluginId) {
        try {
            cleanupPluginResources(pluginId);
            
            ClassLoader classLoader = classLoaders.remove(pluginId);
            if (classLoader != null) {
                loadTimes.remove(pluginId);
                pluginResources.remove(pluginId);
                
                System.gc();
                
                LogUtils.logInfo(TAG, "Unloaded plugin class loader: " + pluginId);
                return true;
            }
            return false;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to unload plugin: " + pluginId, e);
            return false;
        }
    }
    
    /**
     * Clean up plugin resources
     */
    private void cleanupPluginResources(String pluginId) {
        List<WeakReference<Object>> resources = pluginResources.get(pluginId);
        if (resources != null) {
            for (WeakReference<Object> ref : resources) {
                Object resource = ref.get();
                if (resource != null) {
                    try {
                        if (resource instanceof AutoCloseable) {
                            ((AutoCloseable) resource).close();
                        }
                    } catch (Exception e) {
                        LogUtils.logError(TAG, "Failed to close plugin resource", e);
                    }
                }
            }
            resources.clear();
        }
    }
    
    /**
     * Get class loader for a specific plugin
     */
    public ClassLoader getPluginClassLoader(String pluginId) {
        return classLoaders.get(pluginId);
    }
    
    /**
     * Load additional classes from a plugin
     */
    public Class<?> loadPluginClass(String pluginId, String className) {
        ClassLoader classLoader = classLoaders.get(pluginId);
        if (classLoader == null) {
            LogUtils.logError(TAG, "No class loader found for plugin: " + pluginId);
            return null;
        }
        
        try {
            return classLoader.loadClass(className);
        } catch (ClassNotFoundException e) {
            LogUtils.logError(TAG, "Class not found in plugin: " + className, e);
            return null;
        }
    }
    
    /**
     * Invoke a method on a plugin class
     */
    public Object invokePluginMethod(String pluginId, String className, String methodName, Object... args) {
        try {
            Class<?> clazz = loadPluginClass(pluginId, className);
            if (clazz == null) {
                return null;
            }
            
            Method method = findMethod(clazz, methodName, args);
            if (method == null) {
                LogUtils.logError(TAG, "Method not found: " + methodName);
                return null;
            }
            
            Object instance = null;
            if (!java.lang.reflect.Modifier.isStatic(method.getModifiers())) {
                instance = clazz.newInstance();
            }
            
            return method.invoke(instance, args);
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to invoke plugin method: " + methodName, e);
            return null;
        }
    }
    
    private Method findMethod(Class<?> clazz, String methodName, Object... args) {
        Method[] methods = clazz.getDeclaredMethods();
        
        for (Method method : methods) {
            if (method.getName().equals(methodName) && method.getParameterTypes().length == args.length) {
                return method;
            }
        }
        
        return null;
    }
    
    /**
     * Get all loaded plugin class loaders
     */
    public Map<String, ClassLoader> getAllClassLoaders() {
        return new ConcurrentHashMap<>(classLoaders);
    }
    
    /**
     * Get plugin load statistics
     */
    public PluginLoadStats getPluginStats(String pluginId) {
        return new PluginLoadStats(
            loadTimes.get(pluginId),
            loadCounts.getOrDefault(pluginId, 0),
            classLoaders.containsKey(pluginId)
        );
    }
    
    /**
     * Get all plugin IDs currently loaded
     */
    public List<String> getLoadedPluginIds() {
        return new ArrayList<>(classLoaders.keySet());
    }
    
    /**
     * Check if a plugin is currently loaded
     */
    public boolean isPluginLoaded(String pluginId) {
        return classLoaders.containsKey(pluginId);
    }
    
    /**
     * Get the number of loaded plugins
     */
    public int getLoadedPluginCount() {
        return classLoaders.size();
    }
    
    /**
     * Reload a plugin (unload then load again)
     */
    public PluginLoadResult reloadPlugin(PluginInfo pluginInfo) {
        LogUtils.logInfo(TAG, "Reloading plugin: " + pluginInfo.name);
        
        if (isPluginLoaded(pluginInfo.id)) {
            unloadPlugin(pluginInfo.id);
        }
        
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return loadPlugin(pluginInfo);
    }
    
    /**
     * Clean up orphaned DEX files from previous sessions
     */
    private void cleanupOrphanedDexFiles() {
        try {
            File[] dexFiles = optimizedDirectory.listFiles();
            if (dexFiles != null) {
                long currentTime = System.currentTimeMillis();
                long maxAge = 7 * 24 * 60 * 60 * 1000L; // 7 days
                
                for (File file : dexFiles) {
                    if (currentTime - file.lastModified() > maxAge) {
                        if (file.delete()) {
                            LogUtils.logDebug(TAG, "Cleaned up orphaned DEX file: " + file.getName());
                        }
                    }
                }
            }
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to cleanup orphaned DEX files", e);
        }
    }
    
    /**
     * Force cleanup of all resources
     */
    public void forceCleanup() {
        for (String pluginId : new ArrayList<>(classLoaders.keySet())) {
            unloadPlugin(pluginId);
        }
        
        System.gc();
        System.runFinalization();
        
        LogUtils.logInfo(TAG, "Force cleanup completed");
    }
    
    /**
     * Clean up all class loaders and resources
     */
    public void cleanup() {
        for (String pluginId : new ArrayList<>(classLoaders.keySet())) {
            cleanupPluginResources(pluginId);
        }
        
        classLoaders.clear();
        loadTimes.clear();
        loadCounts.clear();
        pluginResources.clear();
        
        LogUtils.logInfo(TAG, "Plugin loader cleanup completed");
    }
    
    /**
     * Plugin load result container
     */
    public static class PluginLoadResult {
        public final boolean success;
        public final String errorMessage;
        public final Object pluginInstance;
        public final ClassLoader classLoader;
        
        private PluginLoadResult(boolean success, String errorMessage, Object pluginInstance, ClassLoader classLoader) {
            this.success = success;
            this.errorMessage = errorMessage;
            this.pluginInstance = pluginInstance;
            this.classLoader = classLoader;
        }
        
        public static PluginLoadResult success(Object pluginInstance, ClassLoader classLoader) {
            return new PluginLoadResult(true, null, pluginInstance, classLoader);
        }
        
        public static PluginLoadResult failure(String errorMessage) {
            return new PluginLoadResult(false, errorMessage, null, null);
        }
    }
    
    /**
     * Plugin load statistics
     */
    public static class PluginLoadStats {
        public final Long loadTime;
        public final int loadCount;
        public final boolean currentlyLoaded;
        
        public PluginLoadStats(Long loadTime, int loadCount, boolean currentlyLoaded) {
            this.loadTime = loadTime;
            this.loadCount = loadCount;
            this.currentlyLoaded = currentlyLoaded;
        }
        
        public long getUptime() {
            if (loadTime != null && currentlyLoaded) {
                return System.currentTimeMillis() - loadTime;
            }
            return 0;
        }
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginLoaderService.java
package com.axiomloader.modding;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import com.axiomloader.utils.LogUtils;

public class PluginLoaderService extends Service {
    
    private static final String TAG = "PluginLoaderService";
    private final IBinder binder = new LocalBinder();
    private PluginManager pluginManager;
    
    public class LocalBinder extends Binder {
        PluginLoaderService getService() {
            return PluginLoaderService.this;
        }
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        pluginManager = PluginManager.getInstance();
        LogUtils.logInfo(TAG, "PluginLoaderService created");
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        LogUtils.logInfo(TAG, "PluginLoaderService started");
        return START_STICKY;
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        LogUtils.logInfo(TAG, "PluginLoaderService destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginManager.java
package com.axiomloader.modding;

import android.content.Context;
import android.content.SharedPreferences;
import android.content.pm.ApplicationInfo;
import android.content.pm.PackageManager;
import com.axiomloader.BuildConfig;
import com.axiomloader.utils.LogUtils;
import com.google.gson.Gson;
import com.google.gson.JsonSyntaxException;
import dalvik.system.DexClassLoader;
import dalvik.system.PathClassLoader;
import timber.log.Timber;

import java.io.*;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;

/**
 * PluginManager - Central management system for DEX-based plugins
 * Handles plugin discovery, loading, lifecycle management, and security
 * 
 * FIXED: Initialization order issue resolved - initialized flag is now set
 * BEFORE any operations that depend on it
 */
public class PluginManager {
    
    private static final String TAG = "PluginManager";
    private static final String PREFS_NAME = "axiom_plugin_prefs";
    private static final String PLUGIN_DIR_NAME = "axiom_plugins";
    private static final String PLUGIN_MANIFEST_FILE = "plugin.json";
    private static final String PLUGIN_DEX_EXTENSION = ".dex";
    private static final String PLUGIN_APK_EXTENSION = ".apk";
    private static final String PLUGIN_JAR_EXTENSION = ".jar";
    
    // Singleton instance
    private static volatile PluginManager instance;
    private final AtomicBoolean initialized = new AtomicBoolean(false);
    
    // Core components
    private Context applicationContext;
    private SharedPreferences preferences;
    private ExecutorService executorService;
    private Gson gson;
    private PluginSecurityManager securityManager;
    
    // Plugin storage
    private final Map<String, PluginInfo> installedPlugins;
    private final Map<String, PluginInfo> loadedPlugins;
    private final Map<String, Object> pluginInstances;
    private final Map<String, ClassLoader> pluginClassLoaders;
    private final Set<String> enabledPlugins;
    
    // Configuration
    private volatile boolean pluginSystemEnabled;
    private File pluginDirectory;
    private File pluginCacheDirectory;
    private boolean developerModeEnabled;
    private boolean hotReloadEnabled;
    private boolean securityChecksEnabled = true;
    
    // Statistics
    private int totalLoadAttempts = 0;
    private int successfulLoads = 0;
    private int failedLoads = 0;
    private long lastScanTime = 0;
    
    private PluginManager() {
        installedPlugins = new ConcurrentHashMap<>();
        loadedPlugins = new ConcurrentHashMap<>();
        pluginInstances = new ConcurrentHashMap<>();
        pluginClassLoaders = new ConcurrentHashMap<>();
        enabledPlugins = Collections.synchronizedSet(new HashSet<>());
        executorService = Executors.newCachedThreadPool();
        gson = new Gson();
    }
    
    public static PluginManager getInstance() {
        if (instance == null) {
            synchronized (PluginManager.class) {
                if (instance == null) {
                    instance = new PluginManager();
                }
            }
        }
        return instance;
    }
    
    /**
     * Initialize the PluginManager
     * ✅ FIXED: Sets initialized flag BEFORE any dependent operations
     */
    public synchronized void initialize(Context context) {
        if (initialized.get()) {
            LogUtils.logWarning(TAG, "PluginManager already initialized");
            return;
        }
        
        try {
            LogUtils.logInfo(TAG, "Starting PluginManager initialization...");
            
            this.applicationContext = context.getApplicationContext();
            this.preferences = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
            
            // Initialize security manager
            securityManager = new PluginSecurityManager(context);
            LogUtils.logInfo(TAG, "Security manager initialized");
            
            // Setup directories
            setupPluginDirectories();
            LogUtils.logInfo(TAG, "Plugin directories setup complete");
            
            // Load preferences
            loadPreferences();
            LogUtils.logInfo(TAG, "Preferences loaded");
            
            // ✅ CRITICAL FIX: Set initialized flag BEFORE scanning
            // This prevents the circular dependency error
            initialized.set(true);
            LogUtils.logInfo(TAG, "PluginManager core initialization complete - ready for operations");
            
            // Now it's safe to scan for existing plugins
            if (pluginSystemEnabled) {
                try {
                    int foundPlugins = scanForPlugins();
                    LogUtils.logInfo(TAG, String.format("Initial plugin scan completed - found %d plugins", foundPlugins));
                } catch (Exception e) {
                    LogUtils.logError(TAG, "Error during initial plugin scan (non-fatal)", e);
                    // Don't fail initialization if scan fails - the system is still usable
                }
            } else {
                LogUtils.logInfo(TAG, "Plugin system disabled - skipping initial scan");
            }
            
            LogUtils.logInfo(TAG, "PluginManager initialization successful");
            
        } catch (Exception e) {
            initialized.set(false); // Reset flag on failure
            LogUtils.logError(TAG, "Failed to initialize PluginManager", e);
            throw new RuntimeException("PluginManager initialization failed", e);
        }
    }
    
    private void setupPluginDirectories() {
        try {
            // Main plugin directory
            pluginDirectory = new File(applicationContext.getExternalFilesDir(null), PLUGIN_DIR_NAME);
            if (!pluginDirectory.exists()) {
                boolean created = pluginDirectory.mkdirs();
                if (!created) {
                    throw new IOException("Failed to create plugin directory");
                }
            }
            
            // Cache directory for DEX optimization
            pluginCacheDirectory = new File(applicationContext.getCacheDir(), "plugin_dex");
            if (!pluginCacheDirectory.exists()) {
                boolean created = pluginCacheDirectory.mkdirs();
                if (!created) {
                    throw new IOException("Failed to create plugin cache directory");
                }
            }
            
            // Create subdirectories
            createSubDirectories();
            
            LogUtils.logInfo(TAG, "Plugin directory: " + pluginDirectory.getAbsolutePath());
            LogUtils.logInfo(TAG, "Cache directory: " + pluginCacheDirectory.getAbsolutePath());
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to setup plugin directories", e);
            throw new RuntimeException("Plugin directory setup failed", e);
        }
    }
    
    private void createSubDirectories() {
        String[] subDirs = {"installed", "temp", "data", "cache", "logs", "backups"};
        
        for (String subDir : subDirs) {
            File dir = new File(pluginDirectory, subDir);
            if (!dir.exists() && !dir.mkdirs()) {
                LogUtils.logWarning(TAG, "Failed to create subdirectory: " + subDir);
            }
        }
    }
    
    private void loadPreferences() {
        pluginSystemEnabled = preferences.getBoolean("plugin_system_enabled", true);
        developerModeEnabled = preferences.getBoolean("developer_mode_enabled", BuildConfig.DEBUG);
        hotReloadEnabled = preferences.getBoolean("hot_reload_enabled", BuildConfig.DEBUG);
        securityChecksEnabled = preferences.getBoolean("security_checks_enabled", true);
        
        // Load enabled plugins list
        Set<String> enabledSet = preferences.getStringSet("enabled_plugins", new HashSet<>());
        enabledPlugins.clear();
        if (enabledSet != null) {
            enabledPlugins.addAll(enabledSet);
        }
        
        LogUtils.logInfo(TAG, String.format("Loaded preferences - System: %s, Developer: %s, Enabled plugins: %d",
                pluginSystemEnabled, developerModeEnabled, enabledPlugins.size()));
    }
    
    private void savePreferences() {
        preferences.edit()
                .putBoolean("plugin_system_enabled", pluginSystemEnabled)
                .putBoolean("developer_mode_enabled", developerModeEnabled)
                .putBoolean("hot_reload_enabled", hotReloadEnabled)
                .putBoolean("security_checks_enabled", securityChecksEnabled)
                .putStringSet("enabled_plugins", new HashSet<>(enabledPlugins))
                .apply();
    }
    
    /**
     * Scan for plugins in the plugin directory
     * ✅ FIXED: Now safely handles being called before full initialization
     */
    public int scanForPlugins() {
        if (!initialized.get()) {
            LogUtils.logWarning(TAG, "PluginManager not fully initialized yet - scan may be limited");
            // Don't throw exception - just log warning and continue
            // This allows the method to work during initialization
        }
        
        lastScanTime = System.currentTimeMillis();
        int foundPlugins = 0;
        
        try {
            LogUtils.logInfo(TAG, "Starting plugin scan in: " + pluginDirectory.getAbsolutePath());
            
            File[] pluginFiles = pluginDirectory.listFiles((dir, name) ->
                    name.endsWith(PLUGIN_DEX_EXTENSION) ||
                    name.endsWith(PLUGIN_APK_EXTENSION) ||
                    name.endsWith(PLUGIN_JAR_EXTENSION));
            
            if (pluginFiles == null || pluginFiles.length == 0) {
                LogUtils.logInfo(TAG, "No plugin files found in directory");
                return 0;
            }
            
            LogUtils.logInfo(TAG, "Found " + pluginFiles.length + " potential plugin files");
            
            for (File pluginFile : pluginFiles) {
                try {
                    LogUtils.logInfo(TAG, "Discovering plugin: " + pluginFile.getName());
                    if (discoverPlugin(pluginFile)) {
                        foundPlugins++;
                    }
                } catch (Exception e) {
                    LogUtils.logError(TAG, "Failed to discover plugin: " + pluginFile.getName(), e);
                }
            }
            
            LogUtils.logInfo(TAG, String.format("Plugin scan completed - Found: %d new, Total: %d",
                    foundPlugins, installedPlugins.size()));
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Plugin scan failed", e);
        }
        
        return foundPlugins;
    }
    
    public int scanForNewPlugins() {
        Set<String> existingPlugins = new HashSet<>(installedPlugins.keySet());
        int totalFound = scanForPlugins();
        
        // Count only new plugins
        int newPlugins = 0;
        for (String pluginId : installedPlugins.keySet()) {
            if (!existingPlugins.contains(pluginId)) {
                newPlugins++;
            }
        }
        
        return newPlugins;
    }
    
    private boolean discoverPlugin(File pluginFile) {
        try {
            // Look for manifest file
            File manifestFile = new File(pluginFile.getParentFile(),
                    pluginFile.getName().replaceAll("\\.(dex|apk|jar)$", "_manifest.json"));
            
            if (!manifestFile.exists()) {
                // Try to extract manifest from archive if it's an APK/JAR
                manifestFile = extractManifestFromArchive(pluginFile);
            }
            
            if (manifestFile == null || !manifestFile.exists()) {
                LogUtils.logWarning(TAG, "No manifest found for: " + pluginFile.getName());
                return false;
            }
            
            // Parse manifest
            PluginManifest manifest = parseManifest(manifestFile);
            if (manifest == null) {
                return false;
            }
            
            // Create plugin info
            PluginInfo pluginInfo = new PluginInfo();
            pluginInfo.id = manifest.id;
            pluginInfo.name = manifest.name;
            pluginInfo.version = manifest.version;
            pluginInfo.author = manifest.author;
            pluginInfo.description = manifest.description;
            pluginInfo.apiVersion = manifest.apiVersion;
            pluginInfo.mainClass = manifest.mainClass;
            pluginInfo.pluginFile = pluginFile;
            pluginInfo.manifestFile = manifestFile;
            pluginInfo.size = pluginFile.length();
            pluginInfo.lastModified = pluginFile.lastModified();
            pluginInfo.status = PluginStatus.DISCOVERED;
            pluginInfo.permissions = manifest.permissions != null ? manifest.permissions : new ArrayList<>();
            
            // Validate plugin
            if (!validatePlugin(pluginInfo)) {
                return false;
            }
            
            // Store plugin info
            installedPlugins.put(pluginInfo.id, pluginInfo);
            LogUtils.logInfo(TAG, String.format("Discovered plugin: %s v%s", pluginInfo.name, pluginInfo.version));
            
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to discover plugin: " + pluginFile.getName(), e);
            return false;
        }
    }
    
    private File extractManifestFromArchive(File archiveFile) {
        // For now, return null - this would need ZIP handling for APK/JAR files
        // Implementation would extract plugin.json from the archive
        return null;
    }
    
    private PluginManifest parseManifest(File manifestFile) {
        try (FileReader reader = new FileReader(manifestFile)) {
            PluginManifest manifest = gson.fromJson(reader, PluginManifest.class);
            
            // Validate required fields
            if (manifest.id == null || manifest.id.isEmpty() ||
                manifest.name == null || manifest.name.isEmpty() ||
                manifest.version == null || manifest.version.isEmpty() ||
                manifest.mainClass == null || manifest.mainClass.isEmpty()) {
                
                LogUtils.logError(TAG, "Invalid manifest - missing required fields");
                return null;
            }
            
            return manifest;
            
        } catch (IOException | JsonSyntaxException e) {
            LogUtils.logError(TAG, "Failed to parse manifest: " + manifestFile.getName(), e);
            return null;
        }
    }
    
    private boolean validatePlugin(PluginInfo pluginInfo) {
        try {
            // Check API version compatibility
            if (pluginInfo.apiVersion > BuildConfig.PLUGIN_API_VERSION) {
                LogUtils.logError(TAG, String.format("Plugin %s requires API version %d, current is %d",
                        pluginInfo.name, pluginInfo.apiVersion, BuildConfig.PLUGIN_API_VERSION));
                return false;
            }
            
            // Security checks
            if (securityChecksEnabled && !securityManager.validatePlugin(pluginInfo)) {
                LogUtils.logError(TAG, "Plugin failed security validation: " + pluginInfo.name);
                return false;
            }
            
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Plugin validation failed: " + pluginInfo.name, e);
            return false;
        }
    }
    
    /**
     * Load a specific plugin
     */
    public boolean loadPlugin(String pluginId) {
        if (!initialized.get()) {
            LogUtils.logError(TAG, "Cannot load plugin - PluginManager not initialized");
            return false;
        }
        
        if (!pluginSystemEnabled) {
            LogUtils.logWarning(TAG, "Plugin system disabled - cannot load plugin: " + pluginId);
            return false;
        }
        
        PluginInfo pluginInfo = installedPlugins.get(pluginId);
        if (pluginInfo == null) {
            LogUtils.logError(TAG, "Plugin not found: " + pluginId);
            return false;
        }
        
        if (loadedPlugins.containsKey(pluginId)) {
            LogUtils.logWarning(TAG, "Plugin already loaded: " + pluginId);
            return true;
        }
        
        totalLoadAttempts++;
        
        try {
            LogUtils.logInfo(TAG, "Loading plugin: " + pluginInfo.name);
            
            // Create class loader
            String dexPath = pluginInfo.pluginFile.getAbsolutePath();
            String optimizedDirectory = pluginCacheDirectory.getAbsolutePath();
            String librarySearchPath = null;
            ClassLoader parent = applicationContext.getClassLoader();
            
            DexClassLoader classLoader = new DexClassLoader(dexPath, optimizedDirectory, librarySearchPath, parent);
            
            // Load main class
            Class<?> mainClass = classLoader.loadClass(pluginInfo.mainClass);
            
            // Create plugin instance
            Object pluginInstance = mainClass.newInstance();
            
            // Initialize plugin if it implements PluginInterface
            if (pluginInstance instanceof PluginInterface) {
                PluginInterface plugin = (PluginInterface) pluginInstance;
                plugin.onLoad(applicationContext);
            }
            
            // Store references
            pluginClassLoaders.put(pluginId, classLoader);
            pluginInstances.put(pluginId, pluginInstance);
            pluginInfo.status = PluginStatus.LOADED;
            loadedPlugins.put(pluginId, pluginInfo);
            
            successfulLoads++;
            LogUtils.logInfo(TAG, String.format("Plugin loaded successfully: %s v%s", 
                    pluginInfo.name, pluginInfo.version));
            
            return true;
            
        } catch (Exception e) {
            failedLoads++;
            pluginInfo.status = PluginStatus.ERROR;
            pluginInfo.errorMessage = e.getMessage();
            LogUtils.logError(TAG, "Failed to load plugin: " + pluginInfo.name, e);
            return false;
        }
    }
    
    /**
     * Unload a specific plugin
     */
    public boolean unloadPlugin(String pluginId) {
        PluginInfo pluginInfo = loadedPlugins.get(pluginId);
        if (pluginInfo == null) {
            LogUtils.logWarning(TAG, "Plugin not loaded: " + pluginId);
            return false;
        }
        
        try {
            LogUtils.logInfo(TAG, "Unloading plugin: " + pluginInfo.name);
            
            // Call plugin's unload method if it implements PluginInterface
            Object pluginInstance = pluginInstances.get(pluginId);
            if (pluginInstance instanceof PluginInterface) {
                PluginInterface plugin = (PluginInterface) pluginInstance;
                plugin.onUnload();
            }
            
            // Clean up references
            pluginInstances.remove(pluginId);
            pluginClassLoaders.remove(pluginId);
            loadedPlugins.remove(pluginId);
            pluginInfo.status = PluginStatus.UNLOADED;
            
            LogUtils.logInfo(TAG, "Plugin unloaded: " + pluginInfo.name);
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to unload plugin: " + pluginInfo.name, e);
            return false;
        }
    }
    
    /**
     * Enable/disable plugin
     */
    public void enablePlugin(String pluginId) {
        enabledPlugins.add(pluginId);
        savePreferences();
        
        if (pluginSystemEnabled) {
            loadPlugin(pluginId);
        }
    }
    
    public void disablePlugin(String pluginId) {
        enabledPlugins.remove(pluginId);
        savePreferences();
        unloadPlugin(pluginId);
    }
    
    /**
     * Load all enabled plugins
     */
    public void loadEnabledPlugins() {
        if (!initialized.get()) {
            LogUtils.logWarning(TAG, "Cannot load plugins - PluginManager not initialized");
            return;
        }
        
        if (!pluginSystemEnabled) {
            LogUtils.logInfo(TAG, "Plugin system disabled - not loading plugins");
            return;
        }
        
        LogUtils.logInfo(TAG, "Loading " + enabledPlugins.size() + " enabled plugins");
        
        for (String pluginId : enabledPlugins) {
            if (installedPlugins.containsKey(pluginId)) {
                loadPlugin(pluginId);
            } else {
                LogUtils.logWarning(TAG, "Enabled plugin not found: " + pluginId);
            }
        }
    }
    
    // System management methods
    public void enablePluginSystem() {
        pluginSystemEnabled = true;
        savePreferences();
        loadEnabledPlugins();
    }
    
    public void disablePluginSystem() {
        // Unload all plugins
        new ArrayList<>(loadedPlugins.keySet()).forEach(this::unloadPlugin);
        pluginSystemEnabled = false;
        savePreferences();
    }
    
    public void refreshPlugins() {
        scanForPlugins();
    }
    
    // Getters
    public boolean isInitialized() {
        return initialized.get();
    }
    
    public boolean isPluginSystemEnabled() {
        return pluginSystemEnabled;
    }
    
    public File getPluginDirectory() {
        return pluginDirectory;
    }
    
    public int getTotalPluginCount() {
        return installedPlugins.size();
    }
    
    public int getEnabledPluginCount() {
        return enabledPlugins.size();
    }
    
    public int getLoadedPluginCount() {
        return loadedPlugins.size();
    }
    
    public List<PluginInfo> getAllPlugins() {
        return new ArrayList<>(installedPlugins.values());
    }
    
    public List<PluginInfo> getLoadedPlugins() {
        return new ArrayList<>(loadedPlugins.values());
    }
    
    public PluginInfo getPluginInfo(String pluginId) {
        return installedPlugins.get(pluginId);
    }
    
    public Object getPluginInstance(String pluginId) {
        return pluginInstances.get(pluginId);
    }
    
    public boolean isPluginLoaded(String pluginId) {
        return loadedPlugins.containsKey(pluginId);
    }
    
    public boolean isPluginEnabled(String pluginId) {
        return enabledPlugins.contains(pluginId);
    }
    
    // Statistics
    public int getTotalLoadAttempts() {
        return totalLoadAttempts;
    }
    
    public int getSuccessfulLoads() {
        return successfulLoads;
    }
    
    public int getFailedLoads() {
        return failedLoads;
    }
    
    public long getLastScanTime() {
        return lastScanTime;
    }
    
    // Memory management
    public void onLowMemory() {
        LogUtils.logWarning(TAG, "Low memory - notifying plugins");
        
        // Notify all loaded plugins
        for (Object instance : pluginInstances.values()) {
            if (instance instanceof PluginInterface) {
                try {
                    ((PluginInterface) instance).onLowMemory();
                } catch (Exception e) {
                    LogUtils.logError(TAG, "Plugin onLowMemory failed", e);
                }
            }
        }
    }
    
    public void onTrimMemory(int level) {
        LogUtils.logInfo(TAG, "Trim memory level: " + level);
        
        // Notify all loaded plugins
        for (Object instance : pluginInstances.values()) {
            if (instance instanceof PluginInterface) {
                try {
                    ((PluginInterface) instance).onTrimMemory(level);
                } catch (Exception e) {
                    LogUtils.logError(TAG, "Plugin onTrimMemory failed", e);
                }
            }
        }
    }
    
    // Cleanup
    public void shutdown() {
        try {
            LogUtils.logInfo(TAG, "Shutting down PluginManager");
            
            // Unload all plugins
            new ArrayList<>(loadedPlugins.keySet()).forEach(this::unloadPlugin);
            
            // Shutdown executor
            if (executorService != null && !executorService.isShutdown()) {
                executorService.shutdown();
            }
            
            // Clear all collections
            installedPlugins.clear();
            loadedPlugins.clear();
            pluginInstances.clear();
            pluginClassLoaders.clear();
            enabledPlugins.clear();
            
            // Reset initialized flag
            initialized.set(false);
            
            LogUtils.logInfo(TAG, "PluginManager shutdown completed");
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Error during shutdown", e);
        }
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginManagerActivity.java
package com.axiomloader.modding;

import android.content.Intent;
import android.os.Bundle;
import android.view.MenuItem;
import android.view.View;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityPluginManagerBinding;
import com.axiomloader.utils.LogUtils;
import timber.log.Timber;

import java.util.ArrayList;
import java.util.List;

public class PluginManagerActivity extends AppCompatActivity {
    
    private ActivityPluginManagerBinding binding;
    private PluginManager pluginManager;
    private PluginAdapter pluginAdapter;
    private List<PluginInfo> pluginList;
    private static final String TAG = "PluginManagerActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityPluginManagerBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.plugin_manager_title);
        }
        
        pluginManager = PluginManager.getInstance();
        pluginList = new ArrayList<>();
        
        setupRecyclerView();
        setupButtons();
        loadPlugins();
        
        LogUtils.logInfo(TAG, "PluginManagerActivity created");
    }
    
    private void setupRecyclerView() {
        pluginAdapter = new PluginAdapter(this, pluginList, new PluginAdapter.PluginActionListener() {
            @Override
            public void onPluginClick(PluginInfo plugin) {
                showPluginDetails(plugin);
            }
            
            @Override
            public void onEnableToggle(PluginInfo plugin, boolean enabled) {
                togglePlugin(plugin, enabled);
            }
            
            @Override
            public void onPluginSettings(PluginInfo plugin) {
                openPluginSettings(plugin);
            }
            
            @Override
            public void onPluginUninstall(PluginInfo plugin) {
                showUninstallDialog(plugin);
            }
        });
        
        binding.recyclerViewPlugins.setLayoutManager(new LinearLayoutManager(this));
        binding.recyclerViewPlugins.setAdapter(pluginAdapter);
    }
    
    private void setupButtons() {
        binding.btnInstallPlugin.setOnClickListener(v -> {
            // Open file picker for plugin installation
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("*/*");
            intent.putExtra(Intent.EXTRA_MIME_TYPES, new String[]{"application/java-archive", "application/vnd.android.package-archive"});
            startActivityForResult(Intent.createChooser(intent, "Select Plugin File"), 100);
        });
        
        binding.btnScanPlugins.setOnClickListener(v -> scanForPlugins());
        
        binding.btnRefreshList.setOnClickListener(v -> loadPlugins());
        
        binding.fabAddPlugin.setOnClickListener(v -> {
            Intent intent = new Intent(this, PluginDevelopmentActivity.class);
            startActivity(intent);
        });
    }
    
    private void loadPlugins() {
        binding.progressBar.setVisibility(View.VISIBLE);
        binding.tvNoPlugins.setVisibility(View.GONE);
        
        new Thread(() -> {
            try {
                List<PluginInfo> allPlugins = pluginManager.getAllPlugins();
                
                runOnUiThread(() -> {
                    binding.progressBar.setVisibility(View.GONE);
                    pluginList.clear();
                    pluginList.addAll(allPlugins);
                    pluginAdapter.notifyDataSetChanged();
                    
                    if (pluginList.isEmpty()) {
                        binding.tvNoPlugins.setVisibility(View.VISIBLE);
                    }
                    
                    updateStats();
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to load plugins", e);
                runOnUiThread(() -> {
                    binding.progressBar.setVisibility(View.GONE);
                    Toast.makeText(this, "Failed to load plugins", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void scanForPlugins() {
        binding.btnScanPlugins.setEnabled(false);
        binding.btnScanPlugins.setText("Scanning...");
        
        new Thread(() -> {
            try {
                int foundPlugins = pluginManager.scanForNewPlugins();
                
                runOnUiThread(() -> {
                    binding.btnScanPlugins.setEnabled(true);
                    binding.btnScanPlugins.setText("Scan Plugins");
                    
                    String message = foundPlugins > 0 ? 
                        "Found " + foundPlugins + " new plugins" : 
                        "No new plugins found";
                    
                    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                    
                    if (foundPlugins > 0) {
                        loadPlugins();
                    }
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Plugin scan failed", e);
                runOnUiThread(() -> {
                    binding.btnScanPlugins.setEnabled(true);
                    binding.btnScanPlugins.setText("Scan Plugins");
                    Toast.makeText(this, "Plugin scan failed", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void togglePlugin(PluginInfo plugin, boolean enabled) {
        new Thread(() -> {
            try {
                if (enabled) {
                    pluginManager.enablePlugin(plugin.id);
                    LogUtils.logInfo(TAG, "Plugin enabled: " + plugin.name);
                } else {
                    pluginManager.disablePlugin(plugin.id);
                    LogUtils.logInfo(TAG, "Plugin disabled: " + plugin.name);
                }
                
                runOnUiThread(() -> {
                    Toast.makeText(this, 
                        plugin.name + (enabled ? " enabled" : " disabled"), 
                        Toast.LENGTH_SHORT).show();
                    updateStats();
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to toggle plugin: " + plugin.name, e);
                runOnUiThread(() -> {
                    Toast.makeText(this, "Failed to toggle plugin", Toast.LENGTH_SHORT).show();
                    pluginAdapter.notifyDataSetChanged(); // Reset switch state
                });
            }
        }).start();
    }
    
    private void showPluginDetails(PluginInfo plugin) {
        Intent intent = new Intent(this, PluginInfoActivity.class);
        intent.putExtra("plugin_id", plugin.id);
        startActivity(intent);
    }
    
    private void openPluginSettings(PluginInfo plugin) {
        Intent intent = new Intent(this, PluginSettingsActivity.class);
        intent.putExtra("plugin_id", plugin.id);
        startActivity(intent);
    }
    
    private void showUninstallDialog(PluginInfo plugin) {
        new AlertDialog.Builder(this)
            .setTitle("Uninstall Plugin")
            .setMessage("Are you sure you want to uninstall " + plugin.name + "?")
            .setPositiveButton("Uninstall", (dialog, which) -> uninstallPlugin(plugin))
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void uninstallPlugin(PluginInfo plugin) {
        new Thread(() -> {
            try {
                // First disable the plugin
                pluginManager.disablePlugin(plugin.id);
                
                // Delete plugin files
                if (plugin.pluginFile != null && plugin.pluginFile.exists()) {
                    boolean deleted = plugin.pluginFile.delete();
                    if (!deleted) {
                        throw new Exception("Failed to delete plugin file");
                    }
                }
                
                if (plugin.manifestFile != null && plugin.manifestFile.exists()) {
                    plugin.manifestFile.delete();
                }
                
                // Remove from manager
                pluginManager.refreshPlugins();
                
                runOnUiThread(() -> {
                    Toast.makeText(this, plugin.name + " uninstalled", Toast.LENGTH_SHORT).show();
                    loadPlugins();
                });
                
                LogUtils.logInfo(TAG, "Plugin uninstalled: " + plugin.name);
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to uninstall plugin: " + plugin.name, e);
                runOnUiThread(() -> {
                    Toast.makeText(this, "Failed to uninstall plugin", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void updateStats() {
        int totalPlugins = pluginManager.getTotalPluginCount();
        int enabledPlugins = pluginManager.getEnabledPluginCount();
        int loadedPlugins = pluginManager.getLoadedPluginCount();
        
        binding.tvTotalPlugins.setText("Total: " + totalPlugins);
        binding.tvEnabledPlugins.setText("Enabled: " + enabledPlugins);
        binding.tvLoadedPlugins.setText("Loaded: " + loadedPlugins);
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (requestCode == 100 && resultCode == RESULT_OK && data != null) {
            // Handle plugin file selection
            // This would implement plugin installation from file picker
            Toast.makeText(this, "Plugin installation feature coming soon", Toast.LENGTH_SHORT).show();
        }
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        loadPlugins();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "PluginManagerActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginManifest.java
package com.axiomloader.modding;

import com.google.gson.annotations.SerializedName;
import java.util.List;
import java.util.Map;

/**
 * PluginManifest - Data structure for plugin.json manifest files
 * Defines all metadata and configuration for a plugin
 */
public class PluginManifest {
    
    // Required fields
    @SerializedName("id")
    public String id;
    
    @SerializedName("name")
    public String name;
    
    @SerializedName("version")
    public String version;
    
    @SerializedName("main_class")
    public String mainClass;
    
    @SerializedName("api_version")
    public int apiVersion;
    
    // Optional metadata
    @SerializedName("author")
    public String author;
    
    @SerializedName("description")
    public String description;
    
    @SerializedName("website")
    public String website;
    
    @SerializedName("license")
    public String license;
    
    @SerializedName("min_android_version")
    public int minAndroidVersion = 21; // Default to API 21
    
    @SerializedName("target_android_version")
    public int targetAndroidVersion = 33;
    
    // Dependencies
    @SerializedName("dependencies")
    public List<PluginDependency> dependencies;
    
    // Permissions required by plugin
    @SerializedName("permissions")
    public List<String> permissions;
    
    // Plugin capabilities and features
    @SerializedName("features")
    public List<String> features;
    
    // Plugin configuration
    @SerializedName("config")
    public Map<String, Object> config;
    
    // Plugin assets and resources
    @SerializedName("assets")
    public List<String> assets;
    
    // Plugin entry points for different hooks
    @SerializedName("hooks")
    public Map<String, String> hooks;
    
    // Plugin compatibility information
    @SerializedName("compatibility")
    public CompatibilityInfo compatibility;
    
    // Plugin update information
    @SerializedName("update_url")
    public String updateUrl;
    
    @SerializedName("download_url")
    public String downloadUrl;
    
    // Plugin signing and security
    @SerializedName("signature")
    public String signature;
    
    @SerializedName("public_key")
    public String publicKey;
    
    // Plugin categories and tags
    @SerializedName("category")
    public String category;
    
    @SerializedName("tags")
    public List<String> tags;
    
    // Plugin icon and screenshots
    @SerializedName("icon")
    public String icon;
    
    @SerializedName("screenshots")
    public List<String> screenshots;
    
    // Plugin metadata
    @SerializedName("size")
    public long size;
    
    @SerializedName("created_date")
    public String createdDate;
    
    @SerializedName("updated_date")
    public String updatedDate;
    
    /**
     * Plugin dependency information
     */
    public static class PluginDependency {
        @SerializedName("id")
        public String id;
        
        @SerializedName("name")
        public String name;
        
        @SerializedName("version")
        public String version;
        
        @SerializedName("required")
        public boolean required = true;
        
        @SerializedName("minimum_version")
        public String minimumVersion;
        
        @SerializedName("maximum_version")
        public String maximumVersion;
        
        @SerializedName("download_url")
        public String downloadUrl;
        
        public boolean isVersionCompatible(String installedVersion) {
            if (installedVersion == null) return false;
            
            // Simple version comparison - could be enhanced
            if (minimumVersion != null && compareVersions(installedVersion, minimumVersion) < 0) {
                return false;
            }
            
            if (maximumVersion != null && compareVersions(installedVersion, maximumVersion) > 0) {
                return false;
            }
            
            return true;
        }
        
        private int compareVersions(String v1, String v2) {
            String[] parts1 = v1.split("\\.");
            String[] parts2 = v2.split("\\.");
            
            int maxLength = Math.max(parts1.length, parts2.length);
            
            for (int i = 0; i < maxLength; i++) {
                int num1 = i < parts1.length ? Integer.parseInt(parts1[i]) : 0;
                int num2 = i < parts2.length ? Integer.parseInt(parts2[i]) : 0;
                
                if (num1 < num2) return -1;
                if (num1 > num2) return 1;
            }
            
            return 0;
        }
    }
    
    /**
     * Plugin compatibility information
     */
    public static class CompatibilityInfo {
        @SerializedName("min_app_version")
        public String minAppVersion;
        
        @SerializedName("max_app_version")
        public String maxAppVersion;
        
        @SerializedName("supported_architectures")
        public List<String> supportedArchitectures;
        
        @SerializedName("blacklisted_devices")
        public List<String> blacklistedDevices;
        
        @SerializedName("required_features")
        public List<String> requiredFeatures;
        
        @SerializedName("supported_android_versions")
        public List<Integer> supportedAndroidVersions;
        
        @SerializedName("requires_root")
        public boolean requiresRoot = false;
        
        @SerializedName("requires_internet")
        public boolean requiresInternet = false;
        
        public boolean isDeviceCompatible(String deviceModel) {
            if (blacklistedDevices != null && blacklistedDevices.contains(deviceModel)) {
                return false;
            }
            return true;
        }
        
        public boolean isArchitectureSupported(String arch) {
            if (supportedArchitectures == null || supportedArchitectures.isEmpty()) {
                return true; // Assume compatible if not specified
            }
            return supportedArchitectures.contains(arch);
        }
        
        public boolean isAndroidVersionSupported(int apiLevel) {
            if (supportedAndroidVersions == null || supportedAndroidVersions.isEmpty()) {
                return true; // Assume compatible if not specified
            }
            return supportedAndroidVersions.contains(apiLevel);
        }
    }
    
    /**
     * Validate the manifest for required fields
     */
    public boolean isValid() {
        return id != null && !id.isEmpty() &&
               name != null && !name.isEmpty() &&
               version != null && !version.isEmpty() &&
               mainClass != null && !mainClass.isEmpty() &&
               apiVersion > 0;
    }
    
    /**
     * Get display version with name
     */
    public String getDisplayVersion() {
        return name + " v" + version;
    }
    
    /**
     * Get formatted size string
     */
    public String getFormattedSize() {
        if (size <= 0) return "Unknown";
        
        if (size < 1024) {
            return size + " B";
        } else if (size < 1024 * 1024) {
            return String.format("%.1f KB", size / 1024.0);
        } else {
            return String.format("%.1f MB", size / (1024.0 * 1024.0));
        }
    }
    
    /**
     * Check if plugin has specific permission
     */
    public boolean hasPermission(String permission) {
        return permissions != null && permissions.contains(permission);
    }
    
    /**
     * Check if plugin has specific feature
     */
    public boolean hasFeature(String feature) {
        return features != null && features.contains(feature);
    }
    
    /**
     * Get configuration value
     */
    @SuppressWarnings("unchecked")
    public <T> T getConfigValue(String key, T defaultValue) {
        if (config == null || !config.containsKey(key)) {
            return defaultValue;
        }
        
        try {
            return (T) config.get(key);
        } catch (ClassCastException e) {
            return defaultValue;
        }
    }
    
    /**
     * Check if plugin supports hot reload
     */
    public boolean supportsHotReload() {
        return hasFeature("hot_reload");
    }
    
    /**
     * Check if plugin is a system plugin
     */
    public boolean isSystemPlugin() {
        return hasFeature("system_plugin");
    }
    
    /**
     * Check if plugin requires restart
     */
    public boolean requiresRestart() {
        return hasFeature("requires_restart");
    }
    
    /**
     * Get plugin category or default
     */
    public String getCategoryOrDefault() {
        return category != null ? category : "General";
    }
    
    /**
     * Get plugin author or default
     */
    public String getAuthorOrDefault() {
        return author != null ? author : "Unknown";
    }
    
    /**
     * Get plugin description or default
     */
    public String getDescriptionOrDefault() {
        return description != null ? description : "No description available";
    }
    
    @Override
    public String toString() {
        return String.format("PluginManifest{id='%s', name='%s', version='%s', author='%s', apiVersion=%d}",
                id, name, version, author, apiVersion);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        PluginManifest that = (PluginManifest) obj;
        return id != null ? id.equals(that.id) : that.id == null;
    }
    
    @Override
    public int hashCode() {
        return id != null ? id.hashCode() : 0;
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginProvider.java
package com.axiomloader.modding;

import android.content.ContentProvider;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import com.axiomloader.utils.LogUtils;

public class PluginProvider extends ContentProvider {
    
    private static final String TAG = "PluginProvider";
    public static final String AUTHORITY = "com.axiomloader.plugins";
    public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY);
    
    @Override
    public boolean onCreate() {
        LogUtils.logInfo(TAG, "PluginProvider created");
        return true;
    }
    
    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection,
                        @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        LogUtils.logDebug(TAG, "Query: " + uri.toString());
        return null;
    }
    
    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return "vnd.android.cursor.dir/vnd.axiomloader.plugin";
    }
    
    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        LogUtils.logDebug(TAG, "Insert: " + uri.toString());
        return null;
    }
    
    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        LogUtils.logDebug(TAG, "Delete: " + uri.toString());
        return 0;
    }
    
    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection,
                      @Nullable String[] selectionArgs) {
        LogUtils.logDebug(TAG, "Update: " + uri.toString());
        return 0;
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginSecurityManager.java
package com.axiomloader.modding;

import android.content.Context;
import android.content.pm.PackageManager;
import android.os.Build;
import com.axiomloader.utils.LogUtils;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.HashSet;
import java.util.Set;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

/**
 * PluginSecurityManager - Handles security validation and sandboxing for plugins
 * Provides security checks, permission validation, and threat detection
 */
public class PluginSecurityManager {
    
    private static final String TAG = "PluginSecurityManager";
    
    private final Context context;
    private final Set<String> trustedSignatures;
    private final Set<String> blacklistedPlugins;
    private final Set<String> dangerousPermissions;
    
    public PluginSecurityManager(Context context) {
        this.context = context;
        this.trustedSignatures = new HashSet<>();
        this.blacklistedPlugins = new HashSet<>();
        this.dangerousPermissions = new HashSet<>();
        
        initializeDangerousPermissions();
        loadTrustedSignatures();
        loadBlacklist();
    }
    
    private void initializeDangerousPermissions() {
        // System-level dangerous permissions
        dangerousPermissions.add("android.permission.WRITE_EXTERNAL_STORAGE");
        dangerousPermissions.add("android.permission.READ_EXTERNAL_STORAGE");
        dangerousPermissions.add("android.permission.CAMERA");
        dangerousPermissions.add("android.permission.RECORD_AUDIO");
        dangerousPermissions.add("android.permission.ACCESS_FINE_LOCATION");
        dangerousPermissions.add("android.permission.ACCESS_COARSE_LOCATION");
        dangerousPermissions.add("android.permission.READ_CONTACTS");
        dangerousPermissions.add("android.permission.WRITE_CONTACTS");
        dangerousPermissions.add("android.permission.SEND_SMS");
        dangerousPermissions.add("android.permission.RECEIVE_SMS");
        dangerousPermissions.add("android.permission.CALL_PHONE");
        dangerousPermissions.add("android.permission.READ_PHONE_STATE");
        dangerousPermissions.add("android.permission.SYSTEM_ALERT_WINDOW");
        dangerousPermissions.add("android.permission.WRITE_SETTINGS");
        
        // Custom dangerous permissions for our app
        dangerousPermissions.add("axiom.permission.MODIFY_SYSTEM");
        dangerousPermissions.add("axiom.permission.ACCESS_LOGS");
        dangerousPermissions.add("axiom.permission.INSTALL_PLUGINS");
        dangerousPermissions.add("axiom.permission.NETWORK_ACCESS");
    }
    
    private void loadTrustedSignatures() {
        // In a real implementation, these would be loaded from a secure store
        // For now, we'll use placeholder signatures
        trustedSignatures.add("TRUSTED_DEVELOPER_1_SHA256");
        trustedSignatures.add("TRUSTED_DEVELOPER_2_SHA256");
    }
    
    private void loadBlacklist() {
        // Load known malicious plugin IDs or signatures
        blacklistedPlugins.add("malicious.plugin.example");
        blacklistedPlugins.add("banned.plugin.id");
    }
    
    /**
     * Validate a plugin for security compliance
     */
    public boolean validatePlugin(PluginInfo pluginInfo) {
        try {
            LogUtils.logInfo(TAG, "Validating plugin security: " + pluginInfo.name);
            
            // Check blacklist
            if (isBlacklisted(pluginInfo)) {
                LogUtils.logError(TAG, "Plugin is blacklisted: " + pluginInfo.id);
                return false;
            }
            
            // Validate file integrity
            if (!validateFileIntegrity(pluginInfo.pluginFile)) {
                LogUtils.logError(TAG, "Plugin file integrity check failed: " + pluginInfo.name);
                return false;
            }
            
            // Check permissions
            if (!validatePermissions(pluginInfo)) {
                LogUtils.logError(TAG, "Plugin permission validation failed: " + pluginInfo.name);
                return false;
            }
            
            // Scan for potential threats
            if (!scanForThreats(pluginInfo.pluginFile)) {
                LogUtils.logError(TAG, "Plugin threat scan failed: " + pluginInfo.name);
                return false;
            }
            
            // Validate signature if present
            if (pluginInfo.manifestFile != null && !validateSignature(pluginInfo)) {
                LogUtils.logWarning(TAG, "Plugin signature validation failed, but allowing: " + pluginInfo.name);
                // Don't fail validation for unsigned plugins in debug mode
            }
            
            // Check API compatibility
            if (!validateApiCompatibility(pluginInfo)) {
                LogUtils.logError(TAG, "Plugin API compatibility check failed: " + pluginInfo.name);
                return false;
            }
            
            LogUtils.logInfo(TAG, "Plugin security validation passed: " + pluginInfo.name);
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Plugin security validation error: " + pluginInfo.name, e);
            return false;
        }
    }
    
    private boolean isBlacklisted(PluginInfo pluginInfo) {
        return blacklistedPlugins.contains(pluginInfo.id);
    }
    
    private boolean validateFileIntegrity(File pluginFile) {
        try {
            // Check if file exists and is readable
            if (!pluginFile.exists() || !pluginFile.canRead()) {
                return false;
            }
            
            // Check file size (prevent extremely large files)
            long maxSize = 50 * 1024 * 1024; // 50MB limit
            if (pluginFile.length() > maxSize) {
                LogUtils.logError(TAG, "Plugin file too large: " + pluginFile.length() + " bytes");
                return false;
            }
            
            // Basic file format validation
            String fileName = pluginFile.getName().toLowerCase();
            if (!fileName.endsWith(".dex") && !fileName.endsWith(".apk") && !fileName.endsWith(".jar")) {
                return false;
            }
            
            // Calculate file hash for integrity
            String fileHash = calculateFileHash(pluginFile);
            if (fileHash == null) {
                return false;
            }
            
            LogUtils.logDebug(TAG, "Plugin file hash: " + fileHash);
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "File integrity validation error", e);
            return false;
        }
    }
    
    private String calculateFileHash(File file) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            FileInputStream fis = new FileInputStream(file);
            
            byte[] byteArray = new byte[1024];
            int bytesCount;
            
            while ((bytesCount = fis.read(byteArray)) != -1) {
                digest.update(byteArray, 0, bytesCount);
            }
            
            fis.close();
            
            byte[] bytes = digest.digest();
            StringBuilder sb = new StringBuilder();
            for (byte b : bytes) {
                sb.append(String.format("%02x", b));
            }
            
            return sb.toString();
            
        } catch (NoSuchAlgorithmException | IOException e) {
            LogUtils.logError(TAG, "Hash calculation failed", e);
            return null;
        }
    }
    
    private boolean validatePermissions(PluginInfo pluginInfo) {
        if (pluginInfo.permissions == null || pluginInfo.permissions.isEmpty()) {
            return true; // No permissions required
        }
        
        for (String permission : pluginInfo.permissions) {
            // Check if permission is dangerous
            if (dangerousPermissions.contains(permission)) {
                LogUtils.logWarning(TAG, "Plugin requests dangerous permission: " + permission);
                
                // In a production app, you might want to prompt user or require special approval
                if (!pluginInfo.trusted) {
                    LogUtils.logWarning(TAG, "Untrusted plugin requesting dangerous permission");
                }
            }
            
            // Check if our app has this permission
            if (permission.startsWith("android.permission.")) {
                int result = context.checkSelfPermission(permission);
                if (result != PackageManager.PERMISSION_GRANTED) {
                    LogUtils.logError(TAG, "Host app doesn't have required permission: " + permission);
                    return false;
                }
            }
        }
        
        return true;
    }
    
    private boolean scanForThreats(File pluginFile) {
        try {
            // Basic threat scanning - look for suspicious patterns
            String fileName = pluginFile.getName().toLowerCase();
            
            // Check for suspicious file names
            String[] suspiciousNames = {"malware", "virus", "trojan", "backdoor", "exploit"};
            for (String suspicious : suspiciousNames) {
                if (fileName.contains(suspicious)) {
                    LogUtils.logError(TAG, "Suspicious filename detected: " + fileName);
                    return false;
                }
            }
            
            // If it's a ZIP-based format (APK, JAR), scan contents
            if (fileName.endsWith(".apk") || fileName.endsWith(".jar")) {
                return scanZipContents(pluginFile);
            }
            
            return true;
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Threat scanning error", e);
            return false;
        }
    }
    
    private boolean scanZipContents(File zipFile) {
        try (ZipInputStream zis = new ZipInputStream(new FileInputStream(zipFile))) {
            ZipEntry entry;
            
            while ((entry = zis.getNextEntry()) != null) {
                String entryName = entry.getName().toLowerCase();
                
                // Check for suspicious entry names
                if (entryName.contains("../") || entryName.contains("..\\")) {
                    LogUtils.logError(TAG, "Suspicious zip entry: " + entryName);
                    return false;
                }
                
                // Check for executable files in unexpected locations
                if (entryName.endsWith(".so") && !entryName.startsWith("lib/")) {
                    LogUtils.logWarning(TAG, "Native library in unexpected location: " + entryName);
                }
                
                // Limit entry size to prevent zip bombs
                if (entry.getSize() > 100 * 1024 * 1024) { // 100MB per entry
                    LogUtils.logError(TAG, "Zip entry too large: " + entryName);
                    return false;
                }
                
                zis.closeEntry();
            }
            
            return true;
            
        } catch (IOException e) {
            LogUtils.logError(TAG, "Zip scanning error", e);
            return false;
        }
    }
    
    private boolean validateSignature(PluginInfo pluginInfo) {
        // In a real implementation, this would verify digital signatures
        // For now, just return true for development
        return true;
    }
    
    private boolean validateApiCompatibility(PluginInfo pluginInfo) {
        // Check minimum Android version
        if (Build.VERSION.SDK_INT < 21) { // Minimum API level 21
            LogUtils.logError(TAG, "Android version too old for plugins");
            return false;
        }
        
        // Check API version compatibility
        if (pluginInfo.apiVersion > com.axiomloader.BuildConfig.PLUGIN_API_VERSION) {
            LogUtils.logError(TAG, String.format("Plugin requires newer API version: %d > %d",
                    pluginInfo.apiVersion, com.axiomloader.BuildConfig.PLUGIN_API_VERSION));
            return false;
        }
        
        return true;
    }
    
    /**
     * Check if a plugin can access a specific resource
     */
    public boolean canAccessResource(String pluginId, String resource) {
        // Implement resource access control
        return true; // Simplified for now
    }
    
    /**
     * Check if a plugin can perform a specific action
     */
    public boolean canPerformAction(String pluginId, String action) {
        // Implement action-based security
        return true; // Simplified for now
    }
    
    /**
     * Get security risk level for a plugin
     */
    public SecurityRiskLevel getRiskLevel(PluginInfo pluginInfo) {
        int riskScore = 0;
        
        // Check permissions
        if (pluginInfo.permissions != null) {
            for (String permission : pluginInfo.permissions) {
                if (dangerousPermissions.contains(permission)) {
                    riskScore += 10;
                }
            }
        }
        
        // Check if trusted
        if (!pluginInfo.trusted) {
            riskScore += 5;
        }
        
        // Check file size (larger files might be more complex/risky)
        if (pluginInfo.size > 10 * 1024 * 1024) { // > 10MB
            riskScore += 2;
        }
        
        // Determine risk level
        if (riskScore >= 20) return SecurityRiskLevel.HIGH;
        if (riskScore >= 10) return SecurityRiskLevel.MEDIUM;
        if (riskScore >= 5) return SecurityRiskLevel.LOW;
        return SecurityRiskLevel.MINIMAL;
    }
    
    /**
     * Security risk levels
     */
    public enum SecurityRiskLevel {
        MINIMAL,
        LOW,
        MEDIUM,
        HIGH
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginSettingsActivity.java
package com.axiomloader.modding;

import android.content.SharedPreferences;
import android.os.Bundle;
import android.view.MenuItem;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityPluginSettingsBinding;
import com.axiomloader.utils.LogUtils;

public class PluginSettingsActivity extends AppCompatActivity {
    
    private ActivityPluginSettingsBinding binding;
    private PluginManager pluginManager;
    private String pluginId;
    private SharedPreferences preferences;
    private static final String TAG = "PluginSettingsActivity";
    private static final String PREFS_NAME = "axiom_plugin_settings";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityPluginSettingsBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.plugin_settings);
        }
        
        pluginManager = PluginManager.getInstance();
        preferences = getSharedPreferences(PREFS_NAME, MODE_PRIVATE);
        
        pluginId = getIntent().getStringExtra("plugin_id");
        
        setupSettings();
        loadCurrentSettings();
        
        LogUtils.logInfo(TAG, "PluginSettingsActivity created" + (pluginId != null ? " for plugin: " + pluginId : ""));
    }
    
    private void setupSettings() {
        binding.switchPluginSystemEnabled.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) {
                togglePluginSystem(isChecked);
            }
        });
        
        binding.switchDeveloperMode.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) {
                toggleDeveloperMode(isChecked);
            }
        });
        
        binding.switchHotReload.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) {
                toggleHotReload(isChecked);
            }
        });
        
        binding.switchSecurityChecks.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) {
                toggleSecurityChecks(isChecked);
            }
        });
        
        binding.switchAutoCleanup.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (buttonView.isPressed()) {
                toggleAutoCleanup(isChecked);
            }
        });
    }
    
    private void loadCurrentSettings() {
        binding.switchPluginSystemEnabled.setChecked(pluginManager.isPluginSystemEnabled());
        
        binding.switchDeveloperMode.setChecked(preferences.getBoolean("developer_mode", com.axiomloader.BuildConfig.DEBUG));
        binding.switchHotReload.setChecked(preferences.getBoolean("hot_reload", com.axiomloader.BuildConfig.DEBUG));
        binding.switchSecurityChecks.setChecked(preferences.getBoolean("security_checks", true));
        binding.switchAutoCleanup.setChecked(preferences.getBoolean("auto_cleanup", true));
        
        updateDeveloperFeatures(preferences.getBoolean("developer_mode", false));
        updateStatistics();
    }
    
    private void togglePluginSystem(boolean enabled) {
        new Thread(() -> {
            try {
                if (enabled) {
                    pluginManager.enablePluginSystem();
                } else {
                    pluginManager.disablePluginSystem();
                }
                
                runOnUiThread(() -> {
                    String message = enabled ? "Plugin system enabled" : "Plugin system disabled";
                    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                    updateStatistics();
                });
                
                LogUtils.logInfo(TAG, "Plugin system toggled: " + enabled);
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to toggle plugin system", e);
                runOnUiThread(() -> {
                    binding.switchPluginSystemEnabled.setChecked(!enabled);
                    Toast.makeText(this, "Failed to toggle plugin system", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }
    
    private void toggleDeveloperMode(boolean enabled) {
        preferences.edit().putBoolean("developer_mode", enabled).apply();
        Toast.makeText(this, "Developer mode " + (enabled ? "enabled" : "disabled"), Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Developer mode toggled: " + enabled);
        updateDeveloperFeatures(enabled);
    }
    
    private void toggleHotReload(boolean enabled) {
        preferences.edit().putBoolean("hot_reload", enabled).apply();
        Toast.makeText(this, "Hot reload " + (enabled ? "enabled" : "disabled"), Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Hot reload toggled: " + enabled);
    }
    
    private void toggleSecurityChecks(boolean enabled) {
        preferences.edit().putBoolean("security_checks", enabled).apply();
        Toast.makeText(this, "Security checks " + (enabled ? "enabled" : "disabled"), Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Security checks toggled: " + enabled);
        
        if (!enabled) {
            new androidx.appcompat.app.AlertDialog.Builder(this)
                .setTitle("Warning")
                .setMessage("Disabling security checks may expose your device to malicious plugins. Only disable this if you understand the risks.")
                .setPositiveButton("I Understand", null)
                .setIcon(android.R.drawable.ic_dialog_alert)
                .show();
        }
    }
    
    private void toggleAutoCleanup(boolean enabled) {
        preferences.edit().putBoolean("auto_cleanup", enabled).apply();
        Toast.makeText(this, "Auto cleanup " + (enabled ? "enabled" : "disabled"), Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Auto cleanup toggled: " + enabled);
    }
    
    private void updateDeveloperFeatures(boolean developerMode) {
        binding.switchHotReload.setEnabled(developerMode);
        binding.tvHotReloadDescription.setAlpha(developerMode ? 1.0f : 0.5f);
    }
    
    private void updateStatistics() {
        new Thread(() -> {
            try {
                int totalPlugins = pluginManager.getTotalPluginCount();
                int enabledPlugins = pluginManager.getEnabledPluginCount();
                int loadedPlugins = pluginManager.getLoadedPluginCount();
                
                runOnUiThread(() -> {
                    binding.tvTotalPlugins.setText("Total Plugins: " + totalPlugins);
                    binding.tvEnabledPlugins.setText("Enabled: " + enabledPlugins);
                    binding.tvLoadedPlugins.setText("Loaded: " + loadedPlugins);
                    
                    binding.tvPluginDirectory.setText("Plugin Directory:\n" + 
                        pluginManager.getPluginDirectory().getAbsolutePath());
                });
                
            } catch (Exception e) {
                LogUtils.logError(TAG, "Failed to update statistics", e);
            }
        }).start();
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "PluginSettingsActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginStoreActivity.java
package com.axiomloader.modding;

import android.os.Bundle;
import android.view.MenuItem;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import com.axiomloader.R;
import com.axiomloader.databinding.ActivityPluginStoreBinding;
import com.axiomloader.utils.LogUtils;

public class PluginStoreActivity extends AppCompatActivity {
    
    private ActivityPluginStoreBinding binding;
    private static final String TAG = "PluginStoreActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityPluginStoreBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.plugin_store);
        }
        
        setupUI();
        showPlaceholderContent();
        
        LogUtils.logInfo(TAG, "PluginStoreActivity created");
    }
    
    private void setupUI() {
        binding.recyclerViewStore.setLayoutManager(new LinearLayoutManager(this));
        
        binding.btnRefreshStore.setOnClickListener(v -> refreshStore());
    }
    
    private void showPlaceholderContent() {
        // Display placeholder message for future implementation
        binding.tvStoreMessage.setText(
            "Plugin Store Coming Soon!\n\n" +
            "The plugin marketplace will allow you to:\n\n" +
            "• Browse community plugins\n" +
            "• Download and install plugins with one click\n" +
            "• Rate and review plugins\n" +
            "• Check for plugin updates\n" +
            "• Submit your own plugins\n\n" +
            "For now, you can manually install plugins by:\n" +
            "1. Placing .dex or .apk files in the plugin directory\n" +
            "2. Creating a manifest.json file\n" +
            "3. Enabling the plugin in Plugin Manager"
        );
    }
    
    private void refreshStore() {
        Toast.makeText(this, "Plugin store refresh - coming soon!", Toast.LENGTH_SHORT).show();
        LogUtils.logInfo(TAG, "Store refresh requested");
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "PluginStoreActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/modding/PluginSystemInitializer.java
package com.axiomloader.modding;

import android.content.Context;
import android.os.Environment;
import com.axiomloader.BuildConfig;
import com.axiomloader.utils.LogUtils;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Arrays;

/**
 * PluginSystemInitializer - Sets up the plugin system environment
 * Creates directories, initializes configuration, and sets up the runtime environment
 */
public class PluginSystemInitializer {
    
    private static final String TAG = "PluginSystemInitializer";
    private final Context context;
    
    public PluginSystemInitializer(Context context) {
        this.context = context;
    }
    
    /**
     * Initialize the complete plugin system
     */
    public void initialize() throws Exception {
        LogUtils.logInfo(TAG, "Initializing plugin system...");
        
        try {
            // Create directory structure
            createDirectoryStructure();
            
            // Setup plugin environment
            setupPluginEnvironment();
            
            // Create sample plugin template
            if (BuildConfig.DEBUG) {
                createSamplePluginTemplate();
            }
            
            // Verify system integrity
            verifySystemIntegrity();
            
            LogUtils.logInfo(TAG, "Plugin system initialized successfully");
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to initialize plugin system", e);
            throw new Exception("Plugin system initialization failed", e);
        }
    }
    
    private void createDirectoryStructure() throws IOException {
        File baseDir = new File(context.getExternalFilesDir(null), "axiom_plugins");
        
        String[] directories = {
            "installed",      // Installed plugin files
            "temp",          // Temporary files during installation
            "data",          // Plugin data storage
            "cache",         // Plugin cache files  
            "logs",          // Plugin-specific logs
            "backups",       // Plugin backups
            "templates",     // Plugin development templates
            "store",         // Downloaded plugins from store
            "development",   // Development workspace
            "sandbox"        // Sandboxed execution environment
        };
        
        for (String dir : directories) {
            File directory = new File(baseDir, dir);
            if (!directory.exists()) {
                boolean created = directory.mkdirs();
                if (!created) {
                    throw new IOException("Failed to create directory: " + directory.getAbsolutePath());
                }
                LogUtils.logDebug(TAG, "Created directory: " + dir);
            }
        }
        
        // Create .nomedia file to prevent media scanning
        File nomediaFile = new File(baseDir, ".nomedia");
        if (!nomediaFile.exists()) {
            boolean created = nomediaFile.createNewFile();
            if (created) {
                LogUtils.logDebug(TAG, "Created .nomedia file");
            }
        }
    }
    
    private void setupPluginEnvironment() throws IOException {
        // Create plugin configuration file
        createPluginConfig();
        
        // Setup DEX optimization directories
        setupDexOptimization();
        
        // Initialize plugin registry
        initializePluginRegistry();
        
        // Create security policy file
        createSecurityPolicy();
    }
    
    private void createPluginConfig() throws IOException {
        File configFile = new File(context.getExternalFilesDir(null), "axiom_plugins/plugin_config.json");
        
        if (!configFile.exists()) {
            String configContent = "{\n" +
                    "  \"version\": \"1.0\",\n" +
                    "  \"api_version\": " + BuildConfig.PLUGIN_API_VERSION + ",\n" +
                    "  \"max_plugins\": 50,\n" +
                    "  \"max_plugin_size\": 52428800,\n" +
                    "  \"security_enabled\": true,\n" +
                    "  \"developer_mode\": " + BuildConfig.DEBUG + ",\n" +
                    "  \"hot_reload_enabled\": " + BuildConfig.DEBUG + ",\n" +
                    "  \"allowed_permissions\": [\n" +
                    "    \"axiom.permission.ACCESS_LOGS\",\n" +
                    "    \"axiom.permission.MODIFY_UI\",\n" +
                    "    \"axiom.permission.NETWORK_ACCESS\",\n" +
                    "    \"axiom.permission.FILE_ACCESS\"\n" +
                    "  ],\n" +
                    "  \"sandbox_enabled\": true,\n" +
                    "  \"update_check_interval\": 86400000,\n" +
                    "  \"backup_enabled\": true,\n" +
                    "  \"plugin_timeout\": 30000,\n" +
                    "  \"memory_limit_mb\": 256\n" +
                    "}";
            
            try (FileWriter writer = new FileWriter(configFile)) {
                writer.write(configContent);
            }
            
            LogUtils.logInfo(TAG, "Created plugin configuration file");
        }
    }
    
    private void setupDexOptimization() {
        File dexOptDir = new File(context.getCacheDir(), "plugin_dex");
        if (!dexOptDir.exists()) {
            boolean created = dexOptDir.mkdirs();
            if (!created) {
                LogUtils.logWarning(TAG, "Failed to create DEX optimization directory");
                return;
            }
        }
        
        // Set appropriate permissions for security
        try {
            dexOptDir.setReadable(true, true);  // Owner read only
            dexOptDir.setWritable(true, true);  // Owner write only
            dexOptDir.setExecutable(true, true); // Owner execute only
            
            LogUtils.logDebug(TAG, "DEX optimization directory setup complete");
        } catch (SecurityException e) {
            LogUtils.logError(TAG, "Could not set DEX directory permissions", e);
        }
    }
    
    private void initializePluginRegistry() throws IOException {
        File registryFile = new File(context.getExternalFilesDir(null), "axiom_plugins/plugin_registry.json");
        
        if (!registryFile.exists()) {
            String registryContent = "{\n" +
                    "  \"version\": \"1.0\",\n" +
                    "  \"last_updated\": " + System.currentTimeMillis() + ",\n" +
                    "  \"plugins\": [],\n" +
                    "  \"enabled_plugins\": [],\n" +
                    "  \"plugin_stats\": {\n" +
                    "    \"total_installs\": 0,\n" +
                    "    \"total_loads\": 0,\n" +
                    "    \"successful_loads\": 0,\n" +
                    "    \"failed_loads\": 0,\n" +
                    "    \"last_scan\": 0\n" +
                    "  },\n" +
                    "  \"system_info\": {\n" +
                    "    \"host_app_version\": \"" + BuildConfig.VERSION_NAME + "\",\n" +
                    "    \"api_version\": " + BuildConfig.PLUGIN_API_VERSION + ",\n" +
                    "    \"android_version\": " + android.os.Build.VERSION.SDK_INT + "\n" +
                    "  }\n" +
                    "}";
            
            try (FileWriter writer = new FileWriter(registryFile)) {
                writer.write(registryContent);
            }
            
            LogUtils.logInfo(TAG, "Created plugin registry file");
        }
    }
    
    private void createSecurityPolicy() throws IOException {
        File policyFile = new File(context.getExternalFilesDir(null), "axiom_plugins/security_policy.json");
        
        if (!policyFile.exists()) {
            String policyContent = "{\n" +
                    "  \"version\": \"1.0\",\n" +
                    "  \"security_level\": \"" + (BuildConfig.DEBUG ? "development" : "production") + "\",\n" +
                    "  \"signature_required\": " + (!BuildConfig.DEBUG) + ",\n" +
                    "  \"allowed_sources\": [\n" +
                    "    \"local_filesystem\",\n" +
                    "    \"official_store\"\n" +
                    "  ],\n" +
                    "  \"blocked_permissions\": [\n" +
                    "    \"android.permission.INSTALL_PACKAGES\",\n" +
                    "    \"android.permission.DELETE_PACKAGES\",\n" +
                    "    \"android.permission.SYSTEM_ALERT_WINDOW\"\n" +
                    "  ],\n" +
                    "  \"sandbox_restrictions\": {\n" +
                    "    \"network_access\": true,\n" +
                    "    \"file_system_access\": \"restricted\",\n" +
                    "    \"system_api_access\": false,\n" +
                    "    \"root_access\": false\n" +
                    "  },\n" +
                    "  \"resource_limits\": {\n" +
                    "    \"max_memory_mb\": 256,\n" +
                    "    \"max_cpu_time_ms\": 30000,\n" +
                    "    \"max_network_requests\": 100\n" +
                    "  }\n" +
                    "}";
            
            try (FileWriter writer = new FileWriter(policyFile)) {
                writer.write(policyContent);
            }
            
            LogUtils.logInfo(TAG, "Created security policy file");
        }
    }
    
    private void createSamplePluginTemplate() throws IOException {
        File templateDir = new File(context.getExternalFilesDir(null), "axiom_plugins/templates");
        File sampleTemplate = new File(templateDir, "sample_plugin_manifest.json");
        
        if (!sampleTemplate.exists()) {
            String templateContent = "{\n" +
                    "  \"id\": \"com.example.sample_plugin\",\n" +
                    "  \"name\": \"Sample Plugin\",\n" +
                    "  \"version\": \"1.0.0\",\n" +
                    "  \"author\": \"Plugin Developer\",\n" +
                    "  \"description\": \"A sample plugin demonstrating basic functionality\",\n" +
                    "  \"main_class\": \"com.example.SamplePlugin\",\n" +
                    "  \"api_version\": " + BuildConfig.PLUGIN_API_VERSION + ",\n" +
                    "  \"min_android_version\": 21,\n" +
                    "  \"permissions\": [\n" +
                    "    \"axiom.permission.ACCESS_LOGS\"\n" +
                    "  ],\n" +
                    "  \"features\": [\n" +
                    "    \"ui_modification\",\n" +
                    "    \"log_analysis\"\n" + 
                    "  ],\n" +
                    "  \"category\": \"utility\",\n" +
                    "  \"tags\": [\"sample\", \"demo\", \"utility\"],\n" +
                    "  \"hooks\": {\n" +
                    "    \"onApplicationStart\": \"onAppStart\",\n" +
                    "    \"onLogReceived\": \"handleLog\"\n" +
                    "  },\n" +
                    "  \"config\": {\n" +
                    "    \"enable_notifications\": true,\n" +
                    "    \"log_level\": \"INFO\",\n" +
                    "    \"auto_start\": false\n" +
                    "  }\n" +
                    "}";
            
            try (FileWriter writer = new FileWriter(sampleTemplate)) {
                writer.write(templateContent);
            }
            
            // Create Java template
            File javaTemplate = new File(templateDir, "SamplePlugin.java");
            String javaContent = "package com.example;\n\n" +
                    "import android.content.Context;\n" +
                    "import com.axiomloader.modding.PluginInterface;\n" +
                    "import java.util.HashMap;\n" +
                    "import java.util.Map;\n\n" +
                    "public class SamplePlugin implements PluginInterface {\n" +
                    "    private Context context;\n\n" +
                    "    @Override\n" +
                    "    public void onLoad(Context context) {\n" +
                    "        this.context = context;\n" +
                    "        // Plugin initialization code here\n" +
                    "    }\n\n" +
                    "    @Override\n" +
                    "    public void onUnload() {\n" +
                    "        // Plugin cleanup code here\n" +
                    "    }\n\n" +
                    "    @Override\n" +
                    "    public Map<String, Object> getPluginInfo() {\n" +
                    "        Map<String, Object> info = new HashMap<>();\n" +
                    "        info.put(\"name\", \"Sample Plugin\");\n" +
                    "        info.put(\"version\", \"1.0.0\");\n" +
                    "        return info;\n" +
                    "    }\n\n" +
                    "    @Override\n" +
                    "    public String getPluginName() {\n" +
                    "        return \"Sample Plugin\";\n" +
                    "    }\n\n" +
                    "    @Override\n" +
                    "    public String getPluginVersion() {\n" +
                    "        return \"1.0.0\";\n" +
                    "    }\n\n" +
                    "    @Override\n" +
                    "    public String getPluginDescription() {\n" +
                    "        return \"A sample plugin demonstrating basic functionality\";\n" +
                    "    }\n\n" +
                    "    // Custom plugin methods\n" +
                    "    public void onAppStart() {\n" +
                    "        // Called when app starts\n" +
                    "    }\n\n" +
                    "    public void handleLog(String logMessage) {\n" +
                    "        // Called when new log message arrives\n" +
                    "    }\n" +
                    "}";
            
            try (FileWriter writer = new FileWriter(javaTemplate)) {
                writer.write(javaContent);
            }
            
            LogUtils.logInfo(TAG, "Created plugin templates");
        }
    }
    
    private void verifySystemIntegrity() throws Exception {
        File baseDir = new File(context.getExternalFilesDir(null), "axiom_plugins");
        
        // Check if base directory exists and is writable
        if (!baseDir.exists() || !baseDir.canWrite()) {
            throw new Exception("Plugin base directory is not accessible");
        }
        
        // Verify required subdirectories
        String[] requiredDirs = {"installed", "temp", "data", "cache", "logs"};
        for (String dirName : requiredDirs) {
            File dir = new File(baseDir, dirName);
            if (!dir.exists() || !dir.canWrite()) {
                throw new Exception("Required directory not accessible: " + dirName);
            }
        }
        
        // Check DEX optimization directory
        File dexOptDir = new File(context.getCacheDir(), "plugin_dex");
        if (!dexOptDir.exists() || !dexOptDir.canWrite()) {
            throw new Exception("DEX optimization directory not accessible");
        }
        
        // Verify configuration files exist
        File configFile = new File(baseDir, "plugin_config.json");
        File registryFile = new File(baseDir, "plugin_registry.json");
        File policyFile = new File(baseDir, "security_policy.json");
        
        if (!configFile.exists() || !registryFile.exists() || !policyFile.exists()) {
            throw new Exception("Required configuration files missing");
        }
        
        LogUtils.logInfo(TAG, "System integrity verification passed");
    }
    
    /**
     * Clean up temporary files and optimize storage
     */
    public void performMaintenance() {
        LogUtils.logInfo(TAG, "Performing plugin system maintenance...");
        
        try {
            // Clean temporary files
            cleanTemporaryFiles();
            
            // Optimize cache
            optimizeCache();
            
            // Clean old logs
            cleanOldLogs();
            
            LogUtils.logInfo(TAG, "Plugin system maintenance completed");
            
        } catch (Exception e) {
            LogUtils.logError(TAG, "Maintenance failed", e);
        }
    }
    
    private void cleanTemporaryFiles() {
        File tempDir = new File(context.getExternalFilesDir(null), "axiom_plugins/temp");
        File[] tempFiles = tempDir.listFiles();
        
        if (tempFiles != null) {
            for (File file : tempFiles) {
                // Delete files older than 1 hour
                if (System.currentTimeMillis() - file.lastModified() > 3600000) {
                    boolean deleted = file.delete();
                    if (deleted) {
                        LogUtils.logDebug(TAG, "Deleted temp file: " + file.getName());
                    }
                }
            }
        }
    }
    
    private void optimizeCache() {
        File cacheDir = new File(context.getExternalFilesDir(null), "axiom_plugins/cache");
        File[] cacheFiles = cacheDir.listFiles();
        
        if (cacheFiles != null) {
            // Sort by last modified time
            Arrays.sort(cacheFiles, (f1, f2) -> Long.compare(f1.lastModified(), f2.lastModified()));
            
            // Keep only 20 most recent cache files
            for (int i = 0; i < cacheFiles.length - 20; i++) {
                boolean deleted = cacheFiles[i].delete();
                if (deleted) {
                    LogUtils.logDebug(TAG, "Deleted old cache file: " + cacheFiles[i].getName());
                }
            }
        }
    }
    
    private void cleanOldLogs() {
        File logsDir = new File(context.getExternalFilesDir(null), "axiom_plugins/logs");
        File[] logFiles = logsDir.listFiles();
        
        if (logFiles != null) {
            long weekAgo = System.currentTimeMillis() - (7 * 24 * 3600000);
            
            for (File logFile : logFiles) {
                if (logFile.lastModified() < weekAgo) {
                    boolean deleted = logFile.delete();
                    if (deleted) {
                        LogUtils.logDebug(TAG, "Deleted old log file: " + logFile.getName());
                    }
                }
            }
        }
    }
    
    /**
     * Reset the entire plugin system (for debugging/testing)
     */
    public void resetPluginSystem() throws IOException {
        LogUtils.logWarning(TAG, "Resetting plugin system - all data will be lost!");
        
        File baseDir = new File(context.getExternalFilesDir(null), "axiom_plugins");
        if (baseDir.exists()) {
            deleteDirectoryRecursively(baseDir);
        }
        
        File dexOptDir = new File(context.getCacheDir(), "plugin_dex");
        if (dexOptDir.exists()) {
            deleteDirectoryRecursively(dexOptDir);
        }
        
        // Re-initialize
        try {
            initialize();
        } catch (Exception e) {
            LogUtils.logError(TAG, "Failed to re-initialize plugin system after reset", e);
            throw new IOException("Failed to re-initialize plugin system after reset", e);
        }
        
        LogUtils.logInfo(TAG, "Plugin system reset completed");
    }
    
    private void deleteDirectoryRecursively(File dir) {
        File[] files = dir.listFiles();
        if (files != null) {
            for (File file : files) {
                if (file.isDirectory()) {
                    deleteDirectoryRecursively(file);
                } else {
                    file.delete();
                }
            }
        }
        dir.delete();
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/ui/LogAdapter.java
package com.axiomloader.ui;

import android.content.ClipData;
import android.content.ClipboardManager;
import android.content.Context;
import android.content.Intent;
import android.graphics.Color;
import android.text.SpannableString;
import android.text.style.ForegroundColorSpan;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.recyclerview.widget.RecyclerView;

import com.axiomloader.R;
import com.axiomloader.utils.LogUtils;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Locale;

public class LogAdapter extends RecyclerView.Adapter<LogAdapter.LogViewHolder> {
    
    private Context context;
    private List<LogUtils.LogEntry> logs;
    private SimpleDateFormat timeFormat;
    private SimpleDateFormat fullDateFormat;
    private OnLogActionListener actionListener;
    
    // Color scheme for different log levels
    private static final int COLOR_VERBOSE = Color.parseColor("#757575"); // Gray
    private static final int COLOR_DEBUG = Color.parseColor("#2196F3"); // Blue
    private static final int COLOR_INFO = Color.parseColor("#4CAF50"); // Green
    private static final int COLOR_WARN = Color.parseColor("#FF9800"); // Orange
    private static final int COLOR_ERROR = Color.parseColor("#F44336"); // Red
    private static final int COLOR_WTF = Color.parseColor("#9C27B0"); // Purple
    
    public interface OnLogActionListener {
        void onFilterByTag(String tag);
        void onFilterByLevel(LogUtils.LogLevel level);
    }
    
    public LogAdapter(Context context) {
        this.context = context;
        this.logs = new ArrayList<>();
        this.timeFormat = new SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault());
        this.fullDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS", Locale.getDefault());
    }
    
    public void setLogs(List<LogUtils.LogEntry> logs) {
        this.logs = logs != null ? logs : new ArrayList<>();
        notifyDataSetChanged();
    }
    
    public void addLog(LogUtils.LogEntry log) {
        logs.add(log);
        notifyItemInserted(logs.size() - 1);
    }
    
    public void clearLogs() {
        logs.clear();
        notifyDataSetChanged();
    }
    
    public void setOnLogActionListener(OnLogActionListener listener) {
        this.actionListener = listener;
    }
    
    @NonNull
    @Override
    public LogViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_log_entry, parent, false);
        return new LogViewHolder(view);
    }
    
    @Override
    public void onBindViewHolder(@NonNull LogViewHolder holder, int position) {
        LogUtils.LogEntry log = logs.get(position);
        holder.bind(log);
    }
    
    @Override
    public int getItemCount() {
        return logs.size();
    }
    
    private int getColorForLogLevel(String level) {
        switch (level) {
            case "VERBOSE":
                return COLOR_VERBOSE;
            case "DEBUG":
                return COLOR_DEBUG;
            case "INFO":
                return COLOR_INFO;
            case "WARN":
                return COLOR_WARN;
            case "ERROR":
                return COLOR_ERROR;
            case "WTF":
            case "ASSERT":
                return COLOR_WTF;
            default:
                return Color.BLACK;
        }
    }
    
    private String getShortLogLevel(String level) {
        switch (level) {
            case "VERBOSE": return "V";
            case "DEBUG": return "D";
            case "INFO": return "I";
            case "WARN": return "W";
            case "ERROR": return "E";
            case "WTF":
            case "ASSERT": return "A";
            default: return "?";
        }
    }
    
    private String getSimpleClassName(String fullClassName) {
        if (fullClassName == null) return "Unknown";
        int lastDot = fullClassName.lastIndexOf('.');
        return lastDot >= 0 ? fullClassName.substring(lastDot + 1) : fullClassName;
    }
    
    private LogUtils.LogLevel getLogLevelFromString(String levelString) {
        switch (levelString) {
            case "VERBOSE": return LogUtils.LogLevel.VERBOSE;
            case "DEBUG": return LogUtils.LogLevel.DEBUG;
            case "INFO": return LogUtils.LogLevel.INFO;
            case "WARN": return LogUtils.LogLevel.WARN;
            case "ERROR": return LogUtils.LogLevel.ERROR;
            case "WTF":
            case "ASSERT": return LogUtils.LogLevel.WTF;
            default: return LogUtils.LogLevel.DEBUG;
        }
    }
    
    class LogViewHolder extends RecyclerView.ViewHolder {
        
        private TextView tvTimestamp;
        private TextView tvLevel;
        private TextView tvTag;
        private TextView tvMessage;
        private TextView tvThread;
        private TextView tvLocation;
        private View levelIndicator;
        
        public LogViewHolder(@NonNull View itemView) {
            super(itemView);
            tvTimestamp = itemView.findViewById(R.id.tv_timestamp);
            tvLevel = itemView.findViewById(R.id.tv_level);
            tvTag = itemView.findViewById(R.id.tv_tag);
            tvMessage = itemView.findViewById(R.id.tv_message);
            tvThread = itemView.findViewById(R.id.tv_thread);
            tvLocation = itemView.findViewById(R.id.tv_location);
            levelIndicator = itemView.findViewById(R.id.level_indicator);
        }
        
        public void bind(LogUtils.LogEntry log) {
            // Set timestamp
            Date date = new Date(log.timestamp);
            tvTimestamp.setText(timeFormat.format(date));
            
            // Set log level with color
            String levelText = getShortLogLevel(log.level);
            tvLevel.setText(levelText);
            int levelColor = getColorForLogLevel(log.level);
            tvLevel.setTextColor(levelColor);
            levelIndicator.setBackgroundColor(levelColor);
            
            // Set tag with color
            SpannableString tagSpan = new SpannableString(log.tag);
            tagSpan.setSpan(new ForegroundColorSpan(levelColor), 0, log.tag.length(), 0);
            tvTag.setText(tagSpan);
            
            // Set message
            tvMessage.setText(log.message);
            
            // Set thread info if available
            if (log.threadName != null && !log.threadName.isEmpty()) {
                tvThread.setText("[" + log.threadName + "]");
                tvThread.setVisibility(View.VISIBLE);
                tvThread.setTextColor(Color.GRAY);
            } else {
                tvThread.setVisibility(View.GONE);
            }
            
            // Set location info if available
            if (log.className != null && log.methodName != null) {
                String className = getSimpleClassName(log.className);
                String location = className + "." + log.methodName + ":" + log.lineNumber;
                tvLocation.setText("(" + location + ")");
                tvLocation.setVisibility(View.VISIBLE);
                tvLocation.setTextColor(Color.GRAY);
            } else {
                tvLocation.setVisibility(View.GONE);
            }
            
            // Set click listener for expanded view
            itemView.setOnClickListener(v -> showLogDetails(log));
            
            // Long click for actions menu
            itemView.setOnLongClickListener(v -> {
                showLogActionMenu(log);
                return true;
            });
        }
        
        private void showLogDetails(LogUtils.LogEntry log) {
            StringBuilder details = new StringBuilder();
            
            details.append("TIMESTAMP: ").append(fullDateFormat.format(new Date(log.timestamp))).append("\n");
            details.append("LEVEL: ").append(log.level).append("\n");
            details.append("TAG: ").append(log.tag).append("\n");
            details.append("MESSAGE: ").append(log.message).append("\n");
            
            if (log.threadName != null && !log.threadName.isEmpty()) {
                details.append("THREAD: ").append(log.threadName).append("\n");
            }
            
            if (log.className != null) {
                details.append("CLASS: ").append(log.className).append("\n");
                details.append("METHOD: ").append(log.methodName).append("\n");
                details.append("LINE: ").append(log.lineNumber).append("\n");
            }
            
            if (log.metadata != null && !log.metadata.isEmpty()) {
                details.append("\nMETADATA:\n");
                for (String key : log.metadata.keySet()) {
                    Object value = log.metadata.get(key);
                    details.append("  ").append(key).append(": ").append(value != null ? value.toString() : "null").append("\n");
                }
            }
            
            new AlertDialog.Builder(context)
                .setTitle("Log Entry Details")
                .setMessage(details.toString())
                .setPositiveButton("Close", null)
                .setNeutralButton("Copy", (dialog, which) -> {
                    copyToClipboard("Log Entry Details", details.toString());
                })
                .setNegativeButton("Share", (dialog, which) -> {
                    shareText("Log Entry Details", details.toString());
                })
                .show();
        }
        
        private void showLogActionMenu(LogUtils.LogEntry log) {
            String[] actions = {"Copy Message", "Copy Full Log", "Filter by Tag", "Filter by Level", "Share Log", "Show Details"};
            
            new AlertDialog.Builder(context)
                .setTitle("Log Actions")
                .setItems(actions, (dialog, which) -> {
                    switch (which) {
                        case 0: // Copy Message
                            copyToClipboard("Log Message", log.message);
                            break;
                        case 1: // Copy Full Log
                            String fullLog = String.format("%s %s/%s: %s", 
                                fullDateFormat.format(new Date(log.timestamp)),
                                getShortLogLevel(log.level), log.tag, log.message);
                            copyToClipboard("Full Log", fullLog);
                            break;
                        case 2: // Filter by Tag
                            if (actionListener != null) {
                                actionListener.onFilterByTag(log.tag);
                            } else {
                                Toast.makeText(context, "Filter by tag: " + log.tag, Toast.LENGTH_SHORT).show();
                            }
                            break;
                        case 3: // Filter by Level
                            if (actionListener != null) {
                                LogUtils.LogLevel level = getLogLevelFromString(log.level);
                                actionListener.onFilterByLevel(level);
                            } else {
                                Toast.makeText(context, "Filter by level: " + log.level, Toast.LENGTH_SHORT).show();
                            }
                            break;
                        case 4: // Share Log
                            shareLog(log);
                            break;
                        case 5: // Show Details
                            showLogDetails(log);
                            break;
                    }
                })
                .show();
        }
        
        private void copyToClipboard(String label, String text) {
            ClipboardManager clipboard = (ClipboardManager) context.getSystemService(Context.CLIPBOARD_SERVICE);
            ClipData clip = ClipData.newPlainText(label, text);
            clipboard.setPrimaryClip(clip);
            Toast.makeText(context, "Copied to clipboard", Toast.LENGTH_SHORT).show();
        }
        
        private void shareLog(LogUtils.LogEntry log) {
            StringBuilder shareText = new StringBuilder();
            shareText.append("=== AXIOM LOG ENTRY ===\n");
            shareText.append("Time: ").append(fullDateFormat.format(new Date(log.timestamp))).append("\n");
            shareText.append("Level: ").append(log.level).append("\n");
            shareText.append("Tag: ").append(log.tag).append("\n");
            shareText.append("Message: ").append(log.message).append("\n");
            
            if (log.threadName != null && !log.threadName.isEmpty()) {
                shareText.append("Thread: ").append(log.threadName).append("\n");
            }
            
            if (log.className != null) {
                shareText.append("Location: ").append(getSimpleClassName(log.className))
                         .append(".").append(log.methodName).append(":").append(log.lineNumber).append("\n");
            }
            
            if (log.metadata != null && !log.metadata.isEmpty()) {
                shareText.append("\nMetadata:\n");
                for (String key : log.metadata.keySet()) {
                    Object value = log.metadata.get(key);
                    shareText.append("- ").append(key).append(": ").append(value != null ? value.toString() : "null").append("\n");
                }
            }
            
            shareText(log.level + " Log from " + log.tag, shareText.toString());
        }
        
        private void shareText(String subject, String text) {
            Intent shareIntent = new Intent(Intent.ACTION_SEND);
            shareIntent.setType("text/plain");
            shareIntent.putExtra(Intent.EXTRA_TEXT, text);
            shareIntent.putExtra(Intent.EXTRA_SUBJECT, subject);
            context.startActivity(Intent.createChooser(shareIntent, "Share Log"));
        }
    }
    
    /**
     * Get filtered logs count
     */
    public int getFilteredCount() {
        return logs.size();
    }
    
    /**
     * Get log at specific position
     */
    public LogUtils.LogEntry getLogAt(int position) {
        if (position >= 0 && position < logs.size()) {
            return logs.get(position);
        }
        return null;
    }
    
    /**
     * Update single log entry
     */
    public void updateLog(int position, LogUtils.LogEntry log) {
        if (position >= 0 && position < logs.size()) {
            logs.set(position, log);
            notifyItemChanged(position);
        }
    }
    
    /**
     * Remove log at position
     */
    public void removeLog(int position) {
        if (position >= 0 && position < logs.size()) {
            logs.remove(position);
            notifyItemRemoved(position);
        }
    }
    
    /**
     * Insert log at specific position
     */
    public void insertLog(int position, LogUtils.LogEntry log) {
        if (position >= 0 && position <= logs.size()) {
            logs.add(position, log);
            notifyItemInserted(position);
        }
    }
    
    /**
     * Get all logs
     */
    public List<LogUtils.LogEntry> getAllLogs() {
        return new ArrayList<>(logs);
    }
    
    /**
     * Check if adapter is empty
     */
    public boolean isEmpty() {
        return logs.isEmpty();
    }
    
    /**
     * Get logs by level
     */
    public List<LogUtils.LogEntry> getLogsByLevel(String level) {
        List<LogUtils.LogEntry> filteredLogs = new ArrayList<>();
        for (LogUtils.LogEntry log : logs) {
            if (log.level.equals(level)) {
                filteredLogs.add(log);
            }
        }
        return filteredLogs;
    }
    
    /**
     * Get logs by tag
     */
    public List<LogUtils.LogEntry> getLogsByTag(String tag) {
        List<LogUtils.LogEntry> filteredLogs = new ArrayList<>();
        for (LogUtils.LogEntry log : logs) {
            if (log.tag.equals(tag)) {
                filteredLogs.add(log);
            }
        }
        return filteredLogs;
    }
    
    /**
     * Search logs by message
     */
    public List<LogUtils.LogEntry> searchLogs(String query) {
        List<LogUtils.LogEntry> searchResults = new ArrayList<>();
        String lowerQuery = query.toLowerCase();
        
        for (LogUtils.LogEntry log : logs) {
            if (log.message.toLowerCase().contains(lowerQuery) || 
                log.tag.toLowerCase().contains(lowerQuery)) {
                searchResults.add(log);
            }
        }
        return searchResults;
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/ui/LoggingActivity.java
package com.axiomloader.ui;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Toast;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;
import androidx.recyclerview.widget.LinearLayoutManager;

import com.axiomloader.R;
import com.axiomloader.databinding.ActivityLoggingBinding;
import com.axiomloader.utils.LogUtils;
import com.google.android.material.snackbar.Snackbar;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Locale;
import java.util.Map;

public class LoggingActivity extends AppCompatActivity {
    
    private ActivityLoggingBinding binding;
    private LogAdapter logAdapter;
    private LogUtils logUtils;
    private List<LogUtils.LogEntry> allLogs;
    private List<LogUtils.LogEntry> filteredLogs;
    private LogUtils.LogLevel selectedLevel = null;
    private String selectedTag = null;
    private String searchQuery = "";
    
    private static final String TAG = "LoggingActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityLoggingBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        setSupportActionBar(binding.toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle("Advanced Logging System");
        }
        
        logUtils = LogUtils.getInstance();
        initializeViews();
        setupRecyclerView();
        setupSpinners();
        setupSearchView();
        setupButtons();
        loadLogs();
        updateStats();
        
        LogUtils.logInfo(TAG, "LoggingActivity created");
    }
    
    private void initializeViews() {
        // Setup log level spinner
        ArrayAdapter<String> levelAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item,
            new String[]{"All Levels", "VERBOSE", "DEBUG", "INFO", "WARN", "ERROR", "WTF"});
        levelAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        binding.spinnerLogLevel.setAdapter(levelAdapter);
        
        // Setup tag spinner
        updateTagSpinner();
        
        // Set default values
        binding.etLogTag.setText("CUSTOM");
        binding.etLogMessage.setHint("Enter your log message here...");
    }
    
    private void setupRecyclerView() {
        logAdapter = new LogAdapter(this);
        binding.recyclerViewLogs.setLayoutManager(new LinearLayoutManager(this));
        binding.recyclerViewLogs.setAdapter(logAdapter);
        
        // Auto-scroll to bottom when new logs are added
        logAdapter.registerAdapterDataObserver(new androidx.recyclerview.widget.RecyclerView.AdapterDataObserver() {
            @Override
            public void onItemRangeInserted(int positionStart, int itemCount) {
                binding.recyclerViewLogs.scrollToPosition(logAdapter.getItemCount() - 1);
            }
        });
    }
    
    private void setupSpinners() {
        binding.spinnerLogLevel.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                if (position == 0) {
                    selectedLevel = null;
                } else {
                    selectedLevel = LogUtils.LogLevel.values()[position - 1];
                }
                filterLogs();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
        
        binding.spinnerTag.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                if (position == 0) {
                    selectedTag = null;
                } else {
                    selectedTag = (String) parent.getItemAtPosition(position);
                }
                filterLogs();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
        
        // Setup custom log level spinner
        ArrayAdapter<String> customLevelAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item,
            new String[]{"VERBOSE", "DEBUG", "INFO", "WARN", "ERROR", "WTF"});
        customLevelAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        binding.spinnerCustomLogLevel.setAdapter(customLevelAdapter);
        binding.spinnerCustomLogLevel.setSelection(2); // Default to INFO
    }
    
    private void setupSearchView() {
        binding.etSearchLogs.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                searchQuery = s.toString();
                filterLogs();
            }
            
            @Override
            public void afterTextChanged(Editable s) {}
        });
    }
    
    private void setupButtons() {
        binding.btnSendLog.setOnClickListener(v -> sendCustomLog());
        binding.btnClearLogs.setOnClickListener(v -> showClearLogsDialog());
        binding.btnExportLogs.setOnClickListener(v -> showExportDialog());
        binding.btnRefresh.setOnClickListener(v -> refreshLogs());
        binding.btnSettings.setOnClickListener(v -> showSettingsDialog());
        binding.btnStats.setOnClickListener(v -> showStatsDialog());
        binding.btnPerformanceTest.setOnClickListener(v -> runPerformanceTest());
        binding.btnGenerateTestLogs.setOnClickListener(v -> generateTestLogs());
    }
    
    private void loadLogs() {
        new Thread(() -> {
            allLogs = logUtils.getAllLogEntries();
            runOnUiThread(() -> {
                filterLogs();
                binding.tvLogCount.setText("Total Logs: " + allLogs.size());
            });
        }).start();
    }
    
    private void filterLogs() {
        if (allLogs == null) return;
        
        filteredLogs = new ArrayList<>();
        
        for (LogUtils.LogEntry entry : allLogs) {
            boolean matches = true;
            
            // Filter by level
            if (selectedLevel != null && !entry.level.equals(selectedLevel.fullName)) {
                matches = false;
            }
            
            // Filter by tag
            if (selectedTag != null && !entry.tag.equals(selectedTag)) {
                matches = false;
            }
            
            // Filter by search query
            if (!searchQuery.isEmpty() && 
                !entry.message.toLowerCase().contains(searchQuery.toLowerCase()) &&
                !entry.tag.toLowerCase().contains(searchQuery.toLowerCase())) {
                matches = false;
            }
            
            if (matches) {
                filteredLogs.add(entry);
            }
        }
        
        logAdapter.setLogs(filteredLogs);
        binding.tvFilteredCount.setText("Filtered: " + filteredLogs.size());
    }
    
    private void updateTagSpinner() {
        List<String> tags = new ArrayList<>();
        tags.add("All Tags");
        tags.addAll(logUtils.getActiveTags());
        
        ArrayAdapter<String> tagAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item, tags);
        tagAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        binding.spinnerTag.setAdapter(tagAdapter);
    }
    
    private void sendCustomLog() {
        String message = binding.etLogMessage.getText().toString().trim();
        String tag = binding.etLogTag.getText().toString().trim();
        
        if (message.isEmpty()) {
            Toast.makeText(this, "Please enter a log message", Toast.LENGTH_SHORT).show();
            return;
        }
        
        if (tag.isEmpty()) {
            tag = "CUSTOM";
        }
        
        int levelPosition = binding.spinnerCustomLogLevel.getSelectedItemPosition();
        LogUtils.LogLevel level = LogUtils.LogLevel.values()[levelPosition];
        
        // Send the log
        switch (level) {
            case VERBOSE:
                LogUtils.logVerbose(tag, message);
                break;
            case DEBUG:
                LogUtils.logDebug(tag, message);
                break;
            case INFO:
                LogUtils.logInfo(tag, message);
                break;
            case WARN:
                LogUtils.logWarning(tag, message);
                break;
            case ERROR:
                LogUtils.logError(tag, message);
                break;
            case WTF:
                LogUtils.logWtf(tag, message);
                break;
        }
        
        // Clear the message field
        binding.etLogMessage.setText("");
        
        // Refresh logs
        refreshLogs();
        
        // Show confirmation
        Snackbar.make(binding.getRoot(), "Log sent: " + level.shortName + "/" + tag, 
            Snackbar.LENGTH_SHORT).show();
    }
    
    private void refreshLogs() {
        loadLogs();
        updateStats();
        updateTagSpinner();
        LogUtils.logDebug(TAG, "Logs refreshed");
    }
    
    private void updateStats() {
        new Thread(() -> {
            Map<String, Object> stats = logUtils.getLogStatistics();
            runOnUiThread(() -> {
                binding.tvMemoryUsage.setText("Memory: " + stats.get("memory_usage_mb") + "MB");
                binding.tvCpuUsage.setText("CPU: " + stats.get("cpu_usage_percent") + "%");
                binding.tvBatteryLevel.setText("Battery: " + stats.get("battery_level") + "%");
                binding.tvNetworkStatus.setText("Network: " + stats.get("network_status"));
                binding.tvStorageAvailable.setText("Storage: " + stats.get("available_storage_gb") + "GB");
                binding.tvActiveTags.setText("Active Tags: " + stats.get("active_tags"));
            });
        }).start();
    }
    
    private void showClearLogsDialog() {
        new AlertDialog.Builder(this)
            .setTitle("Clear All Logs")
            .setMessage("Are you sure you want to clear all logs? This action cannot be undone.")
            .setPositiveButton("Clear", (dialog, which) -> {
                logUtils.clearAllLogs();
                loadLogs();
                Toast.makeText(this, "All logs cleared", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void showExportDialog() {
        String[] formats = {"Text", "JSON", "XML", "CSV"};
        LogUtils.LogFormat[] logFormats = {
            LogUtils.LogFormat.SIMPLE,
            LogUtils.LogFormat.JSON,
            LogUtils.LogFormat.XML,
            LogUtils.LogFormat.CSV
        };
        
        new AlertDialog.Builder(this)
            .setTitle("Export Logs")
            .setItems(formats, (dialog, which) -> exportLogs(logFormats[which]))
            .show();
    }
    
    private void exportLogs(LogUtils.LogFormat format) {
        new Thread(() -> {
            try {
                File exportFile = logUtils.exportLogs(format);
                
                runOnUiThread(() -> {
                    // Share the exported file
                    Intent shareIntent = new Intent(Intent.ACTION_SEND);
                    Uri fileUri = FileProvider.getUriForFile(this,
                        "com.axiomloader.fileprovider", exportFile);
                    shareIntent.setType("text/*");
                    shareIntent.putExtra(Intent.EXTRA_STREAM, fileUri);
                    shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
                    
                    startActivity(Intent.createChooser(shareIntent, "Share Log Export"));
                    
                    Toast.makeText(this, "Logs exported: " + exportFile.getName(), 
                        Toast.LENGTH_LONG).show();
                });
                
            } catch (Exception e) {
                runOnUiThread(() -> 
                    Toast.makeText(this, "Export failed: " + e.getMessage(), 
                        Toast.LENGTH_LONG).show());
                LogUtils.logError(TAG, "Export failed", e);
            }
        }).start();
    }
    
    private void showSettingsDialog() {
        View settingsView = getLayoutInflater().inflate(R.layout.dialog_log_settings, null);
        
        // Get current settings
        androidx.appcompat.widget.SwitchCompat switchFileLogging = settingsView.findViewById(R.id.switch_file_logging);
        androidx.appcompat.widget.SwitchCompat switchCompression = settingsView.findViewById(R.id.switch_compression);
        androidx.appcompat.widget.SwitchCompat switchEncryption = settingsView.findViewById(R.id.switch_encryption);
        androidx.appcompat.widget.SwitchCompat switchCrashReporting = settingsView.findViewById(R.id.switch_crash_reporting);
        androidx.appcompat.widget.SwitchCompat switchPerformanceMonitoring = settingsView.findViewById(R.id.switch_performance_monitoring);
        androidx.appcompat.widget.SwitchCompat switchAutoCleanup = settingsView.findViewById(R.id.switch_auto_cleanup);
        
        switchFileLogging.setChecked(logUtils.isFileLoggingEnabled());
        switchCompression.setChecked(logUtils.isCompressionEnabled());
        switchEncryption.setChecked(logUtils.isEncryptionEnabled());
        switchCrashReporting.setChecked(logUtils.isCrashReportingEnabled());
        switchPerformanceMonitoring.setChecked(logUtils.isPerformanceMonitoringEnabled());
        switchAutoCleanup.setChecked(logUtils.isAutoCleanupEnabled());
        
        new AlertDialog.Builder(this)
            .setTitle("Log Settings")
            .setView(settingsView)
            .setPositiveButton("Save", (dialog, which) -> {
                // Apply settings
                logUtils.setFileLoggingEnabled(switchFileLogging.isChecked());
                logUtils.setCompressionEnabled(switchCompression.isChecked());
                logUtils.setEncryptionEnabled(switchEncryption.isChecked());
                logUtils.setCrashReportingEnabled(switchCrashReporting.isChecked());
                logUtils.setPerformanceMonitoringEnabled(switchPerformanceMonitoring.isChecked());
                logUtils.setAutoCleanupEnabled(switchAutoCleanup.isChecked());
                
                Toast.makeText(this, "Settings saved", Toast.LENGTH_SHORT).show();
                LogUtils.logInfo(TAG, "Log settings updated");
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void showStatsDialog() {
        new Thread(() -> {
            String report = logUtils.generateLogReport();
            runOnUiThread(() -> {
                new AlertDialog.Builder(this)
                    .setTitle("Log Statistics Report")
                    .setMessage(report)
                    .setPositiveButton("Share", (dialog, which) -> {
                        Intent shareIntent = new Intent(Intent.ACTION_SEND);
                        shareIntent.setType("text/plain");
                        shareIntent.putExtra(Intent.EXTRA_TEXT, report);
                        shareIntent.putExtra(Intent.EXTRA_SUBJECT, "Axiom Log Report");
                        startActivity(Intent.createChooser(shareIntent, "Share Report"));
                    })
                    .setNegativeButton("Close", null)
                    .show();
            });
        }).start();
    }
    
    private void runPerformanceTest() {
        new Thread(() -> {
            runOnUiThread(() -> {
                binding.progressBar.setVisibility(View.VISIBLE);
                Toast.makeText(this, "Running performance test...", Toast.LENGTH_SHORT).show();
            });
            
            long startTime = System.currentTimeMillis();
            
            // Generate various types of logs
            for (int i = 0; i < 100; i++) {
                LogUtils.logVerbose("PERF_TEST", "Verbose log message " + i);
                LogUtils.logDebug("PERF_TEST", "Debug log message " + i);
                LogUtils.logInfo("PERF_TEST", "Info log message " + i);
                LogUtils.logWarning("PERF_TEST", "Warning log message " + i);
                LogUtils.logError("PERF_TEST", "Error log message " + i);
                
                if (i % 20 == 0) {
                    // Log some performance metrics
                    LogUtils.logMethodEntry("TestClass", "testMethod" + i);
                    try { Thread.sleep(10); } catch (InterruptedException e) {}
                    LogUtils.logMethodExit("TestClass", "testMethod" + i);
                }
            }
            
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            runOnUiThread(() -> {
                binding.progressBar.setVisibility(View.GONE);
                LogUtils.logInfo(TAG, "Performance test completed in " + duration + "ms (500 logs)");
                Toast.makeText(this, "Performance test completed in " + duration + "ms", 
                    Toast.LENGTH_LONG).show();
                refreshLogs();
            });
        }).start();
    }
    
    private void generateTestLogs() {
        new Thread(() -> {
            String[] testTags = {"UI", "NETWORK", "DATABASE", "AUTH", "CACHE"};
            String[] testMessages = {
                "User clicked button",
                "Network request completed",
                "Database query executed",
                "User authentication successful",
                "Cache hit for key"
            };
            
            for (int i = 0; i < 50; i++) {
                String tag = testTags[i % testTags.length];
                String message = testMessages[i % testMessages.length] + " #" + i;
                
                LogUtils.LogLevel level = LogUtils.LogLevel.values()[i % LogUtils.LogLevel.values().length];
                
                switch (level) {
                    case VERBOSE:
                        LogUtils.logVerbose(tag, message);
                        break;
                    case DEBUG:
                        LogUtils.logDebug(tag, message);
                        break;
                    case INFO:
                        LogUtils.logInfo(tag, message);
                        break;
                    case WARN:
                        LogUtils.logWarning(tag, message);
                        break;
                    case ERROR:
                        LogUtils.logError(tag, message);
                        break;
                    case WTF:
                        LogUtils.logWtf(tag, message);
                        break;
                }
                
                try { Thread.sleep(50); } catch (InterruptedException e) {}
            }
            
            runOnUiThread(() -> {
                Toast.makeText(this, "Generated 50 test logs", Toast.LENGTH_SHORT).show();
                refreshLogs();
            });
        }).start();
    }
    
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.logging_menu, menu);
        return true;
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        
        if (id == android.R.id.home) {
            finish();
            return true;
        } else if (id == R.id.action_refresh) {
            refreshLogs();
            return true;
        } else if (id == R.id.action_clear_search) {
            binding.etSearchLogs.setText("");
            binding.spinnerLogLevel.setSelection(0);
            binding.spinnerTag.setSelection(0);
            return true;
        }
        
        return super.onOptionsItemSelected(item);
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        refreshLogs();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        binding = null;
        LogUtils.logInfo(TAG, "LoggingActivity destroyed");
    }
}

// FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogUtils.java
package com.axiomloader.utils;

import android.app.ActivityManager;
import android.content.Context;
import android.content.SharedPreferences;
import android.os.BatteryManager;
import android.os.Build;
import android.os.Environment;
import android.os.Handler;
import android.os.Looper;
import android.os.StatFs;
import android.security.keystore.KeyGenParameterSpec;
import android.security.keystore.KeyProperties;
import android.util.Log;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import org.apache.commons.lang3.StringUtils;
import timber.log.Timber;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.security.KeyStore;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.regex.Pattern;
import java.util.zip.GZIPOutputStream;
import javax.crypto.Cipher;
import javax.crypto.CipherOutputStream;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;

/**
 * LogUtils - Advanced Timber-based Logging System
 * Features 20+ sophisticated offline logging capabilities
 * Total: 1000+ lines of code
 *
 * FEATURES:
 * 1. Multi-level logging (VERBOSE, DEBUG, INFO, WARN, ERROR, WTF)
 * 2. File-based logging with rotation
 * 3. Log compression (GZIP)
 * 4. Log encryption (AES-256)
 * 5. Performance monitoring
 * 6. Memory usage tracking
 * 7. CPU usage monitoring (Removed due to API limitations and inefficiency)
 * 8. Battery level monitoring
 * 9. Network status tracking
 * 10. Storage space monitoring
 * 11. Crash detection and logging
 * 12. Stack trace analysis
 * 13. Thread information logging
 * 14. Custom log formatting
 * 15. Log filtering and searching
 * 16. Log export functionality
 * 17. Auto-cleanup of old logs
 * 18. Log statistics and analytics
 * 19. Custom log tags management
 * 20. Real-time log monitoring
 * 21. Log visualization helpers
 * 22. Device information logging
 * 23. App lifecycle logging
 * 24. Network request logging
 * 25. Database operation logging
 */
public class LogUtils {

    // Constants
    private static final String TAG = "LogUtils";
    private static final String LOG_DIR = "axiom_logs";
    private static final String PREFS_NAME = "axiom_log_prefs";
    private static final String LOG_FILE_PREFIX = "axiom_log_";
    private static final String LOG_FILE_EXTENSION = ".log";
    private static final String COMPRESSED_EXTENSION = ".gz";
    private static final String ENCRYPTED_EXTENSION = ".enc";
    private static final int MAX_LOG_FILES = 10;
    private static final long MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
    private static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss.SSS";
    private static final String FILE_DATE_FORMAT = "yyyyMMdd_HHmmss";
    private static final String ENCRYPTION_ALGORITHM = "AES";
    private static final String ENCRYPTION_TRANSFORMATION_GCM = "AES/GCM/NoPadding";
    private static final String ENCRYPTION_KEY_ALIAS = "axiom_log_key";
    private static final int GCM_IV_LENGTH = 12; // 12 bytes for AES/GCM

    // Singleton instance
    private static LogUtils instance;
    private static Context applicationContext;

    // Core components
    private final ExecutorService executorService;
    private final Handler mainHandler;
    private final SharedPreferences preferences;
    private final SimpleDateFormat dateFormat;
    private final SimpleDateFormat fileDateFormat;
    private final Gson gson;
    private final AtomicInteger logCounter;

    // Configuration
    private boolean fileLoggingEnabled = true;
    private boolean compressionEnabled = false;
    private boolean encryptionEnabled = false;
    private boolean crashReportingEnabled = true;
    private boolean performanceMonitoringEnabled = true;
    private boolean autoCleanupEnabled = true;
    private boolean coloredLogsEnabled = true;
    private boolean threadInfoEnabled = true;
    private boolean stackTraceEnabled = false;
    private int maxLogFiles = MAX_LOG_FILES;
    private long maxFileSize = MAX_FILE_SIZE;

    // Monitoring data
    private final Map<String, Long> performanceMetrics;
    private final Map<String, Integer> logCounts;
    private final List<LogEntry> recentLogs;
    private final Set<String> activeTags;
    private long lastCleanupTime;
    private SecretKey encryptionKey;

    // Current log file
    private File currentLogFile;
    private BufferedWriter currentLogWriter;
    private long currentFileSize;

    /**
     * LogEntry class to represent individual log entries
     */
    public static class LogEntry {
        public long timestamp;
        public String level;
        public String tag;
        public String message;
        public String threadName;
        public String className;
        public String methodName;
        public int lineNumber;
        public Map<String, Object> metadata;

        public LogEntry(String level, String tag, String message) {
            this.timestamp = System.currentTimeMillis();
            this.level = level;
            this.tag = tag;
            this.message = message;
            this.threadName = Thread.currentThread().getName();
            this.metadata = new HashMap<>();

            // Extract caller information
            StackTraceElement[] stack = Thread.currentThread().getStackTrace();
            if (stack.length > 4) {
                StackTraceElement caller = stack[4];
                this.className = caller.getClassName();
                this.methodName = caller.getMethodName();
                this.lineNumber = caller.getLineNumber();
            }
        }
    }

    /**
     * LogLevel enumeration
     */
    public enum LogLevel {
        VERBOSE(2, "V", "VERBOSE"),
        DEBUG(3, "D", "DEBUG"),
        INFO(4, "I", "INFO"),
        WARN(5, "W", "WARN"),
        ERROR(6, "E", "ERROR"),
        WTF(7, "A", "ASSERT");

        public final int priority;
        public final String shortName;
        public final String fullName;

        LogLevel(int priority, String shortName, String fullName) {
            this.priority = priority;
            this.shortName = shortName;
            this.fullName = fullName;
        }
    }

    /**
     * LogFormat enumeration for different formatting styles
     */
    public enum LogFormat {
        SIMPLE,
        DETAILED,
        JSON,
        XML,
        CSV
    }

    /**
     * Private constructor for singleton pattern
     */
    private LogUtils(Context context) {
        applicationContext = context.getApplicationContext();
        executorService = Executors.newSingleThreadExecutor();
        mainHandler = new Handler(Looper.getMainLooper());
        preferences = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        dateFormat = new SimpleDateFormat(DATE_FORMAT, Locale.getDefault());
        fileDateFormat = new SimpleDateFormat(FILE_DATE_FORMAT, Locale.getDefault());
        gson = new GsonBuilder().setPrettyPrinting().create();
        logCounter = new AtomicInteger(0);

        performanceMetrics = new ConcurrentHashMap<>();
        logCounts = new ConcurrentHashMap<>();
        recentLogs = Collections.synchronizedList(new ArrayList<>());
        activeTags = Collections.synchronizedSet(new HashSet<>());

        loadPreferences();
        initializeEncryption();
        initializeLogging();
        setupCustomTimberTree();

        // Start performance monitoring
        if (performanceMonitoringEnabled) {
            startPerformanceMonitoring();
        }

        // Schedule cleanup if enabled
        if (autoCleanupEnabled) {
            scheduleLogCleanup();
        }

        // Log initialization
        logInfo(TAG, "LogUtils initialized with " + getFeatureCount() + " features");
        logDeviceInfo();
    }

    /**
     * Initialize the LogUtils singleton
     */
    public static synchronized void initialize(Context context) {
        if (instance == null) {
            instance = new LogUtils(context);
        }
    }

    /**
     * Get the singleton instance
     */
    public static LogUtils getInstance() {
        if (instance == null) {
            throw new IllegalStateException("LogUtils not initialized. Call initialize(Context) first.");
        }
        return instance;
    }

    /**
     * Load preferences from SharedPreferences
     */
    private void loadPreferences() {
        fileLoggingEnabled = preferences.getBoolean("file_logging_enabled", true);
        compressionEnabled = preferences.getBoolean("compression_enabled", false);
        encryptionEnabled = preferences.getBoolean("encryption_enabled", false);
        crashReportingEnabled = preferences.getBoolean("crash_reporting_enabled", true);
        performanceMonitoringEnabled = preferences.getBoolean("performance_monitoring_enabled", true);
        autoCleanupEnabled = preferences.getBoolean("auto_cleanup_enabled", true);
        coloredLogsEnabled = preferences.getBoolean("colored_logs_enabled", true);
        threadInfoEnabled = preferences.getBoolean("thread_info_enabled", true);
        stackTraceEnabled = preferences.getBoolean("stack_trace_enabled", false);
        maxLogFiles = preferences.getInt("max_log_files", MAX_LOG_FILES);
        maxFileSize = preferences.getLong("max_file_size", MAX_FILE_SIZE);
    }

    /**
     * Save preferences to SharedPreferences
     */
    private void savePreferences() {
        preferences.edit()
            .putBoolean("file_logging_enabled", fileLoggingEnabled)
            .putBoolean("compression_enabled", compressionEnabled)
            .putBoolean("encryption_enabled", encryptionEnabled)
            .putBoolean("crash_reporting_enabled", crashReportingEnabled)
            .putBoolean("performance_monitoring_enabled", performanceMonitoringEnabled)
            .putBoolean("auto_cleanup_enabled", autoCleanupEnabled)
            .putBoolean("colored_logs_enabled", coloredLogsEnabled)
            .putBoolean("thread_info_enabled", threadInfoEnabled)
            .putBoolean("stack_trace_enabled", stackTraceEnabled)
            .putInt("max_log_files", maxLogFiles)
            .putLong("max_file_size", maxFileSize)
            .apply();
    }

    /**
     * Initialize encryption key using Android Keystore
     */
    private void initializeEncryption() {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            logWarning(TAG, "Android Keystore is not fully supported on this API level. Encryption disabled.");
            encryptionEnabled = false;
            return;
        }

        try {
            KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
            keyStore.load(null);

            if (!keyStore.containsAlias(ENCRYPTION_KEY_ALIAS)) {
                KeyGenerator keyGen = KeyGenerator.getInstance(
                        KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");
                keyGen.init(new KeyGenParameterSpec.Builder(
                        ENCRYPTION_KEY_ALIAS,
                        KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
                        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                        .setKeySize(256)
                        .build());
                keyGen.generateKey();
            }

            encryptionKey = (SecretKey) keyStore.getKey(ENCRYPTION_KEY_ALIAS, null);

        } catch (Exception e) {
            logError(TAG, "Failed to initialize encryption with Keystore", e);
            encryptionEnabled = false;
        }
    }

    /**
     * Initialize logging system
     */
    private void initializeLogging() {
        if (fileLoggingEnabled) {
            createLogDirectory();
            rotateLogFileIfNeeded();
        }

        // Setup uncaught exception handler
        if (crashReportingEnabled) {
            Thread.setDefaultUncaughtExceptionHandler(new CustomUncaughtExceptionHandler());
        }
    }

    /**
     * Setup custom Timber tree
     */
    private void setupCustomTimberTree() {
        Timber.plant(new CustomTimberTree());
    }

    /**
     * Custom Timber Tree implementation
     */
    private class CustomTimberTree extends Timber.DebugTree {
        @Override
        protected void log(int priority, String tag, @NonNull String message, Throwable t) {
            super.log(priority, tag, message, t);

            LogLevel level = priorityToLogLevel(priority);
            LogEntry entry = new LogEntry(level.fullName, tag, message);

            // Add exception info if present
            if (t != null) {
                entry.metadata.put("exception", getStackTraceString(t));
            }

            // Add performance metrics
            addPerformanceMetrics(entry);

            // Store log entry
            storeLogEntry(entry);

            // Update counters
            updateLogCounters(level, tag);

            // Write to file if enabled
            if (fileLoggingEnabled) {
                writeToFile(entry);
            }
        }

        @Override
        protected String createStackElementTag(@NonNull StackTraceElement element) {
            if (stackTraceEnabled) {
                return String.format("(%s:%s)#%s",
                    element.getFileName(),
                    element.getLineNumber(),
                    element.getMethodName());
            } else {
                return super.createStackElementTag(element);
            }
        }
    }

    /**
     * Convert Android log priority to LogLevel
     */
    private LogLevel priorityToLogLevel(int priority) {
        switch (priority) {
            case Log.VERBOSE: return LogLevel.VERBOSE;
            case Log.DEBUG: return LogLevel.DEBUG;
            case Log.INFO: return LogLevel.INFO;
            case Log.WARN: return LogLevel.WARN;
            case Log.ERROR: return LogLevel.ERROR;
            case Log.ASSERT: return LogLevel.WTF;
            default: return LogLevel.DEBUG;
        }
    }

    /**
     * Add performance metrics to log entry
     */
    private void addPerformanceMetrics(LogEntry entry) {
        if (performanceMonitoringEnabled) {
            entry.metadata.put("memory_usage", getMemoryUsage());
            entry.metadata.put("battery_level", getBatteryLevel());
            entry.metadata.put("available_storage", getAvailableStorage());
            entry.metadata.put("network_status", getNetworkStatus());
        }
    }

    /**
     * Store log entry in memory
     */
    private void storeLogEntry(LogEntry entry) {
        synchronized (recentLogs) {
            recentLogs.add(entry);
            activeTags.add(entry.tag);

            // Keep only recent logs (last 1000)
            if (recentLogs.size() > 1000) {
                recentLogs.remove(0);
            }
        }
    }

    /**
     * Update log counters
     */
    private void updateLogCounters(LogLevel level, String tag) {
        String key = level.fullName + "_" + tag;
        logCounts.put(key, logCounts.getOrDefault(key, 0) + 1);
        logCounter.incrementAndGet();
    }

/**
     * Write log entry to file
     */
    private void writeToFile(LogEntry entry) {
        executorService.execute(() -> {
            try {
                ensureLogFileExists();

                String formattedLog = formatLogEntry(entry, LogFormat.DETAILED);

                synchronized (this) {
                    if (currentLogWriter != null) {
                        currentLogWriter.write(formattedLog);
                        currentLogWriter.newLine();
                        currentLogWriter.flush();
                        currentFileSize += formattedLog.getBytes(StandardCharsets.UTF_8).length + 1;

                        // Check if file size exceeds limit
                        if (currentFileSize >= maxFileSize) {
                            rotateLogFile();
                        }
                    }
                }
            } catch (IOException e) {
                logError(TAG, "Failed to write to log file", e);
            }
        });
    }

    /**
     * Format log entry based on specified format
     */
    private String formatLogEntry(LogEntry entry, LogFormat format) {
        switch (format) {
            case JSON:
                return gson.toJson(entry);
            case XML:
                return formatAsXml(entry);
            case CSV:
                return formatAsCsv(entry);
            case DETAILED:
                return formatDetailed(entry);
            case SIMPLE:
            default:
                return formatSimple(entry);
        }
    }

    /**
     * Format log entry as detailed string
     */
    private String formatDetailed(LogEntry entry) {
        StringBuilder sb = new StringBuilder();
        sb.append(dateFormat.format(new Date(entry.timestamp)));
        sb.append(" [").append(entry.level).append("]");

        if (threadInfoEnabled) {
            sb.append(" [").append(entry.threadName).append("]");
        }

        sb.append(" ").append(entry.tag).append(": ").append(entry.message);

        if (entry.className != null && entry.methodName != null) {
            sb.append(" (").append(getSimpleClassName(entry.className))
              .append(".").append(entry.methodName)
              .append(":").append(entry.lineNumber).append(")");
        }

        if (!entry.metadata.isEmpty()) {
            sb.append(" | Metadata: ").append(gson.toJson(entry.metadata));
        }

        return sb.toString();
    }

    /**
     * Format log entry as simple string
     */
    private String formatSimple(LogEntry entry) {
        return String.format("%s %s/%s: %s",
            dateFormat.format(new Date(entry.timestamp)),
            entry.level.charAt(0),
            entry.tag,
            entry.message);
    }

    /**
     * Format log entry as XML
     */
    private String formatAsXml(LogEntry entry) {
        StringBuilder xml = new StringBuilder();
        xml.append("<log>");
        xml.append("<timestamp>").append(entry.timestamp).append("</timestamp>");
        xml.append("<level>").append(entry.level).append("</level>");
        xml.append("<tag>").append(escapeXml(entry.tag)).append("</tag>");
        xml.append("<message>").append(escapeXml(entry.message)).append("</message>");
        xml.append("<thread>").append(escapeXml(entry.threadName)).append("</thread>");
        xml.append("<class>").append(escapeXml(entry.className)).append("</class>");
        xml.append("<method>").append(escapeXml(entry.methodName)).append("</method>");
        xml.append("<line>").append(entry.lineNumber).append("</line>");
        xml.append("</log>");
        return xml.toString();
    }

    /**
     * Format log entry as CSV
     */
    private String formatAsCsv(LogEntry entry) {
        return String.format("\"%d\",\"%s\",\"%s\",\"%s\",\"%s\",\"%s\",\"%s\",\"%d\"",
            entry.timestamp,
            entry.level,
            escapeCsv(entry.tag),
            escapeCsv(entry.message),
            escapeCsv(entry.threadName),
            escapeCsv(entry.className),
            escapeCsv(entry.methodName),
            entry.lineNumber);
    }

    /**
     * Escape XML special characters
     */
    private String escapeXml(String text) {
        if (text == null) return "";
        return text.replace("&", "&amp;")
                  .replace("<", "&lt;")
                  .replace(">", "&gt;")
                  .replace("\"", "&quot;")
                  .replace("'", "&apos;");
    }

    /**
     * Escape CSV special characters
     */
    private String escapeCsv(String text) {
        if (text == null) return "";
        return text.replace("\"", "\"\"");
    }

    /**
     * Get simple class name from full class name
     */
    private String getSimpleClassName(String fullClassName) {
        if (fullClassName == null) return "Unknown";
        int lastDot = fullClassName.lastIndexOf('.');
        return lastDot >= 0 ? fullClassName.substring(lastDot + 1) : fullClassName;
    }

    /**
     * Create log directory if it doesn't exist
     */
    private void createLogDirectory() {
        File logDir = new File(applicationContext.getExternalFilesDir(null), LOG_DIR);
        if (!logDir.exists()) {
            boolean created = logDir.mkdirs();
            if (!created) {
                logError(TAG, "Failed to create log directory: " + logDir.getAbsolutePath());
            }
        }
    }

    /**
     * Ensure log file exists and is writable
     */
    private void ensureLogFileExists() throws IOException {
        synchronized (this) {
            if (currentLogFile == null || !currentLogFile.exists()) {
                rotateLogFile();
            }
        }
    }

    /**
     * Rotate log file if needed
     */
    private void rotateLogFileIfNeeded() {
        synchronized (this) {
            if (currentLogFile != null && currentLogFile.exists() && currentLogFile.length() >= maxFileSize) {
                rotateLogFile();
            }
        }
    }

    /**
     * Rotate to a new log file
     */
    private void rotateLogFile() {
        executorService.execute(() -> {
            synchronized (this) {
                try {
                    // Close current writer
                    if (currentLogWriter != null) {
                        currentLogWriter.close();
                        currentLogWriter = null;
                    }

                    // Process old file if needed
                    if (currentLogFile != null && currentLogFile.exists()) {
                        processOldLogFile(currentLogFile);
                    }

                    // Create new log file
                    File logDir = new File(applicationContext.getExternalFilesDir(null), LOG_DIR);
                    String fileName = LOG_FILE_PREFIX + fileDateFormat.format(new Date()) + LOG_FILE_EXTENSION;
                    currentLogFile = new File(logDir, fileName);
                    currentLogWriter = new BufferedWriter(new FileWriter(currentLogFile, true));
                    currentFileSize = currentLogFile.length();

                    // Cleanup old files if needed
                    if (autoCleanupEnabled) {
                        cleanupOldLogFiles();
                    }

                } catch (IOException e) {
                    logError(TAG, "Failed to rotate log file", e);
                }
            }
        });
    }

    /**
     * Process old log file (compress/encrypt if enabled)
     */
    private void processOldLogFile(File oldFile) {
        try {
            File processedFile = oldFile;

            // Compress if enabled
            if (compressionEnabled) {
                File compressedFile = compressFile(processedFile);
                if (processedFile != compressedFile && processedFile.exists()) {
                    processedFile.delete();
                }
                processedFile = compressedFile;
            }

            // Encrypt if enabled
            if (encryptionEnabled) {
                File encryptedFile = encryptFile(processedFile);
                if (processedFile != encryptedFile && processedFile.exists()) {
                    processedFile.delete();
                }
            }

        } catch (Exception e) {
            logError(TAG, "Failed to process old log file", e);
        }
    }

    /**
     * Compress log file using GZIP
     */
    private File compressFile(File inputFile) throws IOException {
        File compressedFile = new File(inputFile.getAbsolutePath() + COMPRESSED_EXTENSION);

        try (FileInputStream fis = new FileInputStream(inputFile);
             FileOutputStream fos = new FileOutputStream(compressedFile);
             GZIPOutputStream gzos = new GZIPOutputStream(fos)) {

            byte[] buffer = new byte[1024];
            int length;
            while ((length = fis.read(buffer)) > 0) {
                gzos.write(buffer, 0, length);
            }
        }

        logInfo(TAG, "Compressed log file: " + compressedFile.getName());
        return compressedFile;
    }

    /**
     * Encrypt log file using AES/GCM/NoPadding with a unique IV
     */
    private File encryptFile(File inputFile) throws Exception {
        if (encryptionKey == null) {
            logError(TAG, "Encryption key not available. Cannot encrypt file.");
            return inputFile;
        }

        File encryptedFile = new File(inputFile.getAbsolutePath() + ENCRYPTED_EXTENSION);
        
        // Generate a unique IV for each encryption operation
        byte[] iv = new byte[GCM_IV_LENGTH];
        new Random().nextBytes(iv);
        
        Cipher cipher = Cipher.getInstance(ENCRYPTION_TRANSFORMATION_GCM);
        cipher.init(Cipher.ENCRYPT_MODE, encryptionKey, new GCMParameterSpec(128, iv));

        try (FileOutputStream fos = new FileOutputStream(encryptedFile);
             FileInputStream fis = new FileInputStream(inputFile);
             CipherOutputStream cos = new CipherOutputStream(fos, cipher)) {
            
            // Write the IV to the beginning of the encrypted file
            fos.write(iv);

            byte[] buffer = new byte[1024];
            int length;
            while ((length = fis.read(buffer)) > 0) {
                cos.write(buffer, 0, length);
            }
        }

        logInfo(TAG, "Encrypted log file: " + encryptedFile.getName());
        return encryptedFile;
    }

    /**
     * Cleanup old log files
     */
    private void cleanupOldLogFiles() {
        File logDir = new File(applicationContext.getExternalFilesDir(null), LOG_DIR);
        File[] logFiles = logDir.listFiles((dir, name) -> name.startsWith(LOG_FILE_PREFIX));

        if (logFiles != null && logFiles.length > maxLogFiles) {
            // Sort by last modified date
            Arrays.sort(logFiles, (f1, f2) -> Long.compare(f1.lastModified(), f2.lastModified()));

            // Delete oldest files
            int filesToDelete = logFiles.length - maxLogFiles;
            for (int i = 0; i < filesToDelete; i++) {
                if (logFiles[i].delete()) {
                    logInfo(TAG, "Deleted old log file: " + logFiles[i].getName());
                }
            }
        }

        lastCleanupTime = System.currentTimeMillis();
    }

    /**
     * Start performance monitoring
     */
    private void startPerformanceMonitoring() {
        executorService.execute(() -> {
            while (performanceMonitoringEnabled) {
                try {
                    updatePerformanceMetrics();
                    Thread.sleep(30000); // Update every 30 seconds
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                } catch (Exception e) {
                    logError(TAG, "Performance monitoring error", e);
                }
            }
        });
    }

    /**
     * Update performance metrics
     */
    private void updatePerformanceMetrics() {
        long timestamp = System.currentTimeMillis();
        performanceMetrics.put("memory_usage_" + timestamp, getMemoryUsage());
        performanceMetrics.put("battery_level_" + timestamp, (long) getBatteryLevel());
        performanceMetrics.put("storage_available_" + timestamp, getAvailableStorage());
        performanceMetrics.put("network_status_" + timestamp, (long) getNetworkStatusValue());

        // Keep only recent metrics (last hour)
        long oneHourAgo = timestamp - 3600000;
        performanceMetrics.entrySet().removeIf(entry -> {
            String key = entry.getKey();
            String[] parts = key.split("_");
            if (parts.length >= 3) {
                try {
                    long metricTime = Long.parseLong(parts[parts.length - 1]);
                    return metricTime < oneHourAgo;
                } catch (NumberFormatException e) {
                    return false;
                }
            }
            return false;
        });
    }

    /**
     * Get memory usage in MB
     */
    private long getMemoryUsage() {
        ActivityManager activityManager = (ActivityManager) applicationContext.getSystemService(Context.ACTIVITY_SERVICE);
        ActivityManager.MemoryInfo memoryInfo = new ActivityManager.MemoryInfo();
        activityManager.getMemoryInfo(memoryInfo);

        long usedMemory = memoryInfo.totalMem - memoryInfo.availMem;
        return usedMemory / (1024 * 1024); // Convert to MB
    }

    /**
     * Get battery level percentage
     */
    private int getBatteryLevel() {
        try {
            BatteryManager batteryManager = (BatteryManager) applicationContext.getSystemService(Context.BATTERY_SERVICE);
            return batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY);
        } catch (Exception e) {
            return -1;
        }
    }

    /**
     * Get available storage space in GB
     */
    private long getAvailableStorage() {
        try {
            File logDir = applicationContext.getExternalFilesDir(null);
            if (logDir == null) {
                return -1;
            }
            StatFs stat = new StatFs(logDir.getPath());
            long availableBytes = stat.getAvailableBytes();
            return availableBytes / (1024 * 1024 * 1024); // Convert to GB
        } catch (Exception e) {
            return -1;
        }
    }

    /**
     * Get network status
     */
    private String getNetworkStatus() {
        try {
            android.net.ConnectivityManager cm = (android.net.ConnectivityManager)
                applicationContext.getSystemService(Context.CONNECTIVITY_SERVICE);
            android.net.NetworkInfo activeNetwork = cm.getActiveNetworkInfo();

            if (activeNetwork != null && activeNetwork.isConnected()) {
                switch (activeNetwork.getType()) {
                    case android.net.ConnectivityManager.TYPE_WIFI:
                        return "WiFi";
                    case android.net.ConnectivityManager.TYPE_MOBILE:
                        return "Mobile";
                    default:
                        return "Connected";
                }
            }
            return "Disconnected";
        } catch (Exception e) {
            return "Unknown";
        }
    }

    /**
     * Get network status value for performance metrics
     * (0 = Disconnected, 1 = Connected)
     */
    private int getNetworkStatusValue() {
        try {
            android.net.ConnectivityManager cm = (android.net.ConnectivityManager)
                applicationContext.getSystemService(Context.CONNECTIVITY_SERVICE);
            android.net.NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
            return (activeNetwork != null && activeNetwork.isConnected()) ? 1 : 0;
        } catch (Exception e) {
            return -1;
        }
    }

    /**
     * Schedule log cleanup
     */
    private void scheduleLogCleanup() {
        executorService.execute(() -> {
            while (autoCleanupEnabled) {
                try {
                    long currentTime = System.currentTimeMillis();
                    // Cleanup every 24 hours
                    if (currentTime - lastCleanupTime > 86400000) {
                        cleanupOldLogFiles();
                    }
                    Thread.sleep(3600000); // Check every hour
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                } catch (Exception e) {
                    logError(TAG, "Scheduled cleanup error", e);
                }
            }
        });
    }

    /**
     * Log device information
     */
    private void logDeviceInfo() {
        logInfo(TAG, "Device Info - Model: " + Build.MODEL +
                     ", Android: " + Build.VERSION.RELEASE +
                     ", API: " + Build.VERSION.SDK_INT +
                     ", Manufacturer: " + Build.MANUFACTURER);

        logInfo(TAG, "App Info - Package: " + applicationContext.getPackageName() +
                     ", Process ID: " + android.os.Process.myPid() +
                     ", Thread ID: " + android.os.Process.myTid());

        logInfo(TAG, "System Info - Memory: " + getMemoryUsage() + "MB" +
                     ", Storage: " + getAvailableStorage() + "GB" +
                     ", Battery: " + getBatteryLevel() + "%" +
                     ", Network: " + getNetworkStatus());
    }

    /**
     * Get stack trace as string
     */
    private String getStackTraceString(Throwable t) {
        StringWriter sw = new StringWriter();
        PrintWriter pw = new PrintWriter(sw);
        t.printStackTrace(pw);
        return sw.toString();
    }

    /**
     * Get feature count
     */
    private int getFeatureCount() {
        return 24; // CPU monitoring removed
    }

   /**
     * Custom uncaught exception handler
     */
    private class CustomUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
        private final Thread.UncaughtExceptionHandler defaultHandler;

        public CustomUncaughtExceptionHandler() {
            this.defaultHandler = Thread.getDefaultUncaughtExceptionHandler();
        }

        @Override
        public void uncaughtException(@NonNull Thread t, @NonNull Throwable e) {
            try {
                logError(TAG, "FATAL CRASH in thread " + t.getName(), e);

                // Create crash report
                LogEntry crashEntry = new LogEntry("CRASH", "FATAL", "Application crashed: " + e.getMessage());
                crashEntry.metadata.put("thread", t.getName());
                crashEntry.metadata.put("exception_type", e.getClass().getSimpleName());
                crashEntry.metadata.put("stack_trace", getStackTraceString(e));
                crashEntry.metadata.put("device_info", getDeviceInfoMap());
                crashEntry.metadata.put("performance_state", getCurrentPerformanceState());

                // Force write crash log
                String crashLog = formatLogEntry(crashEntry, LogFormat.JSON);
                File crashFile = new File(applicationContext.getExternalFilesDir(null),
                    LOG_DIR + "/crash_" + fileDateFormat.format(new Date()) + ".log");

                try (FileWriter writer = new FileWriter(crashFile)) {
                    writer.write(crashLog);
                    writer.flush();
                }

            } catch (Exception ex) {
                // Ignore errors in crash handler
            } finally {
                if (defaultHandler != null) {
                    defaultHandler.uncaughtException(t, e);
                }
            }
        }
    }

    /**
     * Get device info as map
     */
    private Map<String, Object> getDeviceInfoMap() {
        Map<String, Object> info = new HashMap<>();
        info.put("model", Build.MODEL);
        info.put("manufacturer", Build.MANUFACTURER);
        info.put("android_version", Build.VERSION.RELEASE);
        info.put("api_level", Build.VERSION.SDK_INT);
        info.put("app_version", getAppVersion());
        info.put("available_processors", Runtime.getRuntime().availableProcessors());
        info.put("max_memory", Runtime.getRuntime().maxMemory() / (1024 * 1024));
        return info;
    }

    /**
     * Get current performance state
     */
    private Map<String, Object> getCurrentPerformanceState() {
        Map<String, Object> state = new HashMap<>();
        state.put("memory_usage_mb", getMemoryUsage());
        state.put("battery_level", getBatteryLevel());
        state.put("available_storage_gb", getAvailableStorage());
        state.put("network_status", getNetworkStatus());
        state.put("log_count", logCounter.get());
        return state;
    }

    /**
     * Get app version
     */
    private String getAppVersion() {
        try {
            return applicationContext.getPackageManager()
                .getPackageInfo(applicationContext.getPackageName(), 0).versionName;
        } catch (Exception e) {
            return "Unknown";
        }
    }

    // ==================== PUBLIC API METHODS ====================

    /**
     * Log verbose message
     */
    public static void logVerbose(String tag, String message) {
        Timber.tag(tag).v(message);
    }

    /**
     * Log debug message
     */
    public static void logDebug(String tag, String message) {
        Timber.tag(tag).d(message);
    }

    /**
     * Log info message
     */
    public static void logInfo(String tag, String message) {
        Timber.tag(tag).i(message);
    }

    /**
     * Log warning message
     */
    public static void logWarning(String tag, String message) {
        Timber.tag(tag).w(message);
    }

    /**
     * Log error message
     */
    public static void logError(String tag, String message) {
        Timber.tag(tag).e(message);
    }

    /**
     * Log error message with exception
     */
    public static void logError(String tag, String message, Throwable throwable) {
        Timber.tag(tag).e(throwable, message);
    }

    /**
     * Log WTF message
     */
    public static void logWtf(String tag, String message) {
        Timber.tag(tag).wtf(message);
    }

    /**
     * Log method entry for performance tracking
     */
    public static void logMethodEntry(String className, String methodName) {
        long timestamp = System.currentTimeMillis();
        String key = className + "." + methodName + "_entry";
        getInstance().performanceMetrics.put(key, timestamp);
        logDebug("PERF", "Method Entry: " + className + "." + methodName);
    }

    /**
     * Log method exit for performance tracking
     */
    public static void logMethodExit(String className, String methodName) {
        long timestamp = System.currentTimeMillis();
        String entryKey = className + "." + methodName + "_entry";
        Long entryTime = getInstance().performanceMetrics.get(entryKey);

        if (entryTime != null) {
            long duration = timestamp - entryTime;
            logDebug("PERF", "Method Exit: " + className + "." + methodName + " (Duration: " + duration + "ms)");
            getInstance().performanceMetrics.remove(entryKey);
        } else {
            logDebug("PERF", "Method Exit: " + className + "." + methodName);
        }
    }

    /**
     * Log network request
     */
    public static void logNetworkRequest(String method, String url, int responseCode, long duration) {
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("method", method);
        metadata.put("url", url);
        metadata.put("response_code", responseCode);
        metadata.put("duration_ms", duration);

        LogEntry entry = new LogEntry("NETWORK", "NET", method + " " + url + " -> " + responseCode);
        entry.metadata.putAll(metadata);
        getInstance().storeLogEntry(entry);

        logInfo("NETWORK", method + " " + url + " -> " + responseCode + " (" + duration + "ms)");
    }

    /**
     * Log database operation
     */
    public static void logDatabaseOperation(String operation, String table, long duration) {
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("operation", operation);
        metadata.put("table", table);
        metadata.put("duration_ms", duration);

        LogEntry entry = new LogEntry("DATABASE", "DB", operation + " on " + table);
        entry.metadata.putAll(metadata);
        getInstance().storeLogEntry(entry);

        logInfo("DATABASE", operation + " on " + table + " (" + duration + "ms)");
    }

    /**
     * Log user action
     */
    public static void logUserAction(String action, String screen, Map<String, Object> properties) {
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("action", action);
        metadata.put("screen", screen);
        metadata.put("timestamp", System.currentTimeMillis());
        if (properties != null) {
            metadata.putAll(properties);
        }

        LogEntry entry = new LogEntry("USER_ACTION", "UI", action + " on " + screen);
        entry.metadata.putAll(metadata);
        getInstance().storeLogEntry(entry);

        logInfo("USER_ACTION", action + " on " + screen);
    }

    /**
     * Get all log entries
     */
    public List<LogEntry> getAllLogEntries() {
        synchronized (recentLogs) {
            return new ArrayList<>(recentLogs);
        }
    }

    /**
     * Filter log entries by level
     */
    public List<LogEntry> filterLogsByLevel(LogLevel level) {
        synchronized (recentLogs) {
            List<LogEntry> filtered = new ArrayList<>();
            for (LogEntry entry : recentLogs) {
                if (entry.level.equals(level.fullName)) {
                    filtered.add(entry);
                }
            }
            return filtered;
        }
    }

    /**
     * Filter log entries by tag
     */
    public List<LogEntry> filterLogsByTag(String tag) {
        synchronized (recentLogs) {
            List<LogEntry> filtered = new ArrayList<>();
            for (LogEntry entry : recentLogs) {
                if (entry.tag.equals(tag)) {
                    filtered.add(entry);
                }
            }
            return filtered;
        }
    }

    /**
     * Filter log entries by time range
     */
    public List<LogEntry> filterLogsByTimeRange(long startTime, long endTime) {
        synchronized (recentLogs) {
            List<LogEntry> filtered = new ArrayList<>();
            for (LogEntry entry : recentLogs) {
                if (entry.timestamp >= startTime && entry.timestamp <= endTime) {
                    filtered.add(entry);
                }
            }
            return filtered;
        }
    }

    /**
     * Search log entries by message content
     */
    public List<LogEntry> searchLogsByMessage(String query) {
        synchronized (recentLogs) {
            List<LogEntry> filtered = new ArrayList<>();
            String lowerQuery = query.toLowerCase();
            for (LogEntry entry : recentLogs) {
                if (entry.message.toLowerCase().contains(lowerQuery)) {
                    filtered.add(entry);
                }
            }
            return filtered;
        }
    }

    /**
     * Search log entries by multiple criteria
     */
    public List<LogEntry> searchLogs(String query, LogLevel level, String tag, long startTime, long endTime) {
        synchronized (recentLogs) {
            List<LogEntry> filtered = new ArrayList<>();
            String lowerQuery = query != null ? query.toLowerCase() : null;

            for (LogEntry entry : recentLogs) {
                boolean matches = true;

                if (lowerQuery != null && !entry.message.toLowerCase().contains(lowerQuery)) {
                    matches = false;
                }
                if (level != null && !entry.level.equals(level.fullName)) {
                    matches = false;
                }
                if (tag != null && !entry.tag.equals(tag)) {
                    matches = false;
                }
                if (startTime > 0 && entry.timestamp < startTime) {
                    matches = false;
                }
                if (endTime > 0 && entry.timestamp > endTime) {
                    matches = false;
                }

                if (matches) {
                    filtered.add(entry);
                }
            }
            return filtered;
        }
    }

    /**
     * Get log statistics
     */
    public Map<String, Object> getLogStatistics() {
        Map<String, Object> stats = new HashMap<>();
        stats.put("total_logs", logCounter.get());
        stats.put("active_tags", activeTags.size());
        stats.put("recent_logs_count", recentLogs.size());
        stats.put("log_counts_by_level", getLogCountsByLevel());
        stats.put("performance_metrics_count", performanceMetrics.size());
        stats.put("file_logging_enabled", fileLoggingEnabled);
        stats.put("current_file_size", currentFileSize);
        stats.put("last_cleanup_time", lastCleanupTime);
        stats.put("memory_usage_mb", getMemoryUsage());
        stats.put("battery_level", getBatteryLevel());
        stats.put("available_storage_gb", getAvailableStorage());
        stats.put("network_status", getNetworkStatus());
        return stats;
    }

    /**
     * Get log counts by level
     */
    private Map<String, Integer> getLogCountsByLevel() {
        Map<String, Integer> levelCounts = new HashMap<>();
        for (LogLevel level : LogLevel.values()) {
            int count = 0;
            for (Map.Entry<String, Integer> entry : logCounts.entrySet()) {
                if (entry.getKey().startsWith(level.fullName + "_")) {
                    count += entry.getValue();
                }
            }
            levelCounts.put(level.fullName, count);
        }
        return levelCounts;
    }

    /**
     * Export logs to file
     */
    public File exportLogs(LogFormat format) throws IOException {
        File exportDir = new File(applicationContext.getExternalFilesDir(null), "exports");
        if (!exportDir.exists()) {
            exportDir.mkdirs();
        }

        String fileName = "log_export_" + fileDateFormat.format(new Date());
        switch (format) {
            case JSON:
                fileName += ".json";
                break;
            case XML:
                fileName += ".xml";
                break;
            case CSV:
                fileName += ".csv";
                break;
            default:
                fileName += ".txt";
                break;
        }

        File exportFile = new File(exportDir, fileName);

        try (FileWriter writer = new FileWriter(exportFile)) {
            if (format == LogFormat.CSV) {
                writer.write("Timestamp,Level,Tag,Message,Thread,Class,Method,Line\n");
            } else if (format == LogFormat.XML) {
                writer.write("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<logs>\n");
            } else if (format == LogFormat.JSON) {
                writer.write("{\n  \"logs\": [\n");
            }

            synchronized (recentLogs) {
                for (int i = 0; i < recentLogs.size(); i++) {
                    LogEntry entry = recentLogs.get(i);
                    String formattedEntry = formatLogEntry(entry, format);

                    if (format == LogFormat.JSON) {
                        writer.write("    " + formattedEntry);
                        if (i < recentLogs.size() - 1) {
                            writer.write(",");
                        }
                        writer.write("\n");
                    } else {
                        writer.write(formattedEntry + "\n");
                    }
                }
            }

            if (format == LogFormat.XML) {
                writer.write("</logs>\n");
            } else if (format == LogFormat.JSON) {
                writer.write("  ]\n}\n");
            }
        }

        logInfo(TAG, "Logs exported to: " + exportFile.getAbsolutePath());
        return exportFile;
    }

    /**
     * Clear all logs
     */
    public void clearAllLogs() {
        synchronized (recentLogs) {
            recentLogs.clear();
        }
        logCounts.clear();
        performanceMetrics.clear();
        activeTags.clear();
        logCounter.set(0);

        executorService.execute(() -> {
            try {
                synchronized (this) {
                    if (currentLogWriter != null) {
                        currentLogWriter.close();
                        currentLogWriter = null;
                    }
                }

                File logDir = new File(applicationContext.getExternalFilesDir(null), LOG_DIR);
                File[] logFiles = logDir.listFiles();
                if (logFiles != null) {
                    for (File file : logFiles) {
                        file.delete();
                    }
                }

                rotateLogFile();
                logInfo(TAG, "All logs cleared successfully");
            } catch (Exception e) {
                logError(TAG, "Failed to clear log files", e);
            }
        });
    }

    /**
     * Get active tags
     */
    public Set<String> getActiveTags() {
        return new HashSet<>(activeTags);
    }

    /**
     * Get performance metrics for visualization
     */
    public Map<String, List<Map<String, Object>>> getPerformanceHistory() {
        Map<String, List<Map<String, Object>>> history = new HashMap<>();

        List<Map<String, Object>> memoryHistory = new ArrayList<>();
        List<Map<String, Object>> batteryHistory = new ArrayList<>();
        List<Map<String, Object>> storageHistory = new ArrayList<>();
        List<Map<String, Object>> networkHistory = new ArrayList<>();

        for (Map.Entry<String, Long> entry : performanceMetrics.entrySet()) {
            String key = entry.getKey();
            Long value = entry.getValue();

            if (key.startsWith("memory_usage_")) {
                String timestamp = key.substring("memory_usage_".length());
                Map<String, Object> point = new HashMap<>();
                point.put("timestamp", Long.parseLong(timestamp));
                point.put("value", value);
                memoryHistory.add(point);
            } else if (key.startsWith("battery_level_")) {
                String timestamp = key.substring("battery_level_".length());
                Map<String, Object> point = new HashMap<>();
                point.put("timestamp", Long.parseLong(timestamp));
                point.put("value", value);
                batteryHistory.add(point);
            } else if (key.startsWith("storage_available_")) {
                String timestamp = key.substring("storage_available_".length());
                Map<String, Object> point = new HashMap<>();
                point.put("timestamp", Long.parseLong(timestamp));
                point.put("value", value);
                storageHistory.add(point);
            } else if (key.startsWith("network_status_")) {
                String timestamp = key.substring("network_status_".length());
                Map<String, Object> point = new HashMap<>();
                point.put("timestamp", Long.parseLong(timestamp));
                point.put("value", value);
                networkHistory.add(point);
            }
        }

        history.put("memory", memoryHistory);
        history.put("battery", batteryHistory);
        history.put("storage", storageHistory);
        history.put("network", networkHistory);

        return history;
    }

    // ==================== CONFIGURATION METHODS ====================

    public void setFileLoggingEnabled(boolean enabled) {
        this.fileLoggingEnabled = enabled;
        savePreferences();
        logInfo(TAG, "File logging " + (enabled ? "enabled" : "disabled"));
    }

    public void setCompressionEnabled(boolean enabled) {
        this.compressionEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Log compression " + (enabled ? "enabled" : "disabled"));
    }

    public void setEncryptionEnabled(boolean enabled) {
        this.encryptionEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Log encryption " + (enabled ? "enabled" : "disabled"));
    }

    public void setCrashReportingEnabled(boolean enabled) {
        this.crashReportingEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Crash reporting " + (enabled ? "enabled" : "disabled"));
    }

    public void setPerformanceMonitoringEnabled(boolean enabled) {
        this.performanceMonitoringEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Performance monitoring " + (enabled ? "enabled" : "disabled"));

        if (enabled) {
            startPerformanceMonitoring();
        }
    }

    public void setAutoCleanupEnabled(boolean enabled) {
        this.autoCleanupEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Auto cleanup " + (enabled ? "enabled" : "disabled"));

        if (enabled) {
            scheduleLogCleanup();
        }
    }

    public void setColoredLogsEnabled(boolean enabled) {
        this.coloredLogsEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Colored logs " + (enabled ? "enabled" : "disabled"));
    }

    public void setThreadInfoEnabled(boolean enabled) {
        this.threadInfoEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Thread info " + (enabled ? "enabled" : "disabled"));
    }

    public void setStackTraceEnabled(boolean enabled) {
        this.stackTraceEnabled = enabled;
        savePreferences();
        logInfo(TAG, "Stack trace " + (enabled ? "enabled" : "disabled"));
    }

    public void setMaxLogFiles(int maxFiles) {
        this.maxLogFiles = Math.max(1, Math.min(maxFiles, 100));
        savePreferences();
        logInfo(TAG, "Max log files set to: " + this.maxLogFiles);
    }

    public void setMaxFileSize(long maxSize) {
        this.maxFileSize = Math.max(1024 * 1024, maxSize);
        savePreferences();
        logInfo(TAG, "Max file size set to: " + (this.maxFileSize / (1024 * 1024)) + "MB");
    }

    // Getters for configuration
    public boolean isFileLoggingEnabled() { return fileLoggingEnabled; }
    public boolean isCompressionEnabled() { return compressionEnabled; }
    public boolean isEncryptionEnabled() { return encryptionEnabled; }
    public boolean isCrashReportingEnabled() { return crashReportingEnabled; }
    public boolean isPerformanceMonitoringEnabled() { return performanceMonitoringEnabled; }
    public boolean isAutoCleanupEnabled() { return autoCleanupEnabled; }
    public boolean isColoredLogsEnabled() { return coloredLogsEnabled; }
    public boolean isThreadInfoEnabled() { return threadInfoEnabled; }
    public boolean isStackTraceEnabled() { return stackTraceEnabled; }
    public int getMaxLogFiles() { return maxLogFiles; }
    public long getMaxFileSize() { return maxFileSize; }

    /**
     * Get current log file path
     */
    public String getCurrentLogFilePath() {
        return currentLogFile != null ? currentLogFile.getAbsolutePath() : "No active log file";
    }

    /**
     * Get all log files
     */
    public List<File> getAllLogFiles() {
        List<File> logFiles = new ArrayList<>();
        File logDir = new File(applicationContext.getExternalFilesDir(null), LOG_DIR);
        File[] files = logDir.listFiles((dir, name) -> name.startsWith(LOG_FILE_PREFIX));

        if (files != null) {
            Arrays.sort(files, (f1, f2) -> Long.compare(f2.lastModified(), f1.lastModified()));
            logFiles.addAll(Arrays.asList(files));
        }

        return logFiles;
    }

    /**
     * Shutdown LogUtils
     */
    public void shutdown() {
        try {
            if (currentLogWriter != null) {
                currentLogWriter.close();
                currentLogWriter = null;
            }

            executorService.shutdown();
            logInfo(TAG, "LogUtils shutdown completed");
        } catch (Exception e) {
            logError(TAG, "Error during shutdown", e);
        }
    }

    /**
     * Generate log report
     */
    public String generateLogReport() {
        StringBuilder report = new StringBuilder();
        Map<String, Object> stats = getLogStatistics();

        report.append("=== AXIOM LOG SYSTEM REPORT ===\n");
        report.append("Generated: ").append(dateFormat.format(new Date())).append("\n\n");

        report.append("GENERAL STATISTICS:\n");
        report.append("- Total logs: ").append(stats.get("total_logs")).append("\n");
        report.append("- Active tags: ").append(stats.get("active_tags")).append("\n");
        report.append("- Recent logs in memory: ").append(stats.get("recent_logs_count")).append("\n");
        report.append("- Current file size: ").append(stats.get("current_file_size")).append(" bytes\n\n");

        report.append("SYSTEM STATUS:\n");
        report.append("- Memory usage: ").append(stats.get("memory_usage_mb")).append(" MB\n");
        report.append("- Battery level: ").append(stats.get("battery_level")).append("%\n");
        report.append("- Available storage: ").append(stats.get("available_storage_gb")).append(" GB\n");
        report.append("- Network status: ").append(stats.get("network_status")).append("\n\n");

        report.append("CONFIGURATION:\n");
        report.append("- File logging: ").append(fileLoggingEnabled ? "Enabled" : "Disabled").append("\n");
        report.append("- Compression: ").append(compressionEnabled ? "Enabled" : "Disabled").append("\n");
        report.append("- Encryption: ").append(encryptionEnabled ? "Enabled" : "Disabled").append("\n");
        report.append("- Crash reporting: ").append(crashReportingEnabled ? "Enabled" : "Disabled").append("\n");
        report.append("- Performance monitoring: ").append(performanceMonitoringEnabled ? "Enabled" : "Disabled").append("\n");
        report.append("- Auto cleanup: ").append(autoCleanupEnabled ? "Enabled" : "Disabled").append("\n");

        return report.toString();
    }
}


// FILE: Axiom/app/src/main/jni/Android.mk
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := tomaslib
LOCAL_SRC_FILES := tomaslib.cpp

include $(BUILD_SHARED_LIBRARY)

// FILE: Axiom/app/src/main/jni/Application.mk
APP_ABI := all
APP_PLATFORM := android-21
APP_STL := c++_shared
APP_CPPFLAGS += -std=c++17

// FILE: Axiom/app/src/main/jni/tomaslib.cpp
#include <jni.h>
#include <string>
#include "tomaslib.h"

extern "C" JNIEXPORT jstring JNICALL
Java_com_axiomloader_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from TomasLib C++";
    return env->NewStringUTF(hello.c_str());
}

extern "C" JNIEXPORT jint JNICALL
Java_com_axiomloader_MainActivity_addNumbers(
        JNIEnv* env,
        jobject /* this */,
        jint a,
        jint b) {
    return a + b;
}

extern "C" JNIEXPORT void JNICALL
Java_com_axiomloader_MainActivity_initTomasLib(
        JNIEnv* env,
        jobject /* this */) {
    // Initialize TomasLib native library
    // Add your initialization code here
}

// FILE: Axiom/app/src/main/jni/tomaslib.h
#ifndef TOMASLIB_H
#define TOMASLIB_H

#include <jni.h>

#ifdef __cplusplus
extern "C" {
#endif

/**
 * Returns a greeting string from TomasLib C++
 */
JNIEXPORT jstring JNICALL
Java_com_axiomloader_MainActivity_stringFromJNI(JNIEnv* env, jobject thiz);

/**
 * Adds two integers and returns the result
 */
JNIEXPORT jint JNICALL
Java_com_axiomloader_MainActivity_addNumbers(JNIEnv* env, jobject thiz, jint a, jint b);

/**
 * Says hello to a person with their name
 */
JNIEXPORT jstring JNICALL
Java_com_axiomloader_MainActivity_sayHello(JNIEnv* env, jobject thiz, jstring name);

/**
 * Initialize the TomasLib native library
 */
JNIEXPORT void JNICALL
Java_com_axiomloader_MainActivity_initTomasLib(JNIEnv* env, jobject thiz);

#ifdef __cplusplus
}
#endif

#endif // TOMASLIB_H

// FILE: Axiom/app/src/main/res/drawable/ic_launcher_background.xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#3DDC84"
        android:pathData="M0,0h108v108h-108z" />
    <path
        android:fillColor="#00000000"
        android:pathData="M9,0L9,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,0L19,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M29,0L29,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M39,0L39,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M49,0L49,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M59,0L59,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M69,0L69,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M79,0L79,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M89,0L89,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M99,0L99,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,9L108,9"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,19L108,19"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,29L108,29"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,39L108,39"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,49L108,49"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,59L108,59"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,69L108,69"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,79L108,79"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,89L108,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,99L108,99"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,29L89,29"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,39L89,39"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,49L89,49"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,59L89,59"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,69L89,69"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,79L89,79"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M29,19L29,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M39,19L39,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M49,19L49,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M59,19L59,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M69,19L69,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M79,19L79,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
</vector>


// FILE: Axiom/app/src/main/res/drawable-v24/ic_launcher_foreground.xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path android:pathData="M31,63.928c0,0 6.4,-11 12.1,-13.1c7.2,-2.6 26,-1.4 26,-1.4l38.1,38.1L107,108.928l-32,-1L31,63.928z">
        <aapt:attr name="android:fillColor">
            <gradient
                android:endX="85.84757"
                android:endY="92.4963"
                android:startX="42.9492"
                android:startY="49.59793"
                android:type="linear">
                <item
                    android:color="#44000000"
                    android:offset="0.0" />
                <item
                    android:color="#00000000"
                    android:offset="1.0" />
            </gradient>
        </aapt:attr>
    </path>
    <path
        android:fillColor="#FFFFFF"
        android:fillType="nonZero"
        android:pathData="M65.3,45.828l3.8,-6.6c0.2,-0.4 0.1,-0.9 -0.3,-1.1c-0.4,-0.2 -0.9,-0.1 -1.1,0.3l-3.9,6.7c-6.3,-2.8 -13.4,-2.8 -19.7,0l-3.9,-6.7c-0.2,-0.4 -0.7,-0.5 -1.1,-0.3C38.8,38.328 38.7,38.828 38.9,39.228l3.8,6.6C36.2,49.428 31.7,56.028 31,63.928h46C76.3,56.028 71.8,49.428 65.3,45.828zM43.4,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2c-0.3,-0.7 -0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C45.3,56.528 44.5,57.328 43.4,57.328L43.4,57.328zM64.6,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2s-0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C66.5,56.528 65.6,57.328 64.6,57.328L64.6,57.328z"
        android:strokeWidth="1"
        android:strokeColor="#00000000" />
</vector>

// FILE: Axiom/app/src/main/res/layout/activity_logging.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".ui.LoggingActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="Advanced Logging System" />

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <!-- System Status Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="System Status"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginBottom="12dp" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:id="@+id/tv_memory_usage"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Memory: --"
                                android:textSize="12sp" />

                            <TextView
                                android:id="@+id/tv_cpu_usage"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="CPU: --"
                                android:textSize="12sp" />

                            <TextView
                                android:id="@+id/tv_battery_level"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Battery: --"
                                android:textSize="12sp" />

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:id="@+id/tv_network_status"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Network: --"
                                android:textSize="12sp" />

                            <TextView
                                android:id="@+id/tv_storage_available"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Storage: --"
                                android:textSize="12sp" />

                            <TextView
                                android:id="@+id/tv_active_tags"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Tags: --"
                                android:textSize="12sp" />

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Custom Log Input Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Send Custom Log"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginBottom="12dp" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:layout_marginBottom="8dp">

                        <Spinner
                            android:id="@+id/spinner_custom_log_level"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginEnd="8dp" />

                        <com.google.android.material.textfield.TextInputLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:hint="Tag">

                            <com.google.android.material.textfield.TextInputEditText
                                android:id="@+id/et_log_tag"
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:inputType="text"
                                android:maxLines="1" />

                        </com.google.android.material.textfield.TextInputLayout>

                    </LinearLayout>

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:hint="@string/log_message_hint"
                        android:layout_marginBottom="12dp">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/et_log_message"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="textMultiLine"
                            android:minLines="2"
                            android:maxLines="4" />

                    </com.google.android.material.textfield.TextInputLayout>

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/btn_send_log"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/send_log"
                        app:cornerRadius="8dp" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Controls Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Controls"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginBottom="12dp" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:layout_marginBottom="8dp">

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_refresh"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginEnd="4dp"
                            android:text="Refresh"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_clear_logs"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginStart="4dp"
                            android:layout_marginEnd="4dp"
                            android:text="@string/clear_logs"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_export_logs"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginStart="4dp"
                            android:text="@string/export_logs"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                    </LinearLayout>

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_settings"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginEnd="4dp"
                            android:text="@string/log_settings"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_stats"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginStart="4dp"
                            android:layout_marginEnd="4dp"
                            android:text="Statistics"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_performance_test"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginStart="4dp"
                            android:text="Perf Test"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                    </LinearLayout>

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/btn_generate_test_logs"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:text="Generate Test Logs"
                        style="@style/Widget.Material3.Button.TonalButton" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Filters Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Filters &amp; Search"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginBottom="12dp" />

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:hint="@string/search_logs"
                        android:layout_marginBottom="12dp"
                        app:startIconDrawable="@android:drawable/ic_search_category_default">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/et_search_logs"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="text"
                            android:maxLines="1" />

                    </com.google.android.material.textfield.TextInputLayout>

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical"
                            android:layout_marginEnd="8dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Log Level"
                                android:textSize="14sp"
                                android:textStyle="bold" />

                            <Spinner
                                android:id="@+id/spinner_log_level"
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content" />

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical"
                            android:layout_marginStart="8dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Tag"
                                android:textSize="14sp"
                                android:textStyle="bold" />

                            <Spinner
                                android:id="@+id/spinner_tag"
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content" />

                        </LinearLayout>

                    </LinearLayout>

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Log Count Info Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="2dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:padding="12dp"
                    android:gravity="center_vertical">

                    <TextView
                        android:id="@+id/tv_log_count"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="Total Logs: 0"
                        android:textSize="14sp"
                        android:textStyle="bold" />

                    <TextView
                        android:id="@+id/tv_filtered_count"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="Filtered: 0"
                        android:textSize="14sp"
                        android:gravity="center" />

                    <ProgressBar
                        android:id="@+id/progress_bar"
                        android:layout_width="24dp"
                        android:layout_height="24dp"
                        android:visibility="gone"
                        style="?android:attr/progressBarStyleSmall" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Logs RecyclerView Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="600dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:padding="16dp"
                        android:background="?attr/colorSurface"
                        android:gravity="center_vertical">

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="Log Entries"
                            android:textSize="18sp"
                            android:textStyle="bold" />

                        <com.google.android.material.button.MaterialButton
                            android:layout_width="wrap_content"
                            android:layout_height="32dp"
                            android:text="Auto Scroll"
                            android:textSize="12sp"
                            android:minWidth="0dp"
                            android:paddingStart="12dp"
                            android:paddingEnd="12dp"
                            style="@style/Widget.Material3.Button.TonalButton" />

                    </LinearLayout>

                    <androidx.recyclerview.widget.RecyclerView
                        android:id="@+id/recycler_view_logs"
                        android:layout_width="match_parent"
                        android:layout_height="match_parent"
                        android:padding="8dp"
                        android:scrollbars="vertical"
                        android:fadeScrollbars="false"
                        tools:listitem="@layout/item_log_entry" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

        </LinearLayout>

    </androidx.core.widget.NestedScrollView>

</androidx.coordinatorlayout.widget.CoordinatorLayout>

// FILE: Axiom/app/src/main/res/layout/activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".MainActivity">
        
    <com.google.android.material.appbar.AppBarLayout        
        android:id="@+id/appbar"        
        android:layout_width="match_parent"        
        android:layout_height="wrap_content">
            
        <com.google.android.material.appbar.MaterialToolbar            
            android:id="@+id/toolbar"            
            android:layout_width="match_parent"            
            android:layout_height="?attr/actionBarSize" />
            
    </com.google.android.material.appbar.AppBarLayout>    
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:gravity="center"
        android:padding="32dp"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/app_name"
            android:textSize="32sp"
            android:textStyle="bold"
            android:layout_marginBottom="24dp" />
            
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Advanced Logging &amp; Modding System"
            android:textSize="16sp"
            android:alpha="0.7"
            android:layout_marginBottom="48dp" />
        
        <com.google.android.material.button.MaterialButton
            android:id="@+id/logs_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/open_logs"
            android:textSize="18sp"
            android:padding="16dp"
            android:layout_marginBottom="16dp"
            app:cornerRadius="12dp"
            style="@style/Widget.Material3.Button.UnelevatedButton" />
            
        <com.google.android.material.button.MaterialButton
            android:id="@+id/modding_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/open_modding"
            android:textSize="18sp"
            android:padding="16dp"
            app:cornerRadius="12dp"
            style="@style/Widget.Material3.Button.TonalButton" />
            
    </LinearLayout>

</androidx.coordinatorlayout.widget.CoordinatorLayout>

// FILE: Axiom/app/src/main/res/layout/activity_modding.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".modding.ModdingActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/modding_activity_title" />

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <!-- Plugin System Status Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:gravity="center_vertical"
                        android:layout_marginBottom="12dp">

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="@string/plugin_system"
                            android:textSize="18sp"
                            android:textStyle="bold" />

                        <androidx.appcompat.widget.SwitchCompat
                            android:id="@+id/switch_plugin_system"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content" />

                    </LinearLayout>

                    <TextView
                        android:id="@+id/tv_system_status"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="@string/plugin_system_disabled"
                        android:textSize="14sp"
                        android:layout_marginBottom="16dp"
                        android:textColor="?attr/colorOnSurfaceVariant" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:id="@+id/tv_total_plugins"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Total: 0"
                                android:textSize="12sp"
                                android:textColor="?attr/colorOnSurfaceVariant" />

                            <TextView
                                android:id="@+id/tv_enabled_plugins"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Enabled: 0"
                                android:textSize="12sp"
                                android:textColor="?attr/colorOnSurfaceVariant" />

                        </LinearLayout>

                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">

                            <TextView
                                android:id="@+id/tv_loaded_plugins"
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="Loaded: 0"
                                android:textSize="12sp"
                                android:textColor="?attr/colorOnSurfaceVariant" />

                        </LinearLayout>

                    </LinearLayout>

                    <TextView
                        android:id="@+id/tv_plugin_directory"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Directory: ..."
                        android:textSize="10sp"
                        android:layout_marginTop="8dp"
                        android:textColor="?attr/colorOnSurfaceVariant"
                        android:maxLines="2"
                        android:ellipsize="middle" />

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Quick Actions Card -->
            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Quick Actions"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginBottom="12dp" />

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_refresh_plugins"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginEnd="8dp"
                            android:text="Refresh Status"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                        <com.google.android.material.button.MaterialButton
                            android:id="@+id/btn_scan_plugins"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:layout_marginStart="8dp"
                            android:text="@string/scan_plugins"
                            style="@style/Widget.Material3.Button.OutlinedButton" />

                    </LinearLayout>

                </LinearLayout>

            </com.google.android.material.card.MaterialCardView>

            <!-- Main Features Grid -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">

                <!-- Row 1 -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:layout_marginBottom="12dp">

                    <com.google.android.material.card.MaterialCardView
                        android:id="@+id/card_plugin_manager"
                        android:layout_width="0dp"
                        android:layout_height="120dp"
                        android:layout_weight="1"
                        android:layout_marginEnd="6dp"
                        app:cardCornerRadius="12dp"
                        app:cardElevation="4dp"
                        android:clickable="true"
                        android:focusable="true"
                        android:foreground="?attr/selectableItemBackground">

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="vertical"
                            android:gravity="center"
                            android:padding="16dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="📦"
                                android:textSize="32sp"
                                android:layout_marginBottom="8dp" />

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="@string/plugin_manager"
                                android:textSize="14sp"
                                android:textStyle="bold"
                                android:gravity="center"
                                android:textAlignment="center" />

                        </LinearLayout>

                    </com.google.android.material.card.MaterialCardView>

                    <com.google.android.material.card.MaterialCardView
                        android:id="@+id/card_plugin_development"
                        android:layout_width="0dp"
                        android:layout_height="120dp"
                        android:layout_weight="1"
                        android:layout_marginStart="6dp"
                        app:cardCornerRadius="12dp"
                        app:cardElevation="4dp"
                        android:clickable="true"
                        android:focusable="true"
                        android:foreground="?attr/selectableItemBackground">

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="vertical"
                            android:gravity="center"
                            android:padding="16dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="⚡"
                                android:textSize="32sp"
                                android:layout_marginBottom="8dp" />

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="@string/plugin_development"
                                android:textSize="14sp"
                                android:textStyle="bold"
                                android:gravity="center"
                                android:textAlignment="center" />

                        </LinearLayout>

                    </com.google.android.material.card.MaterialCardView>

                </LinearLayout>

                <!-- Row 2 -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <com.google.android.material.card.MaterialCardView
                        android:id="@+id/card_plugin_store"
                        android:layout_width="0dp"
                        android:layout_height="120dp"
                        android:layout_weight="1"
                        android:layout_marginEnd="6dp"
                        app:cardCornerRadius="12dp"
                        app:cardElevation="4dp"
                        android:clickable="true"
                        android:focusable="true"
                        android:foreground="?attr/selectableItemBackground">

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="vertical"
                            android:gravity="center"
                            android:padding="16dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="🏪"
                                android:textSize="32sp"
                                android:layout_marginBottom="8dp" />

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="@string/plugin_store"
                                android:textSize="14sp"
                                android:textStyle="bold"
                                android:gravity="center"
                                android:textAlignment="center" />

                        </LinearLayout>

                    </com.google.android.material.card.MaterialCardView>

                    <com.google.android.material.card.MaterialCardView
                        android:id="@+id/card_plugin_settings"
                        android:layout_width="0dp"
                        android:layout_height="120dp"
                        android:layout_weight="1"
                        android:layout_marginStart="6dp"
                        app:cardCornerRadius="12dp"
                        app:cardElevation="4dp"
                        android:clickable="true"
                        android:focusable="true"
                        android:foreground="?attr/selectableItemBackground">

                        <LinearLayout
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:orientation="vertical"
                            android:gravity="center"
                            android:padding="16dp">

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="⚙️"
                                android:textSize="32sp"
                                android:layout_marginBottom="8dp" />

                            <TextView
                                android:layout_width="wrap_content"
                                android:layout_height="wrap_content"
                                android:text="@string/plugin_settings"
                                android:textSize="14sp"
                                android:textStyle="bold"
                                android:gravity="center"
                                android:textAlignment="center" />

                        </LinearLayout>

                    </com.google.android.material.card.MaterialCardView>

                </LinearLayout>

            </LinearLayout>

        </LinearLayout>

    </androidx.core.widget.NestedScrollView>

</androidx.coordinatorlayout.widget.CoordinatorLayout>

