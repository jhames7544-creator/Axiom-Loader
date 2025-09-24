// File: Axiom/build.gradle

// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '8.9.1' apply false
    id 'com.android.library' version '8.9.1' apply false
         
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

// File: Axiom/gradle.properties

# Project-wide Gradle settings.
# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.
# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html
# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx512m -Dfile.encoding=UTF-8
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true
# AndroidX package structure to make it clearer which packages are bundled with the
# Android operating system, and which are packaged with your app"s APK
# https://developer.android.com/topic/libraries/support-library/androidx-rn
android.useAndroidX=true
# Kotlin code style for this project: "official" or "obsolete":
kotlin.code.style=official
# Enables namespacing of each library's R class so that its R class includes only the
# resources declared in the library itself and none from the library's dependencies,
# thereby reducing the size of the R class for that library
android.nonTransitiveRClass=true

// File: Axiom/gradlew.bat

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


// File: Axiom/settings.gradle

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

// File: Axiom/app/build.gradle

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
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
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
        // Add this line to explicitly enable BuildConfig generation
        buildConfig true
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
}


// File: Axiom/app/proguard-rules.pro

# Add project specific ProGuard rules here.
# You can control the set of applied configuration files using the
# proguardFiles setting in build.gradle.
#
# For more details, see
#   http://developer.android.com/guide/developing/tools/proguard.html

# If your project uses WebView with JS, uncomment the following
# and specify the fully qualified class name to the JavaScript interface
# class:
#-keepclassmembers class fqcn.of.javascript.interface.for.webview {
#   public *;
#}

# Uncomment this to preserve the line number information for
# debugging stack traces.
#-keepattributes SourceFile,LineNumberTable

# If you keep the line number information, uncomment this to
# hide the original source file name.
#-renamesourcefileattribute SourceFile

// File: Axiom/app/src/main/AndroidManifest.xml

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
    
    <application 
        android:name=".AxiomApplication"
        android:allowBackup="true" 
        android:icon="@mipmap/ic_launcher" 
        android:roundIcon="@mipmap/ic_launcher" 
        android:label="@string/app_name" 
        android:supportsRtl="true" 
        android:theme="@style/AppTheme"
        android:requestLegacyExternalStorage="true">
        
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
            
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.axiomloader.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>
</manifest>

// File: Axiom/app/src/main/java/com/axiomloader/AxiomApplication.java

package com.axiomloader;

import android.app.Application;
import com.axiomloader.utils.LogUtils;
import timber.log.Timber;

public class AxiomApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // Initialize LogUtils with the application context
        LogUtils.initialize(this);

        // Check for debug build to plant the Timber tree
        // FIX: Ensure BuildConfig is imported correctly
        if (com.axiomloader.BuildConfig.DEBUG) {
            Timber.plant(new Timber.DebugTree());
        }

        // You can also use LogUtils directly for logging application lifecycle events
        LogUtils.logInfo("AxiomApplication", "Application onCreate() called.");
    }
}


// File: Axiom/app/src/main/java/com/axiomloader/MainActivity.java

package com.axiomloader;

import android.content.Intent;
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import com.axiomloader.databinding.ActivityMainBinding;
import com.axiomloader.ui.LoggingActivity;
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

// File: Axiom/app/src/main/java/com/axiomloader/ui/LogAdapter.java

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

// File: Axiom/app/src/main/java/com/axiomloader/ui/LoggingActivity.java

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

// File: Axiom/app/src/main/java/com/axiomloader/utils/LogUtils.java

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


// File: Axiom/app/src/main/jni/Android.mk

LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := tomaslib
LOCAL_SRC_FILES := tomaslib.cpp

include $(BUILD_SHARED_LIBRARY)

// File: Axiom/app/src/main/jni/Application.mk

APP_ABI := all
APP_PLATFORM := android-21
APP_STL := c++_shared
APP_CPPFLAGS += -std=c++17

// File: Axiom/app/src/main/jni/tomaslib.cpp

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

// File: Axiom/app/src/main/jni/tomaslib.h

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

// File: Axiom/app/src/main/res/drawable/ic_launcher_background.xml

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


// File: Axiom/app/src/main/res/drawable-v24/ic_launcher_foreground.xml

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

// File: Axiom/app/src/main/res/layout/activity_logging.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
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

// File: Axiom/app/src/main/res/layout/activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
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
            android:text="Advanced Logging System"
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
            app:cornerRadius="12dp"
            style="@style/Widget.Material3.Button.UnelevatedButton" />
            
    </LinearLayout>

</androidx.coordinatorlayout.widget.CoordinatorLayout>

// File: Axiom/app/src/main/res/layout/dialog_log_settings.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Configure Logging Settings"
        android:textSize="16sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_file_logging"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/enable_file_logging"
        android:layout_marginBottom="8dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_compression"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/log_compression"
        android:layout_marginBottom="8dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_encryption"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/log_encryption"
        android:layout_marginBottom="8dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_crash_reporting"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/enable_crash_reporting"
        android:layout_marginBottom="8dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_performance_monitoring"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Performance Monitoring"
        android:layout_marginBottom="8dp" />

    <androidx.appcompat.widget.SwitchCompat
        android:id="@+id/switch_auto_cleanup"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/auto_delete_old_logs"
        android:layout_marginBottom="8dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Advanced Settings"
        android:textSize="14sp"
        android:textStyle="bold"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="8dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical"
        android:layout_marginBottom="8dp">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/max_log_files" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="10"
            android:textStyle="bold" />

    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/log_file_size" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="5 MB"
            android:textStyle="bold" />

    </LinearLayout>

</LinearLayout>


// File: Axiom/app/src/main/res/layout/item_log_entry.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="8dp"
    android:background="?attr/selectableItemBackground"
    android:minHeight="48dp">

    <!-- Level Indicator -->
    <View
        android:id="@+id/level_indicator"
        android:layout_width="4dp"
        android:layout_height="match_parent"
        android:layout_marginEnd="8dp"
        android:background="#2196F3" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- Main Log Line -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <!-- Timestamp -->
            <TextView
                android:id="@+id/tv_timestamp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="12:34:56.789"
                android:textSize="11sp"
                android:textColor="?android:attr/textColorSecondary"
                android:fontFamily="monospace"
                android:layout_marginEnd="8dp"
                tools:text="12:34:56.789" />

            <!-- Log Level -->
            <TextView
                android:id="@+id/tv_level"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="I"
                android:textSize="12sp"
                android:textStyle="bold"
                android:fontFamily="monospace"
                android:layout_marginEnd="4dp"
                android:minWidth="16dp"
                android:gravity="center"
                tools:text="I" />

            <!-- Separator -->
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="/"
                android:textSize="12sp"
                android:textColor="?android:attr/textColorSecondary"
                android:layout_marginEnd="4dp" />

            <!-- Tag -->
            <TextView
                android:id="@+id/tv_tag"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="TAG"
                android:textSize="12sp"
                android:textStyle="bold"
                android:maxWidth="80dp"
                android:ellipsize="end"
                android:singleLine="true"
                android:layout_marginEnd="8dp"
                tools:text="MyTag" />

            <!-- Separator -->
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text=":"
                android:textSize="12sp"
                android:textColor="?android:attr/textColorSecondary"
                android:layout_marginEnd="8dp" />

            <!-- Message -->
            <TextView
                android:id="@+id/tv_message"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Log message here"
                android:textSize="12sp"
                android:textColor="?android:attr/textColorPrimary"
                android:maxLines="3"
                android:ellipsize="end"
                tools:text="This is a sample log message that might be quite long and could wrap to multiple lines" />

        </LinearLayout>

        <!-- Additional Info Line (Thread + Location) -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="2dp"
            android:gravity="center_vertical">

            <!-- Thread Info -->
            <TextView
                android:id="@+id/tv_thread"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="[main]"
                android:textSize="10sp"
                android:textColor="?android:attr/textColorSecondary"
                android:fontFamily="monospace"
                android:layout_marginEnd="8dp"
                android:visibility="gone"
                tools:text="[main]"
                tools:visibility="visible" />

            <!-- Location Info -->
            <TextView
                android:id="@+id/tv_location"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="(MainActivity.onCreate:42)"
                android:textSize="10sp"
                android:textColor="?android:attr/textColorSecondary"
                android:fontFamily="monospace"
                android:ellipsize="start"
                android:singleLine="true"
                android:visibility="gone"
                tools:text="(MainActivity.onCreate:42)"
                tools:visibility="visible" />

        </LinearLayout>

    </LinearLayout>

</LinearLayout>

// File: Axiom/app/src/main/res/menu/logging_menu.xml

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_refresh"
        android:title="Refresh"
        android:icon="@android:drawable/ic_menu_rotate"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_clear_search"
        android:title="Clear Filters"
        android:icon="@android:drawable/ic_menu_close_clear_cancel"
        app:showAsAction="ifRoom" />

</menu>

// File: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

// File: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

// File: Axiom/app/src/main/res/values/colors.xml

<?xml version="1.0" encoding="utf-8"?>
<resources></resources>

// File: Axiom/app/src/main/res/values/strings.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Axiom</string>
    <string name="logging_activity_title">Advanced Logging System</string>
    <string name="open_logs">Open Logs</string>
    <string name="log_level_verbose">VERBOSE</string>
    <string name="log_level_debug">DEBUG</string>
    <string name="log_level_info">INFO</string>
    <string name="log_level_warn">WARN</string>
    <string name="log_level_error">ERROR</string>
    <string name="log_level_wtf">WTF</string>
    <string name="log_message_hint">Enter your log message here...</string>
    <string name="log_tag_hint">Log Tag</string>
    <string name="send_log">Send Log</string>
    <string name="clear_logs">Clear Logs</string>
    <string name="export_logs">Export Logs</string>
    <string name="filter_logs">Filter Logs</string>
    <string name="search_logs">Search Logs</string>
    <string name="log_settings">Log Settings</string>
    <string name="enable_file_logging">Enable File Logging</string>
    <string name="enable_crash_reporting">Enable Crash Reporting</string>
    <string name="max_log_files">Max Log Files</string>
    <string name="log_file_size">Log File Size (MB)</string>
    <string name="auto_delete_old_logs">Auto Delete Old Logs</string>
    <string name="log_format">Log Format</string>
    <string name="timestamp_format">Timestamp Format</string>
    <string name="enable_colors">Enable Colors</string>
    <string name="enable_threading">Enable Threading Info</string>
    <string name="enable_stacktrace">Enable Stack Trace</string>
    <string name="log_compression">Enable Log Compression</string>
    <string name="log_encryption">Enable Log Encryption</string>
    <string name="logs_exported_successfully">Logs exported successfully</string>
    <string name="logs_cleared_successfully">Logs cleared successfully</string>
    <string name="no_logs_found">No logs found</string>
    <string name="log_file_created">Log file created: %1$s</string>
    <string name="crash_detected">Crash detected and logged</string>
    <string name="memory_usage">Memory Usage: %1$s MB</string>
    <string name="cpu_usage">CPU Usage: %1$s%%</string>
    <string name="battery_level">Battery: %1$s%%</string>
    <string name="network_status">Network: %1$s</string>
    <string name="storage_available">Storage: %1$s GB available</string>
</resources>

// File: Axiom/app/src/main/res/values/themes.xml

<resources xmlns:tools="http://schemas.android.com/tools">
  <!-- Base application theme. -->
  <style name="Base.AppTheme" parent="Theme.Material3.DayNight.NoActionBar">
    <!-- Customize your theme here. -->
    <!-- <item name="colorPrimary">@color/my_light_primary</item> -->
  </style>

  <style name="AppTheme" parent="Base.AppTheme" />
</resources>

// File: Axiom/app/src/main/res/values-night/colors.xml

<?xml version="1.0" encoding="utf-8"?>
<resources></resources>

// File: Axiom/app/src/main/res/values-night/themes.xml

<resources xmlns:tools="http://schemas.android.com/tools">
  <!-- Base application theme. -->
  <style name="Base.AppTheme" parent="Theme.Material3.DayNight.NoActionBar">
    <!-- Customize your theme here. -->
    <!-- <item name="colorPrimary">@color/my_light_primary</item> -->
  </style>

  <style name="AppTheme" parent="Base.AppTheme" />
</resources>

// File: Axiom/app/src/main/res/xml/backup_rules.xml

<?xml version="1.0" encoding="utf-8"?><!--
   Sample backup rules file; uncomment and customize as necessary.
   See https://developer.android.com/guide/topics/data/autobackup
   for details.
   Note: This file is ignored for devices older that API 31
   See https://developer.android.com/about/versions/12/backup-restore
-->
<full-backup-content>
  <!--
   <include domain="sharedpref" path="."/>
   <exclude domain="sharedpref" path="device.xml"/>
-->
</full-backup-content>

// File: Axiom/app/src/main/res/xml/data_extraction_rules.xml

<?xml version="1.0" encoding="utf-8"?><!--
   Sample data extraction rules file; uncomment and customize as necessary.
   See https://developer.android.com/about/versions/12/backup-restore#xml-changes
   for details.
-->
<data-extraction-rules>
  <cloud-backup>
    <!-- TODO: Use <include> and <exclude> to control what is backed up.
        <include .../>
        <exclude .../>
        -->
  </cloud-backup>
  <!--
    <device-transfer>
        <include .../>
        <exclude .../>
    </device-transfer>
    -->
</data-extraction-rules>

// File: Axiom/app/src/main/res/xml/file_paths.xml

<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path
        name="axiom_logs"
        path="axiom_logs/" />
    <external-files-path
        name="exports"
        path="exports/" />
    <files-path
        name="internal_logs"
        path="axiom_logs/" />
</paths>

// File: Axiom/gradle/wrapper/gradle-wrapper.properties

distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-bin.zip
networkTimeout=10000
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists

