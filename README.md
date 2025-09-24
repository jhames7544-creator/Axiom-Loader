==== BEGIN FILE: Axiom/app/build.gradle ====
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.axiomloader'
    compileSdk 34

    defaultConfig {
        applicationId "com.axiomloader"
        minSdk 26
        targetSdk 34
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        
        // Enable multiDex for Log4j2
        multiDexEnabled true
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    packagingOptions {
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/ASL2.0'
        exclude 'META-INF/log4j-provider.properties'
        pickFirst 'META-INF/log4j2.xml'
        pickFirst 'META-INF/org/apache/logging/log4j/core/config/plugins/Log4j2Plugins.dat'
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.10.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
    implementation 'androidx.cardview:cardview:1.0.0'
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
    
    // MultiDex support
    implementation 'androidx.multidex:multidex:2.0.1'
    
    // Log4j2 Core Dependencies
    implementation 'org.apache.logging.log4j:log4j-core:2.20.0'
    implementation 'org.apache.logging.log4j:log4j-api:2.20.0'
    implementation 'org.apache.logging.log4j:log4j-slf4j-impl:2.20.0'
    
    // Additional Log4j2 Components
    implementation 'org.apache.logging.log4j:log4j-layout-template-json:2.20.0'
    implementation 'org.apache.logging.log4j:log4j-jcl:2.20.0'
    
    // JSON Processing for advanced logging
    implementation 'com.fasterxml.jackson.core:jackson-core:2.15.2'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.2'
    implementation 'com.fasterxml.jackson.core:jackson-annotations:2.15.2'
    
    // Apache Commons for utilities
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    implementation 'commons-io:commons-io:2.11.0'
    
    // Testing
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
==== END FILE: Axiom/app/build.gradle ====

==== BEGIN FILE: Axiom/app/proguard-rules.pro ====
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
==== END FILE: Axiom/app/proguard-rules.pro ====

==== BEGIN FILE: Axiom/app/src/main/AndroidManifest.xml ====
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Permissions for advanced logging features -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"
        tools:ignore="ScopedStorage" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    
    <application
        android:name=".AxiomApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:requestLegacyExternalStorage="true"
        android:theme="@style/Theme.AxiomLoader"
        tools:targetApi="31">
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:theme="@style/Theme.AxiomLoader">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name=".ui.LoggingActivity"
            android:exported="false"
            android:screenOrientation="portrait"
            android:theme="@style/Theme.AxiomLoader"
            android:parentActivityName=".MainActivity" />
            
        <!-- File Provider for log file sharing -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_provider_paths" />
        </provider>
        
        <!-- Boot receiver for log system initialization -->
        <receiver
            android:name=".utils.LogBootReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="1000">
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.MY_PACKAGE_REPLACED" />
                <action android:name="android.intent.action.PACKAGE_REPLACED" />
                <data android:scheme="package" />
            </intent-filter>
        </receiver>
        
        <!-- Log cleanup service -->
        <service
            android:name=".utils.LogCleanupService"
            android:enabled="true"
            android:exported="false" />

    </application>

</manifest>
==== END FILE: Axiom/app/src/main/AndroidManifest.xml ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/AxiomApplication.java ====
package com.axiomloader;

import android.app.Application;
import android.content.Context;
import androidx.multidex.MultiDex;
import com.axiomloader.utils.LogUtils;

public class AxiomApplication extends Application {
    
    private LogUtils logUtils;
    
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(this);
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        // Initialize LogUtils early
        logUtils = LogUtils.getInstance(this);
        logUtils.info("AxiomApplication", "Application created successfully");
        
        // Set up uncaught exception handler
        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(Thread thread, Throwable throwable) {
                logUtils.fatal("UncaughtException", 
                        "Uncaught exception in thread " + thread.getName(), throwable);
                
                // Call default handler
                Thread.getDefaultUncaughtExceptionHandler().uncaughtException(thread, throwable);
            }
        });
    }
    
    @Override
    public void onTerminate() {
        super.onTerminate();
        
        if (logUtils != null) {
            logUtils.info("AxiomApplication", "Application terminating");
            logUtils.shutdown();
        }
    }
    
    @Override
    public void onLowMemory() {
        super.onLowMemory();
        
        if (logUtils != null) {
            logUtils.warn("AxiomApplication", "Low memory warning received");
        }
    }
    
    @Override
    public void onTrimMemory(int level) {
        super.onTrimMemory(level);
        
        if (logUtils != null) {
            logUtils.warn("AxiomApplication", "Memory trim requested, level: " + level);
        }
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/AxiomApplication.java ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/MainActivity.java ====
package com.axiomloader;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;
import com.axiomloader.ui.LoggingActivity;
import com.axiomloader.utils.LogUtils;

public class MainActivity extends AppCompatActivity {
    
    private Button logsButton;
    private LogUtils logUtils;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize LogUtils
        logUtils = LogUtils.getInstance(this);
        logUtils.info("MainActivity", "Application started");
        
        initializeViews();
        setupClickListeners();
        
        logUtils.debug("MainActivity", "MainActivity onCreate completed");
    }
    
    private void initializeViews() {
        logsButton = findViewById(R.id.btn_logs);
        
        // Style the logs button
        logsButton.setBackground(ContextCompat.getDrawable(this, android.R.drawable.btn_default));
        logsButton.setTextColor(ContextCompat.getColor(this, android.R.color.white));
        logsButton.setTextSize(16f);
        logsButton.setPadding(32, 16, 32, 16);
        
        logUtils.debug("MainActivity", "Views initialized");
    }
    
    private void setupClickListeners() {
        logsButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                logUtils.info("MainActivity", "Logs button clicked - navigating to LoggingActivity");
                
                Intent intent = new Intent(MainActivity.this, LoggingActivity.class);
                startActivity(intent);
            }
        });
        
        logUtils.debug("MainActivity", "Click listeners setup completed");
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        logUtils.debug("MainActivity", "MainActivity resumed");
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        logUtils.debug("MainActivity", "MainActivity paused");
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        logUtils.info("MainActivity", "MainActivity destroyed");
        
        // Don't shutdown LogUtils here as it's a singleton and might be used by other activities
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/MainActivity.java ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/ui/LoggingActivity.java ====
package com.axiomloader.ui;

import android.app.AlertDialog;
import android.content.ClipData;
import android.content.ClipboardManager;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.SharedPreferences;
import android.graphics.Color;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.text.Editable;
import android.text.Html;
import android.text.Spannable;
import android.text.SpannableString;
import android.text.TextWatcher;
import android.text.style.ForegroundColorSpan;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.CompoundButton;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.ProgressBar;
import android.widget.ScrollView;
import android.widget.SeekBar;
import android.widget.Spinner;
import android.widget.Switch;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;
import androidx.swiperefreshlayout.widget.SwipeRefreshLayout;

import com.axiomloader.R;
import com.axiomloader.utils.LogUtils;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Locale;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.regex.Pattern;

public class LoggingActivity extends AppCompatActivity {
    
    // UI Components
    private TextView logDisplay;
    private ScrollView logScrollView;
    private EditText searchEditText;
    private Spinner logLevelSpinner;
    private Spinner threadFilterSpinner;
    private Spinner loggerFilterSpinner;
    private Button btnClearLogs, btnRefreshLogs, btnPauseLogs, btnExportLogs;
    private Button btnGenerateTestLogs, btnSettings, btnStats;
    private CheckBox cbAutoScroll, cbRegexSearch, cbCaseSensitive, cbRealTimeLogging;
    private Switch switchColorCoding, switchDarkTheme;
    private SeekBar seekBarBufferSize;
    private ProgressBar progressBar;
    private SwipeRefreshLayout swipeRefreshLayout;
    private LinearLayout statsContainer, filtersContainer, settingsContainer;
    
    // Statistics TextViews
    private TextView tvTotalLogs, tvErrorCount, tvWarnCount, tvFileSize, tvOldestLog, tvNewestLog;
    
    // Core Components
    private LogUtils logUtils;
    private SharedPreferences prefs;
    private Handler mainHandler;
    private ExecutorService executorService;
    
    // State Variables
    private boolean isPaused = false;
    private boolean autoScroll = true;
    private boolean realTimeLogging = true;
    private String currentSearchQuery = "";
    private String currentLogLevel = "ALL";
    private String currentThreadFilter = "ALL";
    private String currentLoggerFilter = "ALL";
    private List<String> logEntries = new ArrayList<>();
    private List<String> filteredLogEntries = new ArrayList<>();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_logging);
        
        initializeComponents();
        initializeViews();
        setupEventListeners();
        loadSettings();
        startRealTimeUpdates();
        
        LogUtils.getInstance(this).info("LoggingActivity", "LoggingActivity created successfully");
    }
    
    private void initializeComponents() {
        logUtils = LogUtils.getInstance(this);
        prefs = getSharedPreferences("LoggingPrefs", Context.MODE_PRIVATE);
        mainHandler = new Handler(Looper.getMainLooper());
        executorService = Executors.newSingleThreadExecutor();
        
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(getString(R.string.logging_title));
        }
    }
    
    private void initializeViews() {
        logDisplay = findViewById(R.id.tv_log_display);
        logScrollView = findViewById(R.id.scroll_log_display);
        swipeRefreshLayout = findViewById(R.id.swipe_refresh_layout);
        progressBar = findViewById(R.id.progress_bar);
        searchEditText = findViewById(R.id.et_search_logs);
        logLevelSpinner = findViewById(R.id.spinner_log_level);
        threadFilterSpinner = findViewById(R.id.spinner_thread_filter);
        loggerFilterSpinner = findViewById(R.id.spinner_logger_filter);
        
        btnClearLogs = findViewById(R.id.btn_clear_logs);
        btnRefreshLogs = findViewById(R.id.btn_refresh_logs);
        btnPauseLogs = findViewById(R.id.btn_pause_logs);
        btnExportLogs = findViewById(R.id.btn_export_logs);
        btnGenerateTestLogs = findViewById(R.id.btn_generate_test_logs);
        btnSettings = findViewById(R.id.btn_settings);
        btnStats = findViewById(R.id.btn_stats);
        
        cbAutoScroll = findViewById(R.id.cb_auto_scroll);
        cbRegexSearch = findViewById(R.id.cb_regex_search);
        cbCaseSensitive = findViewById(R.id.cb_case_sensitive);
        cbRealTimeLogging = findViewById(R.id.cb_real_time_logging);
        switchColorCoding = findViewById(R.id.switch_color_coding);
        switchDarkTheme = findViewById(R.id.switch_dark_theme);
        
        seekBarBufferSize = findViewById(R.id.seekbar_buffer_size);
        statsContainer = findViewById(R.id.stats_container);
        filtersContainer = findViewById(R.id.filters_container);
        settingsContainer = findViewById(R.id.settings_container);
        
        tvTotalLogs = findViewById(R.id.tv_total_logs);
        tvErrorCount = findViewById(R.id.tv_error_count);
        tvWarnCount = findViewById(R.id.tv_warn_count);
        tvFileSize = findViewById(R.id.tv_file_size);
        tvOldestLog = findViewById(R.id.tv_oldest_log);
        tvNewestLog = findViewById(R.id.tv_newest_log);
        
        setupSpinners();
        setupDefaultValues();
        
        // Debug check - verify all critical UI elements are found
        logUtils.debug("LoggingActivity", "UI initialization check:");
        logUtils.debug("LoggingActivity", "Settings button: " + (btnSettings != null ? "found" : "NOT FOUND"));
        logUtils.debug("LoggingActivity", "Stats button: " + (btnStats != null ? "found" : "NOT FOUND"));
        logUtils.debug("LoggingActivity", "Settings container: " + (settingsContainer != null ? "found" : "NOT FOUND"));
        logUtils.debug("LoggingActivity", "Stats container: " + (statsContainer != null ? "found" : "NOT FOUND"));
    }
    
    private void setupSpinners() {
        String[] logLevels = {"ALL", "TRACE", "DEBUG", "INFO", "WARN", "ERROR", "FATAL"};
        ArrayAdapter<String> logLevelAdapter = new ArrayAdapter<>(this, 
            android.R.layout.simple_spinner_item, logLevels);
        logLevelAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        logLevelSpinner.setAdapter(logLevelAdapter);
        
        List<String> threads = logUtils.getAllThreadNames();
        threads.add(0, "ALL");
        ArrayAdapter<String> threadAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item, threads);
        threadAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        threadFilterSpinner.setAdapter(threadAdapter);
        
        List<String> loggers = logUtils.getAllLoggerNames();
        loggers.add(0, "ALL");
        ArrayAdapter<String> loggerAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item, loggers);
        loggerAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        loggerFilterSpinner.setAdapter(loggerAdapter);
    }
    
    private void setupDefaultValues() {
        cbAutoScroll.setChecked(true);
        cbRealTimeLogging.setChecked(true);
        switchColorCoding.setChecked(true);
        seekBarBufferSize.setProgress(50);
    }
    
    private void setupEventListeners() {
        swipeRefreshLayout.setOnRefreshListener(() -> {
            refreshLogDisplay();
            swipeRefreshLayout.setRefreshing(false);
        });
        
        searchEditText.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                currentSearchQuery = s.toString();
                filterLogs();
            }
            
            @Override
            public void afterTextChanged(Editable s) {}
        });
        
        logLevelSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                String[] logLevels = {"ALL", "TRACE", "DEBUG", "INFO", "WARN", "ERROR", "FATAL"};
                currentLogLevel = logLevels[position];
                filterLogs();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
        
        btnClearLogs.setOnClickListener(v -> showClearLogsDialog());
        btnRefreshLogs.setOnClickListener(v -> refreshLogDisplay());
        btnPauseLogs.setOnClickListener(v -> togglePauseResume());
        btnExportLogs.setOnClickListener(v -> showExportDialog());
        btnGenerateTestLogs.setOnClickListener(v -> generateTestLogs());
        
        btnSettings.setOnClickListener(v -> {
            logUtils.debug("LoggingActivity", "Settings button clicked");
            toggleSettingsVisibility();
        });
        
        btnStats.setOnClickListener(v -> {
            logUtils.debug("LoggingActivity", "Stats button clicked"); 
            toggleStatsVisibility();
        });
        
        cbAutoScroll.setOnCheckedChangeListener((buttonView, isChecked) -> {
            autoScroll = isChecked;
            if (autoScroll) scrollToBottom();
        });
        
        cbRealTimeLogging.setOnCheckedChangeListener((buttonView, isChecked) -> {
            realTimeLogging = isChecked;
            if (realTimeLogging) {
                startRealTimeUpdates();
            } else {
                stopRealTimeUpdates();
            }
        });
        
        switchColorCoding.setOnCheckedChangeListener((buttonView, isChecked) -> {
            refreshLogDisplay();
        });
        
        switchDarkTheme.setOnCheckedChangeListener((buttonView, isChecked) -> {
            applyTheme(isChecked);
        });
    }
    
    private void startRealTimeUpdates() {
        stopRealTimeUpdates();
        if (realTimeLogging && !isPaused) {
            mainHandler.post(logUpdateRunnable);
        }
    }
    
    private void stopRealTimeUpdates() {
        if (mainHandler != null) {
            mainHandler.removeCallbacks(logUpdateRunnable);
        }
    }
    
    private final Runnable logUpdateRunnable = new Runnable() {
        @Override
        public void run() {
            if (realTimeLogging && !isPaused) {
                refreshLogDisplay();
                mainHandler.postDelayed(this, 1000);
            }
        }
    };
    
    private void refreshLogDisplay() {
        executorService.execute(() -> {
            try {
                logEntries = logUtils.getAllLogsFormatted();
                
                mainHandler.post(() -> {
                    filterLogs();
                    updateStatistics();
                    if (autoScroll) scrollToBottom();
                });
            } catch (Exception e) {
                mainHandler.post(() -> {
                    Toast.makeText(this, "Error refreshing logs: " + e.getMessage(), Toast.LENGTH_SHORT).show();
                });
            }
        });
    }
    
    private void filterLogs() {
        filteredLogEntries.clear();
        
        for (String log : logEntries) {
            if (passesAllFilters(log)) {
                filteredLogEntries.add(log);
            }
        }
        
        displayFilteredLogs();
    }
    
    private boolean passesAllFilters(String log) {
        if (!currentSearchQuery.isEmpty()) {
            boolean matches = cbCaseSensitive.isChecked() ? 
                log.contains(currentSearchQuery) : 
                log.toLowerCase().contains(currentSearchQuery.toLowerCase());
            
            if (cbRegexSearch.isChecked()) {
                try {
                    Pattern pattern = Pattern.compile(currentSearchQuery, 
                        cbCaseSensitive.isChecked() ? 0 : Pattern.CASE_INSENSITIVE);
                    matches = pattern.matcher(log).find();
                } catch (Exception e) {
                    matches = false;
                }
            }
            
            if (!matches) return false;
        }
        
        if (!"ALL".equals(currentLogLevel) && !log.contains(currentLogLevel)) {
            return false;
        }
        
        if (!"ALL".equals(currentThreadFilter) && !log.contains(currentThreadFilter)) {
            return false;
        }
        
        if (!"ALL".equals(currentLoggerFilter) && !log.contains(currentLoggerFilter)) {
            return false;
        }
        
        return true;
    }
    
    /**
     * IMPROVED: Display filtered logs with much better formatting
     */
    private void displayFilteredLogs() {
        StringBuilder displayText = new StringBuilder();
        
        if (filteredLogEntries.isEmpty()) {
            displayText.append("No logs match the current filters.\n\n");
            displayText.append("Try:\n");
            displayText.append("• Clearing search terms\n");
            displayText.append("• Changing log level filter\n");
            displayText.append("• Generating test logs\n");
            displayText.append("• Refreshing the display");
        } else {
            for (String log : filteredLogEntries) {
                String cleanedLog = cleanLogEntry(log);
                displayText.append(cleanedLog).append("\n\n");
            }
        }
        
        if (switchColorCoding.isChecked()) {
            logDisplay.setText(Html.fromHtml(colorCodeLogs(displayText.toString())));
        } else {
            logDisplay.setText(displayText.toString());
        }
    }
    
    /**
     * IMPROVED: Clean log entries for better readability
     */
    private String cleanLogEntry(String rawLog) {
        try {
            if (rawLog == null || rawLog.trim().isEmpty()) {
                return "";
            }
            
            String cleaned = rawLog.trim();
            
            // Remove excessive session IDs and thread info for cleaner display
            cleaned = cleaned.replaceAll("\\[AX_\\d+_[^\\]]+\\]\\s*", "");
            cleaned = cleaned.replaceAll("\\[LogUtils-Worker\\]\\s*", "");
            cleaned = cleaned.replaceAll("\\[LogUtils-Scheduled\\]\\s*", "");
            
            // Clean up package names for better readability
            cleaned = cleaned.replace("com.axiomloader.", "");
            
            // Ensure proper spacing around log levels
            cleaned = cleaned.replaceAll("\\[([A-Z]+)\\]", " [$1] ");
            
            // Clean up multiple spaces
            cleaned = cleaned.replaceAll("\\s+", " ");
            
            // Ensure timestamp is properly formatted
            cleaned = cleaned.replaceAll("(\\d{2}:\\d{2}:\\d{2}\\.\\d{3})", "[$1]");
            
            return cleaned.trim();
        } catch (Exception e) {
            return rawLog; // Return original if cleaning fails
        }
    }
    
    /**
     * IMPROVED: Color coding with better contrast and HTML formatting
     */
    private String colorCodeLogs(String logs) {
        // Convert line breaks to HTML
        logs = logs.replace("\n\n", "<br/><br/>");
        logs = logs.replace("\n", "<br/>");
        
        // Color code by log level with better contrast for dark theme
        logs = logs.replaceAll("\\[(ERROR|FATAL)\\]", "<font color='#FF4444'><b>[$1]</b></font>");
        logs = logs.replaceAll("\\[WARN\\]", "<font color='#FFAA00'><b>[WARN]</b></font>");
        logs = logs.replaceAll("\\[INFO\\]", "<font color='#00DD00'><b>[INFO]</b></font>");
        logs = logs.replaceAll("\\[DEBUG\\]", "<font color='#00AAFF'><b>[DEBUG]</b></font>");
        logs = logs.replaceAll("\\[TRACE\\]", "<font color='#AA00FF'><b>[TRACE]</b></font>");
        
        // Highlight timestamps
        logs = logs.replaceAll("(\\[\\d{2}:\\d{2}:\\d{2}\\.\\d{3}\\])", 
                "<font color='#CCCCCC'><i>$1</i></font>");
        
        // Highlight logger names
        logs = logs.replaceAll("\\b(MainActivity|LogUtils|DatabaseManager|NetworkService)\\b", 
                "<font color='#FFDD00'><b>$1</b></font>");
        
        // Highlight error keywords
        logs = logs.replaceAll("\\b(Exception|Error|Failed|Timeout)\\b", 
                "<font color='#FF6666'><b>$1</b></font>");
        
        return logs;
    }
    
    private void updateStatistics() {
        int totalLogs = logEntries.size();
        int errorCount = 0;
        int warnCount = 0;
        
        for (String log : logEntries) {
            if (log.contains("ERROR") || log.contains("FATAL")) errorCount++;
            if (log.contains("WARN")) warnCount++;
        }
        
        long fileSize = logUtils.getTotalLogFileSize();
        String oldestLog = logUtils.getOldestLogTimestamp();
        String newestLog = logUtils.getNewestLogTimestamp();
        
        tvTotalLogs.setText("Total: " + totalLogs);
        tvErrorCount.setText("Errors: " + errorCount);
        tvWarnCount.setText("Warnings: " + warnCount);
        tvFileSize.setText("Size: " + formatFileSize(fileSize));
        tvOldestLog.setText("Oldest: " + oldestLog);
        tvNewestLog.setText("Newest: " + newestLog);
    }
    
    private String formatFileSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return String.format("%.1f KB", bytes / 1024.0);
        return String.format("%.1f MB", bytes / (1024.0 * 1024.0));
    }
    
    private void togglePauseResume() {
        isPaused = !isPaused;
        btnPauseLogs.setText(isPaused ? "Resume" : "Pause");
        
        if (isPaused) {
            stopRealTimeUpdates();
            Toast.makeText(this, "Logging paused", Toast.LENGTH_SHORT).show();
        } else {
            startRealTimeUpdates();
            Toast.makeText(this, "Logging resumed", Toast.LENGTH_SHORT).show();
        }
    }
    
    private void showClearLogsDialog() {
        new AlertDialog.Builder(this)
                .setTitle("Clear All Logs")
                .setMessage("Are you sure you want to clear all logs? This action cannot be undone.")
                .setPositiveButton("Clear", (dialog, which) -> {
                    logUtils.clearAllLogs();
                    refreshLogDisplay();
                    Toast.makeText(this, "Logs cleared", Toast.LENGTH_SHORT).show();
                })
                .setNegativeButton("Cancel", null)
                .show();
    }
    
    private void showExportDialog() {
        String[] exportOptions = {"TXT", "JSON", "XML", "CSV", "HTML"};
        
        new AlertDialog.Builder(this)
                .setTitle("Export Format")
                .setItems(exportOptions, (dialog, which) -> {
                    String format = exportOptions[which];
                    exportLogs(format);
                })
                .show();
    }
    
    private void exportLogs(String format) {
        executorService.execute(() -> {
            try {
                File exportedFile = logUtils.exportLogs(format, filteredLogEntries);
                
                mainHandler.post(() -> {
                    Toast.makeText(this, "Logs exported to: " + exportedFile.getName(), 
                            Toast.LENGTH_LONG).show();
                    
                    shareFile(exportedFile);
                });
            } catch (Exception e) {
                mainHandler.post(() -> {
                    Toast.makeText(this, "Export failed: " + e.getMessage(), Toast.LENGTH_SHORT).show();
                });
            }
        });
    }
    
    private void shareFile(File file) {
        try {
            Uri fileUri = FileProvider.getUriForFile(this, 
                    getPackageName() + ".fileprovider", file);
            
            Intent shareIntent = new Intent(Intent.ACTION_SEND);
            shareIntent.setType("*/*");
            shareIntent.putExtra(Intent.EXTRA_STREAM, fileUri);
            shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            
            startActivity(Intent.createChooser(shareIntent, "Share Log File"));
        } catch (Exception e) {
            Toast.makeText(this, "Unable to share file: " + e.getMessage(), Toast.LENGTH_SHORT).show();
        }
    }
    
    private void generateTestLogs() {
        executorService.execute(() -> {
            logUtils.generateTestLogs(20); // Reduced count for better performance
            mainHandler.post(() -> {
                refreshLogDisplay();
                Toast.makeText(this, "Test logs generated", Toast.LENGTH_SHORT).show();
            });
        });
    }
    
    private void toggleSettingsVisibility() {
        if (settingsContainer != null) {
            boolean isVisible = settingsContainer.getVisibility() == View.VISIBLE;
            settingsContainer.setVisibility(isVisible ? View.GONE : View.VISIBLE);
            
            // Update button text to show current state
            btnSettings.setText(isVisible ? "Settings" : "Hide Settings");
            
            // Debug logging
            logUtils.debug("LoggingActivity", "Settings container toggled: " + (isVisible ? "hidden" : "shown"));
        } else {
            logUtils.error("LoggingActivity", "Settings container is null!");
            Toast.makeText(this, "Settings container not found", Toast.LENGTH_SHORT).show();
        }
    }
    
    private void toggleStatsVisibility() {
        if (statsContainer != null) {
            boolean isVisible = statsContainer.getVisibility() == View.VISIBLE;
            statsContainer.setVisibility(isVisible ? View.GONE : View.VISIBLE);
            
            // Update button text to show current state
            btnStats.setText(isVisible ? "Statistics" : "Hide Stats");
            
            // Debug logging
            logUtils.debug("LoggingActivity", "Stats container toggled: " + (isVisible ? "hidden" : "shown"));
        } else {
            logUtils.error("LoggingActivity", "Stats container is null!");
            Toast.makeText(this, "Stats container not found", Toast.LENGTH_SHORT).show();
        }
    }
    
    private void applyTheme(boolean darkTheme) {
        int bgColor = darkTheme ? Color.BLACK : Color.WHITE;
        int textColor = darkTheme ? Color.WHITE : Color.BLACK;
        
        findViewById(R.id.main_layout).setBackgroundColor(bgColor);
        logDisplay.setTextColor(textColor);
        searchEditText.setTextColor(textColor);
    }
    
    private void scrollToBottom() {
        logScrollView.post(() -> logScrollView.fullScroll(ScrollView.FOCUS_DOWN));
    }
    
    /**
     * IMPROVED: Copy all logs with proper formatting
     */
    public void copyAllLogs(View view) {
        ClipboardManager clipboard = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
        StringBuilder allLogs = new StringBuilder();
        
        allLogs.append("=== Axiom Loader Logs ===\n");
        allLogs.append("Exported: ").append(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date())).append("\n");
        allLogs.append("Total entries: ").append(filteredLogEntries.size()).append("\n");
        allLogs.append("========================\n\n");
        
        for (String log : filteredLogEntries) {
            allLogs.append(cleanLogEntry(log)).append("\n");
        }
        
        ClipData clip = ClipData.newPlainText("Axiom Logs", allLogs.toString());
        clipboard.setPrimaryClip(clip);
        
        Toast.makeText(this, "Logs copied to clipboard (" + filteredLogEntries.size() + " entries)", 
                Toast.LENGTH_SHORT).show();
    }
    
    /**
     * Share current filtered logs
     */
    public void shareCurrentLogs(View view) {
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        
        StringBuilder logsText = new StringBuilder();
        logsText.append("Axiom Loader Logs\n");
        logsText.append("==================\n\n");
        
        for (String log : filteredLogEntries) {
            logsText.append(cleanLogEntry(log)).append("\n");
        }
        
        shareIntent.putExtra(Intent.EXTRA_TEXT, logsText.toString());
        shareIntent.putExtra(Intent.EXTRA_SUBJECT, "Axiom Loader Logs - " + 
                new SimpleDateFormat("yyyy-MM-dd", Locale.US).format(new Date()));
        
        startActivity(Intent.createChooser(shareIntent, "Share Logs"));
    }
    
    /**
     * Bookmark current view state
     */
    public void bookmarkCurrentView(View view) {
        SharedPreferences.Editor editor = prefs.edit();
        editor.putString("bookmark_search", currentSearchQuery);
        editor.putString("bookmark_level", currentLogLevel);
        editor.putString("bookmark_thread", currentThreadFilter);
        editor.putString("bookmark_logger", currentLoggerFilter);
        editor.putLong("bookmark_time", System.currentTimeMillis());
        editor.apply();
        
        Toast.makeText(this, "Current view bookmarked", Toast.LENGTH_SHORT).show();
    }
    
    private void loadSettings() {
        autoScroll = prefs.getBoolean("auto_scroll", true);
        realTimeLogging = prefs.getBoolean("real_time_logging", true);
        
        cbAutoScroll.setChecked(autoScroll);
        cbRealTimeLogging.setChecked(realTimeLogging);
        switchColorCoding.setChecked(prefs.getBoolean("color_coding", true));
        switchDarkTheme.setChecked(prefs.getBoolean("dark_theme", false));
    }
    
    private void saveSettings() {
        SharedPreferences.Editor editor = prefs.edit();
        editor.putBoolean("auto_scroll", autoScroll);
        editor.putBoolean("real_time_logging", realTimeLogging);
        editor.putBoolean("color_coding", switchColorCoding.isChecked());
        editor.putBoolean("dark_theme", switchDarkTheme.isChecked());
        editor.apply();
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
        refreshLogDisplay();
        if (realTimeLogging && !isPaused) {
            startRealTimeUpdates();
        }
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        stopRealTimeUpdates();
        saveSettings();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopRealTimeUpdates();
        if (executorService != null) {
            executorService.shutdown();
        }
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/ui/LoggingActivity.java ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogBootReceiver.java ====
package com.axiomloader.utils;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;

public class LogBootReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction()) ||
            Intent.ACTION_MY_PACKAGE_REPLACED.equals(intent.getAction()) ||
            Intent.ACTION_PACKAGE_REPLACED.equals(intent.getAction())) {
            
            // Initialize LogUtils on boot
            LogUtils logUtils = LogUtils.getInstance(context);
            logUtils.info("LogBootReceiver", "System boot completed - LogUtils initialized");
            
            // Start cleanup service
            Intent serviceIntent = new Intent(context, LogCleanupService.class);
            context.startService(serviceIntent);
        }
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogBootReceiver.java ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogCleanupService.java ====
package com.axiomloader.utils;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import androidx.annotation.Nullable;

public class LogCleanupService extends Service {
    
    private LogUtils logUtils;
    
    @Override
    public void onCreate() {
        super.onCreate();
        logUtils = LogUtils.getInstance(this);
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        if (logUtils != null) {
            logUtils.info("LogCleanupService", "Log cleanup service started");
            
            // Perform cleanup in background thread
            new Thread(() -> {
                try {
                    // This will trigger the cleanup process
                    logUtils.info("LogCleanupService", "Performing background log maintenance");
                    
                    // Stop service after cleanup
                    stopSelf();
                } catch (Exception e) {
                    logUtils.error("LogCleanupService", "Error during cleanup", e);
                    stopSelf();
                }
            }).start();
        }
        
        return START_NOT_STICKY;
    }
    
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        if (logUtils != null) {
            logUtils.info("LogCleanupService", "Log cleanup service destroyed");
        }
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogCleanupService.java ====

==== BEGIN FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogUtils.java ====
package com.axiomloader.utils;

import android.content.Context;
import android.content.SharedPreferences;
import android.os.Build;
import android.os.Environment;
import android.os.Handler;
import android.os.Looper;
import android.os.SystemClock;
import android.util.Log;

import org.apache.logging.log4j.Level;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.core.LoggerContext;
import org.apache.logging.log4j.core.appender.FileAppender;
import org.apache.logging.log4j.core.appender.RollingFileAppender;
import org.apache.logging.log4j.core.config.Configuration;
import org.apache.logging.log4j.core.config.ConfigurationFactory;
import org.apache.logging.log4j.core.config.builder.api.AppenderComponentBuilder;
import org.apache.logging.log4j.core.config.builder.api.ComponentBuilder;
import org.apache.logging.log4j.core.config.builder.api.ConfigurationBuilder;
import org.apache.logging.log4j.core.config.builder.api.LayoutComponentBuilder;
import org.apache.logging.log4j.core.config.builder.api.LoggerComponentBuilder;
import org.apache.logging.log4j.core.config.builder.api.RootLoggerComponentBuilder;
import org.apache.logging.log4j.core.config.builder.impl.BuiltConfiguration;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.ObjectNode;

import org.apache.commons.io.FileUtils;
import org.apache.commons.lang3.StringUtils;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.Date;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Locale;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;
import java.util.regex.Pattern;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

/**
 * Advanced Log4j2-powered logging utility for Axiom Loader
 * 
 * Features:
 * 1. Multi-level logging (TRACE, DEBUG, INFO, WARN, ERROR, FATAL)
 * 2. Rolling file appenders with size-based rotation
 * 3. Automatic log compression and archival
 * 4. Real-time log filtering and searching
 * 5. Performance metrics and statistics
 * 6. Thread-safe operations with concurrent logging
 * 7. Memory management with configurable buffer sizes
 * 8. Log export in multiple formats (TXT, JSON, XML, CSV, HTML)
 * 9. Automatic log cleanup and retention policies
 * 10. Custom log formatters with timestamps and thread info
 */
public class LogUtils {
    
    // Constants
    private static final String TAG = "LogUtils";
    private static final String LOG_DIR_NAME = "axiom_logs";
    private static final String LOG_FILE_PREFIX = "axiom";
    private static final String LOG_FILE_EXTENSION = ".log";
    private static final String ARCHIVE_EXTENSION = ".zip";
    private static final String PREFS_NAME = "log_utils_prefs";
    
    // Configuration Constants
    private static final long DEFAULT_MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
    private static final int DEFAULT_MAX_BACKUP_FILES = 10;
    private static final int DEFAULT_LOG_RETENTION_DAYS = 30;
    private static final int DEFAULT_BUFFER_SIZE = 8192;
    private static final long CLEANUP_INTERVAL_MS = 24 * 60 * 60 * 1000; // 24 hours
    private static final long STATS_UPDATE_INTERVAL_MS = 60 * 1000; // 1 minute
    
    // Singleton instance
    private static volatile LogUtils instance;
    
    // Core components
    private Context context;
    private Logger mainLogger;
    private LoggerContext loggerContext;
    private Configuration loggerConfig;
    private SharedPreferences preferences;
    
    // File management
    private File logDirectory;
    private File currentLogFile;
    private AtomicLong currentFileSize = new AtomicLong(0);
    
    // Threading
    private ExecutorService logExecutor;
    private ScheduledExecutorService scheduledExecutor;
    private Handler mainHandler;
    
    // Configuration
    private long maxFileSize = DEFAULT_MAX_FILE_SIZE;
    private int maxBackupFiles = DEFAULT_MAX_BACKUP_FILES;
    private int logRetentionDays = DEFAULT_LOG_RETENTION_DAYS;
    private int bufferSize = DEFAULT_BUFFER_SIZE;
    private Level logLevel = Level.DEBUG;
    private boolean compressionEnabled = true;
    private boolean realTimeLoggingEnabled = true;
    
    // Statistics and monitoring
    private final Map<String, AtomicLong> logCounters = new ConcurrentHashMap<>();
    private final Map<String, Long> logTimestamps = new ConcurrentHashMap<>();
    private final Set<String> threadNames = Collections.synchronizedSet(new HashSet<>());
    private final Set<String> loggerNames = Collections.synchronizedSet(new HashSet<>());
    private final List<String> recentLogs = Collections.synchronizedList(new ArrayList<>());
    
    // Performance tracking
    private long initializationTime;
    private AtomicLong totalLogCount = new AtomicLong(0);
    private AtomicLong totalBytesLogged = new AtomicLong(0);
    private AtomicLong sessionStartTime = new AtomicLong(System.currentTimeMillis());
    
    // Advanced features
    private final Map<String, String> logMarkers = new ConcurrentHashMap<>();
    private final List<LogAnalysisPattern> analysisPatterns = new ArrayList<>();
    private final ObjectMapper jsonMapper = new ObjectMapper();
    private boolean integrityCheckEnabled = true;
    private String sessionId;
    
    /**
     * Inner class for log analysis patterns
     */
    public static class LogAnalysisPattern {
        public String name;
        public Pattern pattern;
        public String category;
        public int priority;
        
        public LogAnalysisPattern(String name, String regex, String category, int priority) {
            this.name = name;
            this.pattern = Pattern.compile(regex, Pattern.CASE_INSENSITIVE);
            this.category = category;
            this.priority = priority;
        }
    }
    
    /**
     * Private constructor for singleton pattern
     */
    private LogUtils(Context context) {
        this.context = context.getApplicationContext();
        this.sessionId = generateSessionId();
        this.initializationTime = System.currentTimeMillis();
        
        initializeComponents();
        initializeLogging();
        initializeStatistics();
        scheduleMaintenanceTasks();
        initializeAnalysisPatterns();
        
        info(TAG, "LogUtils initialized successfully with session ID: " + sessionId);
    }
    
    /**
     * Get singleton instance
     */
    public static LogUtils getInstance(Context context) {
        if (instance == null) {
            synchronized (LogUtils.class) {
                if (instance == null) {
                    instance = new LogUtils(context);
                }
            }
        }
        return instance;
    }
    
    /**
     * Initialize core components
     */
    private void initializeComponents() {
        preferences = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        logExecutor = Executors.newSingleThreadExecutor(r -> {
            Thread t = new Thread(r, "LogUtils-Worker");
            t.setDaemon(true);
            return t;
        });
        scheduledExecutor = Executors.newScheduledThreadPool(2, r -> {
            Thread t = new Thread(r, "LogUtils-Scheduled");
            t.setDaemon(true);
            return t;
        });
        mainHandler = new Handler(Looper.getMainLooper());
        
        loadConfiguration();
        setupLogDirectory();
    }
    
    /**
     * Load configuration from SharedPreferences
     */
    private void loadConfiguration() {
        maxFileSize = preferences.getLong("max_file_size", DEFAULT_MAX_FILE_SIZE);
        maxBackupFiles = preferences.getInt("max_backup_files", DEFAULT_MAX_BACKUP_FILES);
        logRetentionDays = preferences.getInt("log_retention_days", DEFAULT_LOG_RETENTION_DAYS);
        bufferSize = preferences.getInt("buffer_size", DEFAULT_BUFFER_SIZE);
        compressionEnabled = preferences.getBoolean("compression_enabled", true);
        realTimeLoggingEnabled = preferences.getBoolean("real_time_logging", true);
        integrityCheckEnabled = preferences.getBoolean("integrity_check", true);
        
        String levelStr = preferences.getString("log_level", "DEBUG");
        logLevel = Level.valueOf(levelStr);
    }
    
    /**
     * Setup log directory and current log file
     */
    private void setupLogDirectory() {
        try {
            File externalFilesDir = context.getExternalFilesDir(null);
            if (externalFilesDir != null) {
                logDirectory = new File(externalFilesDir, LOG_DIR_NAME);
            } else {
                logDirectory = new File(context.getFilesDir(), LOG_DIR_NAME);
            }
            
            if (!logDirectory.exists()) {
                boolean created = logDirectory.mkdirs();
                if (!created) {
                    Log.e(TAG, "Failed to create log directory: " + logDirectory.getAbsolutePath());
                }
            }
            
            String logFileName = LOG_FILE_PREFIX + "_" + 
                new SimpleDateFormat("yyyyMMdd", Locale.US).format(new Date()) + 
                LOG_FILE_EXTENSION;
            currentLogFile = new File(logDirectory, logFileName);
            
            if (currentLogFile.exists()) {
                currentFileSize.set(currentLogFile.length());
            }
            
        } catch (Exception e) {
            Log.e(TAG, "Error setting up log directory", e);
        }
    }
    
    /**
     * Initialize Log4j2 logging system
     */
    private void initializeLogging() {
        try {
            // Create configuration builder
            ConfigurationBuilder<BuiltConfiguration> builder = 
                ConfigurationFactory.newConfigurationBuilder();
            
            // Configure root logger level
            builder.setStatusLevel(Level.ERROR);
            builder.setConfigurationName("AxiomLogConfig");
            
            // Create pattern layout with cleaner formatting
            LayoutComponentBuilder layoutBuilder = builder.newLayout("PatternLayout")
                    .addAttribute("pattern", 
                        "%d{HH:mm:ss.SSS} [%level] %logger{20} - %msg%n");
            
            // Create rolling file appender
            AppenderComponentBuilder appenderBuilder = builder.newAppender("RollingFile", "RollingFile")
                    .addAttribute("fileName", currentLogFile.getAbsolutePath())
                    .addAttribute("filePattern", 
                        logDirectory.getAbsolutePath() + "/" + LOG_FILE_PREFIX + "-%i" + LOG_FILE_EXTENSION)
                    .add(layoutBuilder)
                    .addComponent(builder.newComponent("Policies")
                        .addComponent(builder.newComponent("SizeBasedTriggeringPolicy")
                            .addAttribute("size", maxFileSize + "B"))
                        .addComponent(builder.newComponent("TimeBasedTriggeringPolicy")
                            .addAttribute("interval", "1")
                            .addAttribute("modulate", true)))
                    .addComponent(builder.newComponent("DefaultRolloverStrategy")
                        .addAttribute("max", String.valueOf(maxBackupFiles)));
            
            builder.add(appenderBuilder);
            
            // Create console appender for debugging with cleaner format
            AppenderComponentBuilder consoleBuilder = builder.newAppender("Console", "CONSOLE")
                    .addAttribute("target", "SYSTEM_OUT")
                    .add(layoutBuilder);
            
            builder.add(consoleBuilder);
            
            // Create root logger
            RootLoggerComponentBuilder rootLogger = builder.newRootLogger(logLevel)
                    .add(builder.newAppenderRef("RollingFile"))
                    .add(builder.newAppenderRef("Console"));
            
            builder.add(rootLogger);
            
            // Build and apply configuration
            loggerConfig = builder.build();
            loggerContext = (LoggerContext) LogManager.getContext(false);
            loggerContext.setConfiguration(loggerConfig);
            loggerContext.updateLoggers();
            
            // Get main logger
            mainLogger = LogManager.getLogger("AxiomLogger");
            
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize Log4j2", e);
            // Fallback to Android Log
            fallbackToAndroidLog();
        }
    }
    
    /**
     * Initialize statistics tracking
     */
    private void initializeStatistics() {
        // Initialize counters for each log level
        for (Level level : new Level[]{Level.TRACE, Level.DEBUG, Level.INFO, Level.WARN, Level.ERROR, Level.FATAL}) {
            logCounters.put(level.name(), new AtomicLong(0));
        }
        
        // Additional statistics
        logCounters.put("TOTAL", new AtomicLong(0));
        logCounters.put("ERRORS_TODAY", new AtomicLong(0));
        logCounters.put("CRASHES", new AtomicLong(0));
        logCounters.put("PERFORMANCE_ISSUES", new AtomicLong(0));
    }
    
    /**
     * Schedule maintenance tasks
     */
    private void scheduleMaintenanceTasks() {
        // Schedule log cleanup task
        scheduledExecutor.scheduleAtFixedRate(this::performLogCleanup, 
                CLEANUP_INTERVAL_MS, CLEANUP_INTERVAL_MS, TimeUnit.MILLISECONDS);
        
        // Schedule statistics update task
        scheduledExecutor.scheduleAtFixedRate(this::updateStatistics,
                STATS_UPDATE_INTERVAL_MS, STATS_UPDATE_INTERVAL_MS, TimeUnit.MILLISECONDS);
        
        // Schedule compression task
        if (compressionEnabled) {
            scheduledExecutor.scheduleAtFixedRate(this::compressOldLogs,
                    6 * 60 * 60 * 1000, 6 * 60 * 60 * 1000, TimeUnit.MILLISECONDS); // Every 6 hours
        }
        
        // Schedule integrity check task
        if (integrityCheckEnabled) {
            scheduledExecutor.scheduleAtFixedRate(this::performIntegrityCheck,
                    2 * 60 * 60 * 1000, 2 * 60 * 60 * 1000, TimeUnit.MILLISECONDS); // Every 2 hours
        }
    }
    
    /**
     * Initialize analysis patterns for log categorization
     */
    private void initializeAnalysisPatterns() {
        // Error patterns
        analysisPatterns.add(new LogAnalysisPattern("OutOfMemoryError", 
                "OutOfMemoryError|OOM|memory.*leak", "MEMORY", 1));
        analysisPatterns.add(new LogAnalysisPattern("NetworkError", 
                "IOException|ConnectException|timeout|network", "NETWORK", 2));
        analysisPatterns.add(new LogAnalysisPattern("SecurityViolation", 
                "SecurityException|permission.*denied|unauthorized", "SECURITY", 1));
        analysisPatterns.add(new LogAnalysisPattern("DatabaseError", 
                "SQLException|database.*error|cursor.*closed", "DATABASE", 3));
        analysisPatterns.add(new LogAnalysisPattern("UIError", 
                "ViewRootImpl|IllegalStateException.*fragment", "UI", 4));
        
        // Performance patterns
        analysisPatterns.add(new LogAnalysisPattern("SlowOperation", 
                "slow|performance|took.*[0-9]+.*ms", "PERFORMANCE", 3));
        analysisPatterns.add(new LogAnalysisPattern("ANR", 
                "ANR|Application Not Responding|main.*thread.*blocked", "PERFORMANCE", 1));
        
        // System patterns
        analysisPatterns.add(new LogAnalysisPattern("LowStorage", 
                "storage.*low|disk.*full|insufficient.*space", "SYSTEM", 2));
        analysisPatterns.add(new LogAnalysisPattern("BatteryOptimization", 
                "battery.*optimization|doze.*mode|standby", "POWER", 3));
        
        // Custom application patterns
        analysisPatterns.add(new LogAnalysisPattern("AxiomLoad", 
                "loading|loader|axiom.*start", "APPLICATION", 4));
        analysisPatterns.add(new LogAnalysisPattern("UserAction", 
                "user.*click|button.*press|gesture", "USER_INTERACTION", 5));
    }
    
    /**
     * Generate unique session ID
     */
    private String generateSessionId() {
        return String.format(Locale.US, "AX_%d_%s", 
                System.currentTimeMillis(),
                Build.MODEL.replaceAll("\\s+", "").substring(0, 
                Math.min(4, Build.MODEL.length())));
    }
    
    /**
     * Fallback to Android logging system
     */
    private void fallbackToAndroidLog() {
        Log.w(TAG, "Falling back to Android Log system");
        // Set flag to use Android Log instead of Log4j2
        // Implementation would use Android Log.d, Log.i, etc.
    }
    
    // ========== PUBLIC LOGGING METHODS ==========
    
    /**
     * Log TRACE level message
     */
    public void trace(String logger, String message) {
        logMessage(Level.TRACE, logger, message, null);
    }
    
    public void trace(String logger, String message, Throwable throwable) {
        logMessage(Level.TRACE, logger, message, throwable);
    }
    
    /**
     * Log DEBUG level message
     */
    public void debug(String logger, String message) {
        logMessage(Level.DEBUG, logger, message, null);
    }
    
    public void debug(String logger, String message, Throwable throwable) {
        logMessage(Level.DEBUG, logger, message, throwable);
    }
    
    /**
     * Log INFO level message
     */
    public void info(String logger, String message) {
        logMessage(Level.INFO, logger, message, null);
    }
    
    public void info(String logger, String message, Throwable throwable) {
        logMessage(Level.INFO, logger, message, throwable);
    }
    
    /**
     * Log WARN level message
     */
    public void warn(String logger, String message) {
        logMessage(Level.WARN, logger, message, null);
    }
    
    public void warn(String logger, String message, Throwable throwable) {
        logMessage(Level.WARN, logger, message, throwable);
    }
    
    /**
     * Log ERROR level message
     */
    public void error(String logger, String message) {
        logMessage(Level.ERROR, logger, message, null);
    }
    
    public void error(String logger, String message, Throwable throwable) {
        logMessage(Level.ERROR, logger, message, throwable);
    }
    
    /**
     * Log FATAL level message
     */
    public void fatal(String logger, String message) {
        logMessage(Level.FATAL, logger, message, null);
    }
    
    public void fatal(String logger, String message, Throwable throwable) {
        logMessage(Level.FATAL, logger, message, throwable);
    }
    
   /**
     * Core logging method - IMPROVED FOR CLEANER OUTPUT
     */
    private void logMessage(Level level, String loggerName, String message, Throwable throwable) {
        if (!realTimeLoggingEnabled || level.intLevel() > logLevel.intLevel()) {
            return;
        }
        
        logExecutor.execute(() -> {
            try {
                // Track statistics
                updateLogStatistics(level, loggerName, message);
                
                // Get logger instance
                Logger logger = LogManager.getLogger(loggerName);
                
                // Add context information - CLEANED UP
                String threadName = Thread.currentThread().getName();
                threadNames.add(threadName);
                loggerNames.add(loggerName);
                
                // Enhanced message with minimal context - SIMPLIFIED
                String enhancedMessage = enhanceMessageClean(message, threadName);
                
                // Log with Log4j2
                if (throwable != null) {
                    logger.log(level, enhancedMessage, throwable);
                } else {
                    logger.log(level, enhancedMessage);
                }
                
                // Store recent logs for quick access - FORMATTED
                storeRecentLogClean(level, loggerName, enhancedMessage, throwable);
                
                // Perform pattern analysis
                analyzeLogPattern(level, enhancedMessage);
                
                // Check for rotation if needed
                checkLogRotation();
                
            } catch (Exception e) {
                Log.e(TAG, "Error in logMessage", e);
            }
        });
    }

    /**
     * IMPROVED: Enhance message with minimal context for cleaner display
     */
    private String enhanceMessageClean(String message, String threadName) {
        StringBuilder enhanced = new StringBuilder();
        
        // Only add thread name if it's not the main logging thread and is relevant
        if (!threadName.equals("LogUtils-Worker") && 
            !threadName.equals("LogUtils-Scheduled") &&
            !threadName.equals("main")) {
            enhanced.append("[").append(threadName).append("] ");
        }
        
        enhanced.append(message);
        
        // Only add memory info for actual critical errors
        if (message.toLowerCase().contains("outofmemoryerror") || 
            message.toLowerCase().contains("fatal") ||
            (message.toLowerCase().contains("error") && message.toLowerCase().contains("memory"))) {
            Runtime runtime = Runtime.getRuntime();
            long usedMemory = runtime.totalMemory() - runtime.freeMemory();
            enhanced.append(" [Memory: ").append(usedMemory / 1024 / 1024).append("MB]");
        }
        
        return enhanced.toString();
    }
    
    /**
     * Update log statistics
     */
    private void updateLogStatistics(Level level, String loggerName, String message) {
        logCounters.get(level.name()).incrementAndGet();
        logCounters.get("TOTAL").incrementAndGet();
        totalLogCount.incrementAndGet();
        totalBytesLogged.addAndGet(message.length());
        
        logTimestamps.put(level.name() + "_LAST", System.currentTimeMillis());
        
        // Track specific error types
        if (level.intLevel() <= Level.ERROR.intLevel()) {
            if (message.toLowerCase().contains("crash")) {
                logCounters.get("CRASHES").incrementAndGet();
            }
            if (message.toLowerCase().contains("slow") || message.toLowerCase().contains("performance")) {
                logCounters.get("PERFORMANCE_ISSUES").incrementAndGet();
            }
        }
    }
    
    /**
     * IMPROVED: Store recent logs with clean formatting
     */
    private void storeRecentLogClean(Level level, String loggerName, String message, Throwable throwable) {
        String timestamp = new SimpleDateFormat("HH:mm:ss.SSS", Locale.US).format(new Date());
        
        // Create clean log entry format
        String logEntry = String.format("[%s] [%s] %s - %s", 
                timestamp, level.name(), loggerName.replace("com.axiomloader.", ""), message);
        
        if (throwable != null) {
            logEntry += " | " + throwable.getClass().getSimpleName() + ": " + throwable.getMessage();
        }
        
        synchronized (recentLogs) {
            recentLogs.add(logEntry);
            // Keep only last 500 entries in memory for better performance
            if (recentLogs.size() > 500) {
                recentLogs.remove(0);
            }
        }
    }
    
    /**
     * Analyze log patterns for categorization
     */
    private void analyzeLogPattern(Level level, String message) {
        for (LogAnalysisPattern pattern : analysisPatterns) {
            if (pattern.pattern.matcher(message).find()) {
                String key = "PATTERN_" + pattern.name;
                logCounters.computeIfAbsent(key, k -> new AtomicLong(0)).incrementAndGet();
                
                // Store pattern match for reporting
                logMarkers.put(key + "_LAST", 
                        new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date()));
                break; // Only match first pattern to avoid double counting
            }
        }
    }
    
    /**
     * Check if log rotation is needed
     */
    private void checkLogRotation() {
        if (currentLogFile.length() > maxFileSize) {
            rotateLogFile();
        }
    }
    
    /**
     * Rotate log file
     */
    private void rotateLogFile() {
        try {
            String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.US).format(new Date());
            String rotatedFileName = LOG_FILE_PREFIX + "_" + timestamp + LOG_FILE_EXTENSION;
            File rotatedFile = new File(logDirectory, rotatedFileName);
            
            if (currentLogFile.renameTo(rotatedFile)) {
                currentLogFile = new File(logDirectory, 
                        LOG_FILE_PREFIX + "_current" + LOG_FILE_EXTENSION);
                currentFileSize.set(0);
                
                info(TAG, "Log file rotated to: " + rotatedFileName);
                
                // Compress old log file if enabled
                if (compressionEnabled) {
                    compressLogFile(rotatedFile);
                }
            }
        } catch (Exception e) {
            error(TAG, "Failed to rotate log file", e);
        }
    }
    
    /**
     * Update statistics periodically
     */
    private void updateStatistics() {
        try {
            // Update file sizes
            long totalSize = 0;
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX) && 
                    (file.getName().endsWith(LOG_FILE_EXTENSION) || 
                     file.getName().endsWith(ARCHIVE_EXTENSION)));
            
            if (logFiles != null) {
                for (File file : logFiles) {
                    totalSize += file.length();
                }
            }
            
            // Store in preferences for persistence
            preferences.edit()
                    .putLong("total_log_size", totalSize)
                    .putLong("total_log_count", totalLogCount.get())
                    .putLong("session_start_time", sessionStartTime.get())
                    .apply();
                    
        } catch (Exception e) {
            error(TAG, "Error updating statistics", e);
        }
    }
    
    /**
     * Perform log cleanup based on retention policy
     */
    private void performLogCleanup() {
        try {
            long cutoffTime = System.currentTimeMillis() - (logRetentionDays * 24 * 60 * 60 * 1000L);
            
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX) && 
                    file.lastModified() < cutoffTime);
            
            if (logFiles != null) {
                int deletedCount = 0;
                long deletedSize = 0;
                
                for (File file : logFiles) {
                    deletedSize += file.length();
                    if (file.delete()) {
                        deletedCount++;
                    }
                }
                
                if (deletedCount > 0) {
                    info(TAG, String.format("Cleaned up %d old log files, freed %d bytes", 
                            deletedCount, deletedSize));
                }
            }
        } catch (Exception e) {
            error(TAG, "Error during log cleanup", e);
        }
    }
    
    /**
     * Compress old log files
     */
    private void compressOldLogs() {
        try {
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX) && 
                    file.getName().endsWith(LOG_FILE_EXTENSION) &&
                    !file.equals(currentLogFile) &&
                    (System.currentTimeMillis() - file.lastModified()) > (24 * 60 * 60 * 1000)); // 1 day old
            
            if (logFiles != null) {
                for (File logFile : logFiles) {
                    compressLogFile(logFile);
                }
            }
        } catch (Exception e) {
            error(TAG, "Error compressing old logs", e);
        }
    }
    
    /**
     * Compress individual log file
     */
    private void compressLogFile(File logFile) {
        try {
            String compressedName = logFile.getName().replace(LOG_FILE_EXTENSION, ARCHIVE_EXTENSION);
            File compressedFile = new File(logDirectory, compressedName);
            
            try (ZipOutputStream zos = new ZipOutputStream(
                    Files.newOutputStream(compressedFile.toPath()))) {
                
                ZipEntry entry = new ZipEntry(logFile.getName());
                zos.putNextEntry(entry);
                
                byte[] buffer = Files.readAllBytes(logFile.toPath());
                zos.write(buffer);
                zos.closeEntry();
                
                // Delete original file after successful compression
                if (logFile.delete()) {
                    info(TAG, "Compressed and deleted: " + logFile.getName());
                }
            }
        } catch (Exception e) {
            error(TAG, "Failed to compress log file: " + logFile.getName(), e);
        }
    }
    
    /**
     * Perform integrity check on log files
     */
    private void performIntegrityCheck() {
        try {
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX));
            
            if (logFiles != null) {
                for (File file : logFiles) {
                    if (!file.canRead() || file.length() < 0) {
                        warn(TAG, "Integrity issue detected in file: " + file.getName());
                    }
                }
            }
        } catch (Exception e) {
            error(TAG, "Error during integrity check", e);
        }
    }
    
    /**
     * Custom HTML escape method
     */
    private String escapeHtml(String text) {
        if (text == null) return "";
        return text.replace("&", "&amp;")
                   .replace("<", "&lt;")
                   .replace(">", "&gt;")
                   .replace("\"", "&quot;")
                   .replace("'", "&#x27;");
    }
    
    // ========== PUBLIC UTILITY METHODS ==========
    
    /**
     * IMPROVED: Get all logs formatted for display with cleaner formatting
     */
    public List<String> getAllLogsFormatted() {
        List<String> allLogs = new ArrayList<>();
        
        try {
            // First add recent in-memory logs (already formatted)
            synchronized (recentLogs) {
                allLogs.addAll(new ArrayList<>(recentLogs));
            }
            
            // Then add file logs if needed
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX) && 
                    file.getName().endsWith(LOG_FILE_EXTENSION));
            
            if (logFiles != null && allLogs.size() < 100) {
                // Sort by modification time (newest first)
                Arrays.sort(logFiles, (f1, f2) -> 
                        Long.compare(f2.lastModified(), f1.lastModified()));
                
                for (File file : logFiles) {
                    try {
                        List<String> fileLines = Files.readAllLines(file.toPath(), StandardCharsets.UTF_8);
                        // Format file lines to match our clean format
                        for (String line : fileLines) {
                            if (!line.trim().isEmpty()) {
                                String cleanLine = cleanFileLogEntry(line);
                                if (!cleanLine.isEmpty()) {
                                    allLogs.add(cleanLine);
                                }
                            }
                        }
                        
                        // Limit total logs for performance
                        if (allLogs.size() > 200) break;
                    } catch (Exception e) {
                        error(TAG, "Error reading log file: " + file.getName(), e);
                    }
                }
            }
            
        } catch (Exception e) {
            error(TAG, "Error reading log files", e);
            // Return recent logs as fallback
            synchronized (recentLogs) {
                allLogs.clear();
                allLogs.addAll(new ArrayList<>(recentLogs));
            }
        }
        
        return allLogs;
    }
    
    /**
     * Clean log entries from file for consistent formatting
     */
    private String cleanFileLogEntry(String rawLine) {
        try {
            // Skip empty or very short lines
            if (rawLine == null || rawLine.trim().length() < 10) {
                return "";
            }
            
            // Extract timestamp, level, logger, and message
            String cleaned = rawLine.trim();
            
            // Replace complex logger names
            cleaned = cleaned.replaceAll("com\\.axiomloader\\.", "");
            
            // Remove session context if present
            cleaned = cleaned.replaceAll("\\[AX_[^\\]]+\\]\\s*", "");
            cleaned = cleaned.replaceAll("\\[LogUtils-Worker\\]\\s*", "");
            
            // Ensure consistent spacing
            cleaned = cleaned.replaceAll("\\s+", " ");
            
            return cleaned;
        } catch (Exception e) {
            return rawLine; // Return original if cleaning fails
        }
    }
    
    /**
     * Get all thread names that have logged
     */
    public List<String> getAllThreadNames() {
        return new ArrayList<>(threadNames);
    }
    
    /**
     * Get all logger names that have been used
     */
    public List<String> getAllLoggerNames() {
        List<String> cleanNames = new ArrayList<>();
        for (String name : loggerNames) {
            cleanNames.add(name.replace("com.axiomloader.", ""));
        }
        return cleanNames;
    }
    
    /**
     * Clear all logs
     */
    public void clearAllLogs() {
        logExecutor.execute(() -> {
            try {
                File[] logFiles = logDirectory.listFiles(file -> 
                        file.getName().startsWith(LOG_FILE_PREFIX));
                
                if (logFiles != null) {
                    for (File file : logFiles) {
                        file.delete();
                    }
                }
                
                // Clear in-memory logs
                synchronized (recentLogs) {
                    recentLogs.clear();
                }
                
                // Reset statistics
                for (AtomicLong counter : logCounters.values()) {
                    counter.set(0);
                }
                
                // Recreate current log file
                setupLogDirectory();
                
                info(TAG, "All logs cleared successfully");
                
            } catch (Exception e) {
                error(TAG, "Error clearing logs", e);
            }
        });
    }
    
    /**
     * Export logs in specified format
     */
    public File exportLogs(String format, List<String> logEntries) throws IOException {
        String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.US).format(new Date());
        String fileName = "exported_logs_" + timestamp + "." + format.toLowerCase();
        File exportFile = new File(logDirectory, fileName);
        
        switch (format.toUpperCase()) {
            case "TXT":
                exportAsTxt(exportFile, logEntries);
                break;
            case "JSON":
                exportAsJson(exportFile, logEntries);
                break;
            case "XML":
                exportAsXml(exportFile, logEntries);
                break;
            case "CSV":
                exportAsCsv(exportFile, logEntries);
                break;
            case "HTML":
                exportAsHtml(exportFile, logEntries);
                break;
            default:
                throw new IllegalArgumentException("Unsupported export format: " + format);
        }
        
        info(TAG, "Logs exported to: " + exportFile.getAbsolutePath());
        return exportFile;
    }
    
    /**
     * Export logs as plain text
     */
    private void exportAsTxt(File file, List<String> logEntries) throws IOException {
        try (PrintWriter writer = new PrintWriter(new FileWriter(file, StandardCharsets.UTF_8))) {
            writer.println("=== Axiom Loader Log Export ===");
            writer.println("Export Date: " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date()));
            writer.println("Session ID: " + sessionId);
            writer.println("Total Entries: " + logEntries.size());
            writer.println("=================================");
            writer.println();
            
            for (String entry : logEntries) {
                writer.println(entry);
            }
        }
    }
    
    /**
     * Export logs as JSON
     */
    private void exportAsJson(File file, List<String> logEntries) throws IOException {
        ObjectNode root = jsonMapper.createObjectNode();
        root.put("exportDate", new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date()));
        root.put("sessionId", sessionId);
        root.put("totalEntries", logEntries.size());
        
        // Add log entries
        ArrayNode logs = jsonMapper.createArrayNode();
        for (String entry : logEntries) {
            logs.add(entry);
        }
        root.set("logs", logs);
        
        jsonMapper.writerWithDefaultPrettyPrinter().writeValue(file, root);
    }
    
    /**
     * Export logs as XML
     */
    private void exportAsXml(File file, List<String> logEntries) throws IOException {
        try (PrintWriter writer = new PrintWriter(new FileWriter(file, StandardCharsets.UTF_8))) {
            writer.println("<?xml version=\"1.0\" encoding=\"UTF-8\"?>");
            writer.println("<logExport>");
            writer.println("  <metadata>");
            writer.println("    <exportDate>" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date()) + "</exportDate>");
            writer.println("    <sessionId>" + sessionId + "</sessionId>");
            writer.println("    <totalEntries>" + logEntries.size() + "</totalEntries>");
            writer.println("  </metadata>");
            
            writer.println("  <logs>");
            for (String entry : logEntries) {
                writer.println("    <log><![CDATA[" + entry + "]]></log>");
            }
            writer.println("  </logs>");
            writer.println("</logExport>");
        }
    }
    
    /**
     * Export logs as CSV
     */
    private void exportAsCsv(File file, List<String> logEntries) throws IOException {
        try (PrintWriter writer = new PrintWriter(new FileWriter(file, StandardCharsets.UTF_8))) {
            writer.println("Timestamp,Level,Logger,Message");
            
            for (String entry : logEntries) {
                String[] parts = parseLogEntryToParts(entry);
                writer.printf("\"%s\",\"%s\",\"%s\",\"%s\"%n",
                        parts[0], parts[1], parts[2], parts[3]);
            }
        }
    }
    
    /**
     * Export logs as HTML
     */
    private void exportAsHtml(File file, List<String> logEntries) throws IOException {
        try (PrintWriter writer = new PrintWriter(new FileWriter(file, StandardCharsets.UTF_8))) {
            writer.println("<!DOCTYPE html>");
            writer.println("<html><head>");
            writer.println("<title>Axiom Loader Log Export</title>");
            writer.println("<style>");
            writer.println("body { font-family: monospace; background: #1a1a1a; color: #ffffff; padding: 20px; }");
            writer.println(".log-entry { margin: 5px 0; padding: 5px; }");
            writer.println(".error { color: #ff6b6b; }");
            writer.println(".warn { color: #ffd93d; }");
            writer.println(".info { color: #6bcf7f; }");
            writer.println(".debug { color: #4d96ff; }");
            writer.println(".trace { color: #9b59b6; }");
            writer.println("</style></head><body>");
            
            writer.println("<h1>Axiom Loader Log Export</h1>");
            writer.println("<p>Export Date: " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US).format(new Date()) + "</p>");
            
            for (String entry : logEntries) {
                String cssClass = getCssClassForLogLevel(entry);
                writer.println("<div class=\"log-entry " + cssClass + "\">" + 
                        escapeHtml(entry) + "</div>");
            }
            
            writer.println("</body></html>");
        }
    }
    
    /**
     * Parse log entry into parts for CSV export
     */
    private String[] parseLogEntryToParts(String entry) {
        String[] parts = {"", "INFO", "unknown", entry};
        
        try {
            // Parse our clean format: [timestamp] [level] logger - message
            if (entry.startsWith("[") && entry.contains("] [")) {
                int firstClose = entry.indexOf("]");
                parts[0] = entry.substring(1, firstClose);
                
                String remaining = entry.substring(firstClose + 3);
                if (remaining.startsWith("[") && remaining.contains("]")) {
                    int secondClose = remaining.indexOf("]");
                    parts[1] = remaining.substring(1, secondClose);
                    
                    remaining = remaining.substring(secondClose + 2);
                    if (remaining.contains(" - ")) {
                        String[] loggerAndMessage = remaining.split(" - ", 2);
                        parts[2] = loggerAndMessage[0].trim();
                        parts[3] = loggerAndMessage.length > 1 ? loggerAndMessage[1] : "";
                    }
                }
            }
        } catch (Exception e) {
            // Keep defaults if parsing fails
        }
        
        return parts;
    }
    
    /**
     * Get CSS class for log level
     */
    private String getCssClassForLogLevel(String entry) {
        if (entry.contains("[ERROR]") || entry.contains("[FATAL]")) return "error";
        if (entry.contains("[WARN]")) return "warn";
        if (entry.contains("[INFO]")) return "info";
        if (entry.contains("[DEBUG]")) return "debug";
        if (entry.contains("[TRACE]")) return "trace";
        return "";
    }
    
    /**
     * Generate test logs for demonstration
     */
    public void generateTestLogs(int count) {
        logExecutor.execute(() -> {
            String[] testMessages = {
                "Application started successfully",
                "User interface loaded",
                "Database connection established", 
                "Processing user request",
                "Network request completed",
                "Cache updated successfully",
                "Background task finished",
                "Configuration loaded",
                "Memory usage is normal",
                "Performance check passed"
            };
            
            Level[] levels = {Level.INFO, Level.DEBUG, Level.WARN, Level.ERROR};
            String[] loggers = {"MainActivity", "DatabaseManager", "NetworkService", "CacheManager"};
            
            for (int i = 0; i < count; i++) {
                Level level = levels[i % levels.length];
                String logger = loggers[i % loggers.length];
                String message = testMessages[i % testMessages.length] + " #" + (i + 1);
                
                logMessage(level, logger, message, null);
                
                try {
                    Thread.sleep(50); // Small delay to spread timestamps
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
            
            info(TAG, "Generated " + count + " test log entries");
        });
    }
    
    /**
     * Get total log file size
     */
    public long getTotalLogFileSize() {
        long totalSize = 0;
        try {
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX));
            
            if (logFiles != null) {
                for (File file : logFiles) {
                    totalSize += file.length();
                }
            }
        } catch (Exception e) {
            error(TAG, "Error calculating total log size", e);
        }
        return totalSize;
    }
    
    /**
     * Get oldest log timestamp
     */
    public String getOldestLogTimestamp() {
        try {
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX));
            
            if (logFiles != null && logFiles.length > 0) {
                File oldestFile = Collections.min(Arrays.asList(logFiles), 
                        Comparator.comparing(File::lastModified));
                return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US)
                        .format(new Date(oldestFile.lastModified()));
            }
        } catch (Exception e) {
            error(TAG, "Error getting oldest log timestamp", e);
        }
        return "N/A";
    }
    
    /**
     * Get newest log timestamp
     */
    public String getNewestLogTimestamp() {
        try {
            File[] logFiles = logDirectory.listFiles(file -> 
                    file.getName().startsWith(LOG_FILE_PREFIX));
            
            if (logFiles != null && logFiles.length > 0) {
                File newestFile = Collections.max(Arrays.asList(logFiles), 
                        Comparator.comparing(File::lastModified));
                return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US)
                        .format(new Date(newestFile.lastModified()));
            }
        } catch (Exception e) {
            error(TAG, "Error getting newest log timestamp", e);
        }
        return "N/A";
    }
    
    /**
     * Get log statistics
     */
    public Map<String, Long> getLogStatistics() {
        Map<String, Long> stats = new HashMap<>();
        for (Map.Entry<String, AtomicLong> entry : logCounters.entrySet()) {
            stats.put(entry.getKey(), entry.getValue().get());
        }
        stats.put("TOTAL_BYTES", totalBytesLogged.get());
        stats.put("SESSION_DURATION", System.currentTimeMillis() - sessionStartTime.get());
        stats.put("TOTAL_FILE_SIZE", getTotalLogFileSize());
        return stats;
    }
    
    /**
     * Set log level dynamically
     */
    public void setLogLevel(String levelStr) {
        try {
            this.logLevel = Level.valueOf(levelStr.toUpperCase());
            preferences.edit().putString("log_level", levelStr.toUpperCase()).apply();
            
            // Update Log4j2 configuration
            if (loggerContext != null) {
                org.apache.logging.log4j.core.config.Configurator.setRootLevel(logLevel);
                loggerContext.updateLoggers();
            }
            
            info(TAG, "Log level changed to: " + levelStr);
        } catch (IllegalArgumentException e) {
            error(TAG, "Invalid log level: " + levelStr, e);
        }
    }
    
    /**
     * Enable/disable real-time logging
     */
    public void setRealTimeLogging(boolean enabled) {
        this.realTimeLoggingEnabled = enabled;
        preferences.edit().putBoolean("real_time_logging", enabled).apply();
        info(TAG, "Real-time logging " + (enabled ? "enabled" : "disabled"));
    }
    
    /**
     * Enable/disable compression
     */
    public void setCompressionEnabled(boolean enabled) {
        this.compressionEnabled = enabled;
        preferences.edit().putBoolean("compression_enabled", enabled).apply();
        info(TAG, "Log compression " + (enabled ? "enabled" : "disabled"));
    }
    
    /**
     * Shutdown LogUtils and cleanup resources
     */
    public void shutdown() {
        try {
            info(TAG, "Shutting down LogUtils");
            
            if (scheduledExecutor != null) {
                scheduledExecutor.shutdown();
                scheduledExecutor.awaitTermination(5, TimeUnit.SECONDS);
            }
            
            if (logExecutor != null) {
                logExecutor.shutdown();
                logExecutor.awaitTermination(5, TimeUnit.SECONDS);
            }
            
            if (loggerContext != null) {
                loggerContext.stop();
            }
            
            // Save final statistics
            updateStatistics();
            
        } catch (Exception e) {
            Log.e(TAG, "Error during shutdown", e);
        }
    }
}
==== END FILE: Axiom/app/src/main/java/com/axiomloader/utils/LogUtils.java ====

==== BEGIN FILE: Axiom/app/src/main/jni/Android.mk ====
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := tomaslib
LOCAL_SRC_FILES := tomaslib.cpp

include $(BUILD_SHARED_LIBRARY)
==== END FILE: Axiom/app/src/main/jni/Android.mk ====

==== BEGIN FILE: Axiom/app/src/main/jni/Application.mk ====
APP_ABI := all
APP_PLATFORM := android-21
APP_STL := c++_shared
APP_CPPFLAGS += -std=c++17
==== END FILE: Axiom/app/src/main/jni/Application.mk ====

==== BEGIN FILE: Axiom/app/src/main/jni/tomaslib.cpp ====
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
==== END FILE: Axiom/app/src/main/jni/tomaslib.cpp ====

==== BEGIN FILE: Axiom/app/src/main/jni/tomaslib.h ====
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
==== END FILE: Axiom/app/src/main/jni/tomaslib.h ====

==== BEGIN FILE: Axiom/app/src/main/res/drawable/ic_launcher_background.xml ====
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

==== END FILE: Axiom/app/src/main/res/drawable/ic_launcher_background.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/drawable-v24/ic_launcher_foreground.xml ====
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
==== END FILE: Axiom/app/src/main/res/drawable-v24/ic_launcher_foreground.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/layout/activity_logging.xml ====
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/background_dark"
    tools:context=".ui.LoggingActivity">

    <!-- Progress Bar -->
    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="4dp"
        android:visibility="gone"
        style="?android:attr/progressBarStyleHorizontal"
        android:progressTint="@color/axiom_blue" />

    <!-- Main Control Buttons -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="4dp"
        android:gravity="center">

        <Button
            android:id="@+id/btn_clear_logs"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_clear_logs"
            android:textSize="10sp"
            android:background="#dc3545"
            android:textColor="@color/white" />

        <Button
            android:id="@+id/btn_refresh_logs"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_refresh_logs"
            android:textSize="10sp"
            android:background="@color/success_green"
            android:textColor="#ffffff" />

        <Button
            android:id="@+id/btn_pause_logs"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_pause_logs"
            android:textSize="10sp"
            android:background="#ffc107"
            android:textColor="#000000" />

        <Button
            android:id="@+id/btn_export_logs"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_export_logs"
            android:textSize="10sp"
            android:background="#007acc"
            android:textColor="#ffffff" />

    </LinearLayout>

    <!-- Secondary Control Buttons -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="4dp"
        android:gravity="center">

        <Button
            android:id="@+id/btn_generate_test_logs"
            android:layout_width="0dp"
            android:layout_height="36dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_generate_test_logs"
            android:textSize="10sp"
            android:background="#6f42c1"
            android:textColor="#ffffff" />

        <Button
            android:id="@+id/btn_settings"
            android:layout_width="0dp"
            android:layout_height="36dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="@string/btn_settings"
            android:textSize="10sp"
            android:background="#fd7e14"
            android:textColor="#ffffff" />

        <Button
            android:id="@+id/btn_stats"
            android:layout_width="0dp"
            android:layout_height="36dp"
            android:layout_weight="1"
            android:layout_margin="2dp"
            android:text="Statistics"
            android:textSize="10sp"
            android:background="#20c997"
            android:textColor="#ffffff" />

    </LinearLayout>

    <!-- Compact Filters Section -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="@color/background_medium"
        android:padding="8dp"
        android:layout_margin="4dp">

        <!-- Search Box -->
        <EditText
            android:id="@+id/et_search_logs"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:hint="@string/hint_search_logs"
            android:textColor="#ffffff"
            android:textColorHint="#cccccc"
            android:background="#3a3a3a"
            android:padding="8dp"
            android:textSize="13sp"
            android:layout_marginBottom="4dp" />

        <!-- Filter Spinners -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="4dp">

            <Spinner
                android:id="@+id/spinner_log_level"
                android:layout_width="0dp"
                android:layout_height="36dp"
                android:layout_weight="1"
                android:layout_margin="1dp"
                android:background="#3a3a3a" />

            <Spinner
                android:id="@+id/spinner_thread_filter"
                android:layout_width="0dp"
                android:layout_height="36dp"
                android:layout_weight="1"
                android:layout_margin="1dp"
                android:background="#3a3a3a" />

            <Spinner
                android:id="@+id/spinner_logger_filter"
                android:layout_width="0dp"
                android:layout_height="36dp"
                android:layout_weight="1"
                android:layout_margin="1dp"
                android:background="#3a3a3a" />

        </LinearLayout>

        <!-- Filter Options -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <CheckBox
                android:id="@+id/cb_regex_search"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="@string/regex_search"
                android:textColor="@color/white"
                android:textSize="10sp" />

            <CheckBox
                android:id="@+id/cb_case_sensitive"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="@string/case_sensitive"
                android:textColor="#ffffff"
                android:textSize="10sp" />

            <CheckBox
                android:id="@+id/cb_auto_scroll"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="@string/btn_auto_scroll"
                android:textColor="#ffffff"
                android:textSize="10sp" />

            <CheckBox
                android:id="@+id/cb_real_time_logging"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Real-time"
                android:textColor="#ffffff"
                android:textSize="10sp" />

        </LinearLayout>

    </LinearLayout>

    <!-- Collapsible sections in a small scrollable area -->
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="0"
        android:maxHeight="200dp"
        android:nestedScrollingEnabled="false">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <!-- Statistics Container (Initially Hidden) -->
            <LinearLayout
                android:id="@+id/stats_container"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:background="#2a2a2a"
                android:padding="8dp"
                android:layout_margin="4dp"
                android:visibility="gone">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Log Statistics"
                    android:textColor="#ffffff"
                    android:textStyle="bold"
                    android:textSize="14sp"
                    android:layout_marginBottom="4dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <TextView
                        android:id="@+id/tv_total_logs"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_total_logs"
                        android:textColor="#ffffff"
                        android:textSize="11sp"
                        android:padding="2dp" />

                    <TextView
                        android:id="@+id/tv_error_count"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_error_count"
                        android:textColor="#ff6b6b"
                        android:textSize="11sp"
                        android:padding="2dp" />

                    <TextView
                        android:id="@+id/tv_warn_count"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_warn_count"
                        android:textColor="#ffd93d"
                        android:textSize="11sp"
                        android:padding="2dp" />

                </LinearLayout>

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <TextView
                        android:id="@+id/tv_file_size"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_file_size"
                        android:textColor="#ffffff"
                        android:textSize="11sp"
                        android:padding="2dp" />

                    <TextView
                        android:id="@+id/tv_oldest_log"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_oldest_log"
                        android:textColor="#cccccc"
                        android:textSize="11sp"
                        android:padding="2dp" />

                    <TextView
                        android:id="@+id/tv_newest_log"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/stats_newest_log"
                        android:textColor="#cccccc"
                        android:textSize="11sp"
                        android:padding="2dp" />

                </LinearLayout>

            </LinearLayout>

            <!-- Settings Container (Initially Hidden) -->
            <LinearLayout
                android:id="@+id/settings_container"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:background="#2a2a2a"
                android:padding="8dp"
                android:layout_margin="4dp"
                android:visibility="gone">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@string/log_settings_title"
                    android:textColor="#ffffff"
                    android:textStyle="bold"
                    android:textSize="14sp"
                    android:layout_marginBottom="4dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/setting_color_coding"
                        android:textColor="#ffffff"
                        android:textSize="12sp" />

                    <Switch
                        android:id="@+id/switch_color_coding"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginEnd="8dp" />

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="@string/setting_dark_theme"
                        android:textColor="#ffffff"
                        android:textSize="12sp" />

                    <Switch
                        android:id="@+id/switch_dark_theme"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content" />

                </LinearLayout>

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:layout_marginTop="4dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/setting_buffer_size"
                        android:textColor="#ffffff"
                        android:textSize="12sp" />

                    <SeekBar
                        android:id="@+id/seekbar_buffer_size"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:max="100"
                        android:progress="50" />

                </LinearLayout>

            </LinearLayout>

            <!-- Quick Actions Container -->
            <LinearLayout
                android:id="@+id/filters_container"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:background="#2a2a2a"
                android:padding="8dp"
                android:layout_margin="4dp"
                android:visibility="visible">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Quick Actions"
                    android:textColor="#ffffff"
                    android:textStyle="bold"
                    android:textSize="14sp"
                    android:layout_marginBottom="4dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <Button
                        android:layout_width="0dp"
                        android:layout_height="32dp"
                        android:layout_weight="1"
                        android:layout_margin="1dp"
                        android:text="Copy All"
                        android:textSize="10sp"
                        android:background="#17a2b8"
                        android:textColor="#ffffff"
                        android:onClick="copyAllLogs" />

                    <Button
                        android:layout_width="0dp"
                        android:layout_height="32dp"
                        android:layout_weight="1"
                        android:layout_margin="1dp"
                        android:text="Share"
                        android:textSize="10sp"
                        android:background="#6c757d"
                        android:textColor="#ffffff"
                        android:onClick="shareCurrentLogs" />

                    <Button
                        android:layout_width="0dp"
                        android:layout_height="32dp"
                        android:layout_weight="1"
                        android:layout_margin="1dp"
                        android:text="Bookmark"
                        android:textSize="10sp"
                        android:background="#e83e8c"
                        android:textColor="#ffffff"
                        android:onClick="bookmarkCurrentView" />

                </LinearLayout>

            </LinearLayout>

        </LinearLayout>

    </ScrollView>

    <!-- Log Display Header -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Live Log Display"
        android:textColor="#ffffff"
        android:textStyle="bold"
        android:textSize="14sp"
        android:layout_margin="4dp"
        android:gravity="center" />

    <!-- Main Log Display Area - INDEPENDENT SCROLLING -->
    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
        android:id="@+id/swipe_refresh_layout"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_margin="4dp">

        <ScrollView
            android:id="@+id/scroll_log_display"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#1f1f1f"
            android:padding="8dp"
            android:scrollbars="vertical"
            android:fadeScrollbars="false"
            android:scrollbarStyle="outsideOverlay">

            <TextView
                android:id="@+id/tv_log_display"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Initializing advanced logging system...\n\nReal-time logs will appear here automatically\n\nClick 'GENERATE TEST' to create sample logs\n\nUse filters above to search and organize logs\n\nColor coding is enabled by default"
                android:textColor="#ffffff"
                android:textSize="12sp"
                android:fontFamily="monospace"
                android:textIsSelectable="true"
                android:padding="8dp"
                android:lineSpacingExtra="4dp"
                android:lineSpacingMultiplier="1.2" />

        </ScrollView>

    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>

    <!-- Footer Status Bar -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:background="#333333"
        android:padding="6dp"
        android:gravity="center_vertical">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Axiom Advanced Logging System v2.0"
            android:textColor="#cccccc"
            android:textSize="10sp"
            android:gravity="start" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Log4j2 Enhanced"
            android:textColor="#007acc"
            android:textSize="10sp"
            android:textStyle="bold" />

    </LinearLayout>

</LinearLayout>
==== END FILE: Axiom/app/src/main/res/layout/activity_logging.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/layout/activity_main.xml ====
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp"
    android:background="@color/background_dark"
    tools:context=".MainActivity">

    <!-- App Title -->
    <TextView
        android:id="@+id/tv_app_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/main_title"
        android:textSize="32sp"
        android:textStyle="bold"
        android:textColor="@color/text_primary"
        android:layout_marginBottom="48dp"
        android:gravity="center" />

    <!-- Axiom Logo Text -->
    <TextView
        android:id="@+id/tv_axiom_logo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="AXIOM"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="@color/axiom_cyan"
        android:layout_marginBottom="32dp"
        android:gravity="center" />

    <!-- Main Container for Button -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="center"
        android:background="@color/background_medium"
        android:padding="32dp"
        android:layout_marginBottom="32dp">

        <!-- Logs Button -->
        <Button
            android:id="@+id/btn_logs"
            android:layout_width="280dp"
            android:layout_height="60dp"
            android:text="@string/logs_button"
            android:textSize="18sp"
            android:textStyle="bold"
            android:textColor="@color/text_primary"
            android:background="@color/axiom_blue"
            android:layout_marginBottom="16dp"
            android:elevation="4dp"
            android:stateListAnimator="@null" />

        <!-- Button Description -->
        <TextView
            android:id="@+id/tv_logs_desc"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/logs_button_desc"
            android:textSize="14sp"
            android:textColor="@color/text_secondary"
            android:gravity="center"
            android:layout_marginTop="8dp" />

    </LinearLayout>

    <!-- App Version Info -->
    <TextView
        android:id="@+id/tv_version"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="v1.0.0"
        android:textSize="12sp"
        android:textColor="@color/text_disabled"
        android:layout_marginTop="32dp" />

    <!-- Status Bar -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center"
        android:layout_marginTop="24dp"
        android:padding="8dp"
        android:background="@color/background_light">

        <TextView
            android:id="@+id/tv_status"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="System Ready"
            android:textSize="12sp"
            android:textColor="@color/success_green"
            android:layout_weight="1"
            android:gravity="center" />

        <View
            android:id="@+id/status_indicator"
            android:layout_width="8dp"
            android:layout_height="8dp"
            android:background="@color/success_green"
            android:layout_marginStart="8dp" />

    </LinearLayout>

</LinearLayout>
==== END FILE: Axiom/app/src/main/res/layout/activity_main.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/layout/content_main.xml ====
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  android:layout_width="match_parent"
  android:layout_height="match_parent">

  <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello user!"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
==== END FILE: Axiom/app/src/main/res/layout/content_main.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/layout/item_log_entry.xml ====
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="12dp"
    android:background="?android:attr/selectableItemBackground"
    android:clickable="true"
    android:focusable="true">

    <!-- Main Log Entry Row -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <!-- Time Column -->
        <TextView
            android:id="@+id/tv_time"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:text="12:34:56"
            android:textSize="10sp"
            android:textColor="@android:color/darker_gray"
            android:fontFamily="monospace"
            android:gravity="center" />

        <!-- Level Column -->
        <TextView
            android:id="@+id/tv_level"
            android:layout_width="50dp"
            android:layout_height="wrap_content"
            android:text="INFO"
            android:textSize="10sp"
            android:textStyle="bold"
            android:fontFamily="monospace"
            android:gravity="center"
            android:background="#E3F2FD"
            android:padding="2dp"
            android:layout_marginEnd="4dp" />

        <!-- Category Column -->
        <TextView
            android:id="@+id/tv_category"
            android:layout_width="60dp"
            android:layout_height="wrap_content"
            android:text="APP"
            android:textSize="9sp"
            android:textColor="@android:color/darker_gray"
            android:fontFamily="monospace"
            android:gravity="center"
            android:layout_marginEnd="4dp" />

        <!-- Tag Column -->
        <TextView
            android:id="@+id/tv_tag"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:text="MainActivity"
            android:textSize="10sp"
            android:textStyle="bold"
            android:textColor="@android:color/black"
            android:fontFamily="monospace"
            android:maxLines="1"
            android:ellipsize="end"
            android:layout_marginEnd="8dp" />

        <!-- Message Column -->
        <TextView
            android:id="@+id/tv_message"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Application started successfully"
            android:textSize="12sp"
            android:textColor="@android:color/black"
            android:maxLines="2"
            android:ellipsize="end" />

    </LinearLayout>

    <!-- Metadata Row (Initially Hidden) -->
    <TextView
        android:id="@+id/tv_metadata"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:text="metadata: value1=test, value2=123"
        android:textSize="10sp"
        android:textColor="@android:color/darker_gray"
        android:fontFamily="monospace"
        android:background="#F5F5F5"
        android:padding="4dp"
        android:visibility="gone" />

    <!-- Stack Trace Row (Initially Hidden) -->
    <TextView
        android:id="@+id/tv_stack_trace"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:text="java.lang.Exception: Error message\n\tat com.example.Class.method(Class.java:123)"
        android:textSize="9sp"
        android:textColor="@android:color/holo_red_dark"
        android:fontFamily="monospace"
        android:background="#FFEBEE"
        android:padding="4dp"
        android:visibility="gone"
        android:scrollHorizontally="true" />

    <!-- Divider -->
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_marginTop="8dp"
        android:background="#E0E0E0" />

</LinearLayout>
==== END FILE: Axiom/app/src/main/res/layout/item_log_entry.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml ====
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
==== END FILE: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml ====
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
==== END FILE: Axiom/app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/values/colors.xml ====
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Primary Colors -->
    <color name="axiom_blue">#007acc</color>
    <color name="axiom_blue_dark">#005a99</color>
    <color name="axiom_cyan">#00d4ff</color>
    <color name="axiom_cyan_dark">#00a0cc</color>
    
    <!-- Background Colors -->
    <color name="background_dark">#1a1a1a</color>
    <color name="background_medium">#2a2a2a</color>
    <color name="background_light">#3a3a3a</color>
    
    <!-- Text Colors -->
    <color name="text_primary">#ffffff</color>
    <color name="text_secondary">#cccccc</color>
    <color name="text_disabled">#666666</color>
    
    <!-- Status Colors -->
    <color name="success_green">#28a745</color>
    <color name="warning_yellow">#ffc107</color>
    <color name="danger_red">#dc3545</color>
    <color name="info_blue">#17a2b8</color>
    
    <!-- Log Level Colors -->
    <color name="log_error">#ff6b6b</color>
    <color name="log_warn">#ffd93d</color>
    <color name="log_info">#6bcf7f</color>
    <color name="log_debug">#4d96ff</color>
    <color name="log_trace">#9b59b6</color>
    
    <!-- Button Colors -->
    <color name="button_clear">#dc3545</color>
    <color name="button_refresh">#28a745</color>
    <color name="button_pause">#ffc107</color>
    <color name="button_export">#007acc</color>
    <color name="button_generate">#6f42c1</color>
    <color name="button_settings">#fd7e14</color>
    <color name="button_stats">#20c997</color>
    <color name="button_action">#17a2b8</color>
    <color name="button_secondary">#6c757d</color>
    <color name="button_bookmark">#e83e8c</color>
    
    <!-- Standard Material Colors -->
    <color name="black">#000000</color>
    <color name="white">#ffffff</color>
    <color name="transparent">#00000000</color>
</resources>
==== END FILE: Axiom/app/src/main/res/values/colors.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/values/strings.xml ====
<resources>
    <string name="app_name">Axiom Loader</string>
    <string name="hello_world">Hello World!</string>
    <string name="action_settings">Settings</string>
    
    <!-- Main Activity -->
    <string name="main_title">Axiom Loader</string>
    <string name="logs_button">Advanced Logs</string>
    <string name="logs_button_desc">Access comprehensive logging system</string>
    
    <!-- Logging Activity -->
    <string name="logging_title">Advanced Log System</string>
    <string name="log_viewer_title">Log Viewer</string>
    <string name="log_controls_title">Log Controls</string>
    <string name="log_filters_title">Filters &amp; Search</string>
    <string name="log_export_title">Export Options</string>
    <string name="log_settings_title">Log Settings</string>
    
    <!-- Log Levels -->
    <string name="log_level_all">ALL</string>
    <string name="log_level_trace">TRACE</string>
    <string name="log_level_debug">DEBUG</string>
    <string name="log_level_info">INFO</string>
    <string name="log_level_warn">WARN</string>
    <string name="log_level_error">ERROR</string>
    <string name="log_level_fatal">FATAL</string>
    <string name="log_level_off">OFF</string>
    
    <!-- Log Controls -->
    <string name="btn_clear_logs">Clear Logs</string>
    <string name="btn_refresh_logs">Refresh</string>
    <string name="btn_pause_logs">Pause</string>
    <string name="btn_resume_logs">Resume</string>
    <string name="btn_auto_scroll">Auto Scroll</string>
    <string name="btn_search_logs">Search</string>
    <string name="btn_filter_logs">Filter</string>
    <string name="btn_export_logs">Export</string>
    <string name="btn_settings">Settings</string>
    <string name="btn_generate_test_logs">Generate Test Logs</string>
    
    <!-- Export Options -->
    <string name="export_txt">Export as TXT</string>
    <string name="export_json">Export as JSON</string>
    <string name="export_xml">Export as XML</string>
    <string name="export_csv">Export as CSV</string>
    <string name="export_html">Export as HTML</string>
    <string name="share_logs">Share Logs</string>
    
    <!-- Settings -->
    <string name="setting_max_log_files">Max Log Files</string>
    <string name="setting_max_file_size">Max File Size (MB)</string>
    <string name="setting_log_retention">Log Retention (Days)</string>
    <string name="setting_buffer_size">Buffer Size</string>
    <string name="setting_auto_compress">Auto Compress Old Logs</string>
    <string name="setting_real_time_logging">Real-time Logging</string>
    <string name="setting_include_stack_trace">Include Stack Traces</string>
    <string name="setting_log_format">Log Format</string>
    <string name="setting_color_coding">Color Coding</string>
    <string name="setting_dark_theme">Dark Theme</string>
    
    <!-- Search & Filter -->
    <string name="hint_search_logs">Search logs...</string>
    <string name="filter_by_level">Filter by Level</string>
    <string name="filter_by_date">Filter by Date</string>
    <string name="filter_by_thread">Filter by Thread</string>
    <string name="filter_by_logger">Filter by Logger</string>
    <string name="regex_search">Regex Search</string>
    <string name="case_sensitive">Case Sensitive</string>
    
    <!-- Statistics -->
    <string name="stats_total_logs">Total Logs</string>
    <string name="stats_error_count">Error Count</string>
    <string name="stats_warn_count">Warning Count</string>
    <string name="stats_file_size">File Size</string>
    <string name="stats_oldest_log">Oldest Log</string>
    <string name="stats_newest_log">Newest Log</string>
    
    <!-- Messages -->
    <string name="msg_logs_cleared">Logs cleared successfully</string>
    <string name="msg_logs_exported">Logs exported successfully</string>
    <string name="msg_export_failed">Export failed</string>
    <string name="msg_no_logs_found">No logs found</string>
    <string name="msg_search_no_results">No search results</string>
    <string name="msg_copying_to_clipboard">Copying to clipboard...</string>
    <string name="msg_log_file_created">Log file created</string>
    <string name="msg_permission_required">Storage permission required</string>
    
    <!-- Dialogs -->
    <string name="dialog_confirm_clear">Confirm Clear Logs</string>
    <string name="dialog_clear_logs_message">Are you sure you want to clear all logs? This action cannot be undone.</string>
    <string name="dialog_export_options">Export Options</string>
    <string name="dialog_log_details">Log Details</string>
    <string name="dialog_yes">Yes</string>
    <string name="dialog_no">No</string>
    <string name="dialog_ok">OK</string>
    <string name="dialog_cancel">Cancel</string>
    
    <!-- Time Formats -->
    <string name="time_format_full">yyyy-MM-dd HH:mm:ss.SSS</string>
    <string name="time_format_simple">HH:mm:ss</string>
    <string name="time_format_date">yyyy-MM-dd</string>
    
    <!-- File Names -->
    <string name="log_file_prefix">axiom_logs</string>
    <string name="export_file_prefix">exported_logs</string>
</resources>
==== END FILE: Axiom/app/src/main/res/values/strings.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/values/themes.xml ====
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools">
    
    <!-- Base application theme -->
    <style name="Theme.AxiomLoader" parent="Theme.AppCompat.DayNight.DarkActionBar">
        <!-- Primary brand color -->
        <item name="colorPrimary">@color/axiom_blue</item>
        <item name="colorPrimaryVariant">@color/axiom_blue_dark</item>
        <item name="colorOnPrimary">@color/white</item>
        
        <!-- Secondary brand color -->
        <item name="colorSecondary">@color/axiom_cyan</item>
        <item name="colorSecondaryVariant">@color/axiom_cyan_dark</item>
        <item name="colorOnSecondary">@color/black</item>
        
        <!-- Status bar color -->
        <item name="android:statusBarColor" tools:targetApi="l">@color/background_dark</item>
        
        <!-- Background colors -->
        <item name="android:windowBackground">@color/background_dark</item>
        <item name="colorSurface">@color/background_medium</item>
        <item name="colorOnSurface">@color/white</item>
        
        <!-- Action bar -->
        <item name="colorPrimaryDark">@color/background_dark</item>
        <item name="actionBarStyle">@style/ActionBarStyle</item>
        
        <!-- Text colors -->
        <item name="android:textColorPrimary">@color/text_primary</item>
        <item name="android:textColorSecondary">@color/text_secondary</item>
        
        <!-- Button style -->
        <item name="buttonStyle">@style/ButtonStyle</item>
    </style>
    
    <!-- Action Bar Style -->
    <style name="ActionBarStyle" parent="Widget.AppCompat.ActionBar.Solid">
        <item name="background">@color/background_dark</item>
        <item name="titleTextStyle">@style/ActionBarTitleTextStyle</item>
    </style>
    
    <!-- Action Bar Title Text Style -->
    <style name="ActionBarTitleTextStyle" parent="TextAppearance.AppCompat.Widget.ActionBar.Title">
        <item name="android:textColor">@color/white</item>
        <item name="android:textSize">18sp</item>
        <item name="android:textStyle">bold</item>
    </style>
    
    <!-- Button Style -->
    <style name="ButtonStyle" parent="Widget.AppCompat.Button">
        <item name="android:background">@color/axiom_blue</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:textSize">14sp</item>
        <item name="android:textStyle">bold</item>
        <item name="android:padding">12dp</item>
        <item name="android:layout_margin">4dp</item>
    </style>
    
    <!-- EditText Style -->
    <style name="EditTextStyle" parent="Widget.AppCompat.EditText">
        <item name="android:background">@color/background_light</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:textColorHint">@color/text_secondary</item>
        <item name="android:padding">8dp</item>
    </style>
    
    <!-- Spinner Style -->
    <style name="SpinnerStyle" parent="Widget.AppCompat.Spinner">
        <item name="android:background">@color/background_light</item>
        <item name="android:layout_margin">2dp</item>
    </style>

</resources>
==== END FILE: Axiom/app/src/main/res/values/themes.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/values-night/colors.xml ====
<?xml version="1.0" encoding="utf-8"?>
<resources></resources>
==== END FILE: Axiom/app/src/main/res/values-night/colors.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/values-night/themes.xml ====
<resources xmlns:tools="http://schemas.android.com/tools">
  <!-- Base application theme. -->
  <style name="Base.AppTheme" parent="Theme.Material3.DayNight.NoActionBar">
    <!-- Customize your theme here. -->
    <!-- <item name="colorPrimary">@color/my_light_primary</item> -->
  </style>

  <style name="AppTheme" parent="Base.AppTheme" />
</resources>
==== END FILE: Axiom/app/src/main/res/values-night/themes.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/xml/backup_rules.xml ====
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!-- Include log files in backup -->
    <include domain="external" path="axiom_logs/" />
    
    <!-- Include app preferences -->
    <include domain="sharedpref" path="." />
    
    <!-- Exclude temporary files -->
    <exclude domain="cache" path="." />
    <exclude domain="external" path="temp/" />
</full-backup-content>
==== END FILE: Axiom/app/src/main/res/xml/backup_rules.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/xml/data_extraction_rules.xml ====
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup>
        <!-- Include log files in cloud backup -->
        <include domain="external" path="axiom_logs/" />
        
        <!-- Include preferences -->
        <include domain="sharedpref" path="." />
        
        <!-- Exclude sensitive data -->
        <exclude domain="external" path="sensitive/" />
    </cloud-backup>
    
    <device-transfer>
        <!-- Include essential data for device transfer -->
        <include domain="sharedpref" path="." />
        <include domain="external" path="axiom_logs/" />
    </device-transfer>
</data-extraction-rules>
==== END FILE: Axiom/app/src/main/res/xml/data_extraction_rules.xml ====

==== BEGIN FILE: Axiom/app/src/main/res/xml/file_provider_paths.xml ====
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- External files directory for log files -->
    <external-files-path name="axiom_logs" path="axiom_logs/" />
    
    <!-- External cache directory for temporary files -->
    <external-cache-path name="axiom_cache" path="/" />
    
    <!-- Internal files directory -->
    <files-path name="axiom_internal" path="axiom_logs/" />
    
    <!-- External storage root -->
    <external-path name="external_storage_root" path="." />
    
    <!-- Downloads directory -->
    <external-path name="downloads" path="Download/" />
</paths>
==== END FILE: Axiom/app/src/main/res/xml/file_provider_paths.xml ====

==== BEGIN FILE: Axiom/build.gradle ====
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '8.9.1' apply false
    id 'com.android.library' version '8.9.1' apply false
         
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
==== END FILE: Axiom/build.gradle ====

==== BEGIN FILE: Axiom/gradle/wrapper/gradle-wrapper.properties ====
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-bin.zip
networkTimeout=10000
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
==== END FILE: Axiom/gradle/wrapper/gradle-wrapper.properties ====

==== BEGIN FILE: Axiom/gradle.properties ====
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
==== END FILE: Axiom/gradle.properties ====

==== BEGIN FILE: Axiom/gradlew.bat ====
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

==== END FILE: Axiom/gradlew.bat ====

==== BEGIN FILE: Axiom/settings.gradle ====
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
==== END FILE: Axiom/settings.gradle ====

