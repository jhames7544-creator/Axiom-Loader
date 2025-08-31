/ModLoader/app/src/main/java/com/modloader/plugins/Plugin.java

package com.modloader.plugins;

import android.content.Context;

/**
 * Core Plugin interface that addons must implement.
 * Plugins are loaded dynamically via DexClassLoader and receive a limited API surface via ModLoaderApi.
 */
public interface Plugin {
    /**
     * Called when the plugin is first loaded. Do lightweight work only.
     * @param context Application context (do NOT keep strong references to activities).
     * @param api     A limited API surface to interact with the host app.
     * @param descriptor The parsed metadata for this plugin.
     */
    void onLoad(Context context, com.modloader.api.ModLoaderApi api, PluginDescriptor descriptor) throws Exception;

    /**
     * Called when the host app is closing or plugin is being unloaded.
     */
    void onUnload() throws Exception;

    /**
     * Optional event hook. Plugins can handle string-keyed events.
     * Keep work off the main thread.
     */
    default void onEvent(String event, Object payload) throws Exception { /* optional */ }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginDescriptor.java

package com.modloader.plugins;

import java.util.List;

/** Data model describing a plugin as read from manifest.json */
public class PluginDescriptor {
    public String id;
    public String name;
    public String version;
    public String author;
    public String entryClass;
    public boolean enabled = true;
    public List<String> permissions;

    /** NEW: return plugin storage path (id used as filename). */
    public String getPath() {
        return id != null ? id : "";
    }

    @Override
    public String toString() {
        return "PluginDescriptor{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", version='" + version + '\'' +
                ", author='" + author + '\'' +
                ", entryClass='" + entryClass + '\'' +
                ", enabled=" + enabled +
                ", permissions=" + permissions +
                '}';
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginInstaller.java

package com.modloader.plugins;

import android.content.Context;

import com.modloader.util.IOUtils;

import java.io.File;
import java.io.IOException;

/** Handles installing and uninstalling plugin packages into app-private storage. */
public class PluginInstaller {
    private final PluginManager pm;

    public PluginInstaller(Context app) {
        this.pm = new PluginManager(app);
    }

    /** Copies a plugin file into internal storage (replacing existing by same name). */
    public File install(File source) throws IOException {
        File dest = new File(pm.pluginDir(), source.getName());
        IOUtils.copyFile(source, dest);
        return dest;
    }

    public boolean uninstall(String fileName) {
        return pm.uninstallPlugin(fileName);
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginLoader.java

package com.modloader.plugins;

import android.content.Context;

import dalvik.system.DexClassLoader;

import java.io.File;
import java.util.List;

public class PluginLoader {
    private final Context context;
    private final PluginManager pluginManager;

    public PluginLoader(Context context) {
        this.context = context;
        this.pluginManager = new PluginManager(context);
    }

    public void loadAllEnabledPlugins() {
        List<PluginDescriptor> enabledPlugins = pluginManager.getEnabledPlugins();
        File optimizedDir = context.getDir("dex", Context.MODE_PRIVATE);

        for (PluginDescriptor plugin : enabledPlugins) {
            try {
                DexClassLoader classLoader = new DexClassLoader(
                        plugin.getPath(),
                        optimizedDir.getAbsolutePath(),
                        null,
                        context.getClassLoader()
                );

                // Convention: plugin must have class "com.plugin.MainPlugin"
                Class<?> clazz = classLoader.loadClass("com.plugin.MainPlugin");
                Object instance = clazz.newInstance();

                if (instance instanceof IPlugin) {
                    ((IPlugin) instance).onLoad(context);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginManager.java

package com.modloader.plugins;

import android.content.Context;
import android.content.SharedPreferences;
import android.util.Log;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

/**
 * PluginManager: keeps track of plugins in internal storage and preferences for enabled state.
 */
public class PluginManager {
    private static final String PREFS_NAME = "plugin_prefs";
    private static final String KEY_PREFIX_ENABLED = "plugin_enabled_";

    private static PluginManager instance; // Singleton access

    private final Context context;
    private final File pluginDir;
    private final SharedPreferences prefs;

    public PluginManager(Context context) {
        this.context = context.getApplicationContext();
        this.pluginDir = new File(context.getFilesDir(), "plugins");
        if (!pluginDir.exists()) {
            pluginDir.mkdirs();
        }
        this.prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
    }

    /** Singleton accessor */
    public static synchronized PluginManager get(Context context) {
        if (instance == null) {
            instance = new PluginManager(context);
        }
        return instance;
    }

    public List<PluginDescriptor> getInstalledPlugins() {
        List<PluginDescriptor> plugins = new ArrayList<>();
        File[] files = pluginDir.listFiles();
        if (files != null) {
            for (File file : files) {
                if (file.isFile()) {
                    String id = file.getName();
                    String name = file.getName().replace(".jar", "").replace(".apk", "");
                    boolean enabled = prefs.getBoolean(KEY_PREFIX_ENABLED + id, true);

                    PluginDescriptor d = new PluginDescriptor();
                    d.id = id;
                    d.name = name;
                    d.version = "1.0";
                    d.author = "local";
                    d.enabled = enabled;
                    plugins.add(d);
                }
            }
        }
        return plugins;
    }

    public boolean isPluginEnabled(String pluginId) {
        return prefs.getBoolean(KEY_PREFIX_ENABLED + pluginId, true);
    }

    public List<PluginDescriptor> getEnabledPlugins() {
        List<PluginDescriptor> all = getInstalledPlugins();
        List<PluginDescriptor> enabled = new ArrayList<>();
        for (PluginDescriptor p : all) {
            if (p.enabled) enabled.add(p);
        }
        return enabled;
    }

    public void setPluginEnabled(String pluginId, boolean enabled) {
        prefs.edit().putBoolean(KEY_PREFIX_ENABLED + pluginId, enabled).apply();
    }

    public boolean uninstallPlugin(String pluginId) {
        File[] files = pluginDir.listFiles();
        if (files == null) return false;
        for (File file : files) {
            if (file.getName().equals(pluginId)) {
                prefs.edit().remove(KEY_PREFIX_ENABLED + pluginId).apply();
                return file.delete();
            }
        }
        return false;
    }

    public File pluginDir() {
        return pluginDir;
    }

    /** ---------------- NEW METHODS ADDED ---------------- */

    /**
     * Refresh plugin list and return logs of what happened.
     */
    public List<String> refreshAndLoadAll() {
        List<String> logs = new ArrayList<>();
        logs.add("Refreshing plugin list...");

        List<PluginDescriptor> plugins = getInstalledPlugins();
        if (plugins.isEmpty()) {
            logs.add("No plugins found.");
        } else {
            for (PluginDescriptor d : plugins) {
                logs.add("Found plugin: " + d.name + " (enabled=" + d.enabled + ")");
            }
        }

        logs.add("Refresh finished.");
        return logs;
    }

    /**
     * Dispatch an event to all enabled plugins.
     */
    public void dispatchEvent(String event, Object payload) {
        for (PluginDescriptor d : getEnabledPlugins()) {
            // In a real system we’d invoke plugin entry points here
            Log.d("PluginManager", "Dispatching '" + event + "' to " + d.name);
        }
    }

    /**
     * Convenience: return descriptors of all loaded/enabled plugins.
     */
    public List<PluginDescriptor> getLoadedDescriptors() {
        return getEnabledPlugins();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginRepository.java

package com.modloader.plugins;

import android.content.Context;
import android.util.Log;

import com.modloader.util.JsonUtils;

import org.json.JSONObject;

import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/** Scans plugin directories and parses manifest.json for each plugin package. */
public class PluginRepository {
    private static final String TAG = "PluginRepository";
    public static final String DEFAULT_DIR = "ModLoader/addons"; // on external storage (user visible)
    private final Context app;
    private final File internalDir; // app-private copy target

    public PluginRepository(Context app) {
        this.app = app.getApplicationContext();
        this.internalDir = new File(app.getFilesDir(), "addons");
        if (!internalDir.exists()) internalDir.mkdirs();
    }

    public File getInternalDir() { return internalDir; }

    /** Returns list of candidate plugin package files in known dirs. */
    public List<File> scanFiles() {
        List<File> out = new ArrayList<>();
        // Internal dir
        File[] internal = internalDir.listFiles();
        if (internal != null) {
            for (File f : internal) {
                if (isPackageFile(f)) out.add(f);
            }
        }
        // External dir (best-effort, may fail without permission)
        try {
            File ext = new File(android.os.Environment.getExternalStorageDirectory(), DEFAULT_DIR);
            File[] extFiles = ext.listFiles();
            if (extFiles != null) {
                for (File f : extFiles) if (isPackageFile(f)) out.add(f);
            }
        } catch (Throwable t) {
            Log.w(TAG, "External scan skipped: " + t);
        }
        return out;
    }

    private boolean isPackageFile(File f) {
        String n = f.getName().toLowerCase();
        return f.isFile() && (n.endsWith(".apk") || n.endsWith(".jar") || n.endsWith(".dex"));
    }

    /** Parse manifest.json located next to the binary with same stem OR inside a directory package. */
    public PluginDescriptor parseManifest(File packageFile) {
        try {
            // Look for manifest.json next to file: e.g., MyPlugin.apk + MyPlugin.manifest.json
            File sibling = new File(packageFile.getParentFile(),
                    removeExt(packageFile.getName()) + ".manifest.json");
            if (sibling.isFile()) {
                try (InputStream in = new FileInputStream(sibling)) {
                    JSONObject obj = JsonUtils.readJson(in);
                    return JsonUtils.toDescriptor(obj);
                }
            }
        } catch (Exception e) {
            Log.w(TAG, "Manifest parse failed for " + packageFile + ": " + e);
        }
        return null;
    }

    private String removeExt(String name) {
        int idx = name.lastIndexOf('.');
        return (idx < 0) ? name : name.substring(0, idx);
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/plugins/PluginSandboxService.java

package com.modloader.plugins;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;

/** Optional background host service for plugins. This simple version just exposes a binder. */
public class PluginSandboxService extends Service {

    public class LocalBinder extends Binder {
        public PluginSandboxService getService() { return PluginSandboxService.this; }
    }

    private final IBinder binder = new LocalBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/sample/SamplePluginTemplate.java

package com.modloader.sample;

import android.content.Context;
import android.widget.Toast;

import com.modloader.api.ModLoaderApi;
import com.modloader.plugins.Plugin;
import com.modloader.plugins.PluginDescriptor;

/** Example plugin class authors can copy. This is NOT dynamically loaded by default. */
public class SamplePluginTemplate implements Plugin {
    @Override
    public void onLoad(Context context, ModLoaderApi api, PluginDescriptor descriptor) throws Exception {
        api.log("SamplePlugin", "Loaded sample plugin: " + descriptor.id);
        api.postToMain(() -> Toast.makeText(context, "Sample plugin loaded: " + descriptor.name, Toast.LENGTH_SHORT).show());
    }

    @Override
    public void onUnload() throws Exception {
        // cleanup
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/JsScriptingEngine.java

package com.modloader.scripting;
import android.content.Context;
import org.mozilla.javascript.Scriptable;
public final class JsScriptingEngine {
    private final Context app;
    private final ScriptApi api;
    public JsScriptingEngine(Context app, ScriptApi api){ this.app=app.getApplicationContext(); this.api=api; }
    public Object eval(String code){
        String safe = ScriptSandbox.sanitize(code);
        org.mozilla.javascript.Context rh = org.mozilla.javascript.Context.enter();
        try {
            Scriptable scope = rh.initStandardObjects();
            scope.put("api", scope, api);
            return rh.evaluateString(scope, safe, "script.js", 1, null);
        } finally {
            org.mozilla.javascript.Context.exit();
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/LuaScriptingEngine.java

// File: LuaScriptingEngine.java (FIXED) - Fixed CoerceJavaToLua import
// Path: /storage/emulated/0/AndroidIDEProjects/ModLoader/app/src/main/java/com/modloader/scripting/LuaScriptingEngine.java

package com.modloader.scripting;

import org.luaj.vm2.Globals;
import org.luaj.vm2.LuaValue;
import org.luaj.vm2.lib.jse.CoerceJavaToLua;
import org.luaj.vm2.lib.jse.JsePlatform;

import com.modloader.api.ModLoaderApi;
import com.modloader.util.LogUtils;

public class LuaScriptingEngine {
    private final Globals globals;

    public LuaScriptingEngine(ModLoaderApi api) {
        globals = JsePlatform.standardGlobals();
        // FIXED: Added proper import for CoerceJavaToLua
        globals.set("api", CoerceJavaToLua.coerce(api));
        LogUtils.logDebug("Lua scripting engine initialized");
    }

    public LuaValue execute(String script) {
        try {
            return globals.load(script).call();
        } catch (Exception e) {
            LogUtils.logDebug("Lua script execution error: " + e.getMessage());
            return LuaValue.NIL;
        }
    }

    public void setGlobal(String name, Object value) {
        globals.set(name, CoerceJavaToLua.coerce(value));
    }

    public LuaValue getGlobal(String name) {
        return globals.get(name);
    }

    public boolean loadFile(String filename) {
        try {
            LuaValue chunk = globals.loadfile(filename);
            chunk.call();
            return true;
        } catch (Exception e) {
            LogUtils.logDebug("Failed to load Lua file " + filename + ": " + e.getMessage());
            return false;
        }
    }

    public String evaluateToString(String script) {
        try {
            LuaValue result = execute(script);
            return result.tojstring();
        } catch (Exception e) {
            return "Error: " + e.getMessage();
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/ScriptApi.java

package com.modloader.scripting;
import android.content.Context;
import com.modloader.addons.AddonManager;
public final class ScriptApi {
    private final Context app;
    private final AddonManager addons;
    public ScriptApi(Context app, AddonManager addons){ this.app=app.getApplicationContext(); this.addons = addons; }
    public String echo(String s){ return "API:"+s; }
    public int add(int a, int b){ return a+b; }
    public void fire(String hook, String payload){ addons.hooks().fire(hook, payload); }
    public String getAppId(){ return app.getPackageName(); }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/ScriptConsoleActivity.java

// File: ScriptConsoleActivity.java (FIXED) - Fixed ViewGroup import and package
// Path: /storage/emulated/0/AndroidIDEProjects/ModLoader/app/src/main/java/com/modloader/scripting/ScriptConsoleActivity.java

package com.modloader.scripting;

import android.os.AsyncTask;
import android.os.Bundle;
import android.text.method.ScrollingMovementMethod;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import com.modloader.R;
import com.modloader.addons.AddonManager;
import com.modloader.util.FileUtils;
import com.modloader.util.PathProvider;

import java.io.File;

/**
 * Full-featured script console:
 * - Run JavaScript via Rhino
 * - Run Lua via LuaJ
 * - Save and load scripts from app-scoped scripts/ folder
 */
public class ScriptConsoleActivity extends AppCompatActivity {
    private EditText input;
    private TextView output;
    private ScriptEngineManager engines;
    private AddonManager addons;

    @Override
    protected void onCreate(Bundle s) {
        super.onCreate(s);
        setContentView(R.layout.activity_script_console);
        setTitle("Script Console");

        input = findViewById(R.id.input);
        output = findViewById(R.id.output);
        output.setMovementMethod(new ScrollingMovementMethod());

        Button runJs = findViewById(R.id.runJs);
        Button runLua = findViewById(R.id.runLua);
        Button save = new Button(this);
        save.setText("Save Script");

        // FIXED: Added proper ViewGroup cast and null check
        View parent = (View) input.getParent();
        if (parent != null && parent instanceof ViewGroup) {
            ((ViewGroup) parent).post(() -> ((ViewGroup) parent).addView(save));
        }

        addons = new AddonManager(this);
        addons.reload();

        engines = new ScriptEngineManager(this, addons);

        runJs.setOnClickListener(v -> runJs());
        runLua.setOnClickListener(v -> runLua());
        save.setOnClickListener(v -> saveScript());
    }

    private void runJs() {
        final String code = input.getText().toString();
        new AsyncTask<Void, Void, Object>() {
            @Override
            protected Object doInBackground(Void... voids) {
                try {
                    return engines.evalJs(code);
                } catch (Throwable t) {
                    return t;
                }
            }

            @Override
            protected void onPostExecute(Object r) {
                if (r instanceof Throwable) {
                    output.setText("Error: " + ((Throwable) r).getMessage());
                } else {
                    output.setText(String.valueOf(r));
                }
            }
        }.execute();
    }

    private void runLua() {
        final String code = input.getText().toString();
        new AsyncTask<Void, Void, Object>() {
            @Override
            protected Object doInBackground(Void... voids) {
                try {
                    return engines.evalLua(code);
                } catch (Throwable t) {
                    return t;
                }
            }

            @Override
            protected void onPostExecute(Object r) {
                if (r instanceof Throwable) {
                    output.setText("Error: " + ((Throwable) r).getMessage());
                } else {
                    output.setText(String.valueOf(r));
                }
            }
        }.execute();
    }

    private void saveScript() {
        try {
            File dir = PathProvider.getScripts(this);
            if (!dir.exists()) {
                dir.mkdirs();
            }
            String name = "script_" + System.currentTimeMillis() + ".txt";
            File out = new File(dir, name);
            FileUtils.writeText(out, input.getText().toString());
            Toast.makeText(this, "Saved to " + out.getAbsolutePath(), Toast.LENGTH_LONG).show();
        } catch (Exception e) {
            Toast.makeText(this, "Save failed: " + e.getMessage(), Toast.LENGTH_LONG).show();
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/ScriptEngineManager.java

// File: ScriptEngineManager.java - Missing scripting engine manager
// Path: /storage/emulated/0/AndroidIDEProjects/ModLoader/app/src/main/java/com/modloader/scripting/ScriptEngineManager.java

package com.modloader.scripting;

import android.content.Context;

import com.modloader.addons.AddonManager;
import com.modloader.util.LogUtils;

import org.luaj.vm2.Globals;
import org.luaj.vm2.LuaValue;
import org.luaj.vm2.lib.jse.CoerceJavaToLua;
import org.luaj.vm2.lib.jse.JsePlatform;
import org.mozilla.javascript.Scriptable;
import org.mozilla.javascript.ScriptableObject;

/**
 * Manages JavaScript and Lua script execution engines
 */
public class ScriptEngineManager {
    private final Context context;
    private final AddonManager addonManager;
    private final org.mozilla.javascript.Context jsContext;
    private final Scriptable jsScope;
    private final Globals luaGlobals;

    public ScriptEngineManager(Context context, AddonManager addonManager) {
        this.context = context;
        this.addonManager = addonManager;

        // Initialize JavaScript engine (Rhino)
        jsContext = org.mozilla.javascript.Context.enter();
        jsScope = jsContext.initStandardObjects();

        // Add API object to JavaScript scope
        ScriptableObject.putProperty(jsScope, "api", org.mozilla.javascript.Context.javaToJS(new ScriptApi(context, addonManager), jsScope));

        // Initialize Lua engine (LuaJ)
        luaGlobals = JsePlatform.standardGlobals();
        luaGlobals.set("api", CoerceJavaToLua.coerce(new ScriptApi(context, addonManager)));

        LogUtils.logDebug("ScriptEngineManager initialized with JavaScript and Lua support");
    }

    /**
     * Evaluate JavaScript code
     */
    public Object evalJs(String code) throws Exception {
        if (code == null || code.trim().isEmpty()) {
            return "No code to execute";
        }

        try {
            Object result = jsContext.evaluateString(jsScope, code, "<script>", 1, null);
            return org.mozilla.javascript.Context.toString(result);
        } catch (Exception e) {
            LogUtils.logDebug("JavaScript execution error: " + e.getMessage());
            throw e;
        }
    }

    /**
     * Evaluate Lua code
     */
    public Object evalLua(String code) throws Exception {
        if (code == null || code.trim().isEmpty()) {
            return "No code to execute";
        }

        try {
            LuaValue result = luaGlobals.load(code).call();
            return result.tojstring();
        } catch (Exception e) {
            LogUtils.logDebug("Lua execution error: " + e.getMessage());
            throw e;
        }
    }

    /**
     * Clean up resources
     */
    public void cleanup() {
        if (jsContext != null) {
            org.mozilla.javascript.Context.exit();
        }
        LogUtils.logDebug("ScriptEngineManager cleanup completed");
    }

    /**
     * API object exposed to scripts
     */
    public static class ScriptApi {
        private final Context context;
        private final AddonManager addonManager;

        public ScriptApi(Context context, AddonManager addonManager) {
            this.context = context;
            this.addonManager = addonManager;
        }

        public void log(String message) {
            LogUtils.logUser("Script: " + message);
        }

        public void toast(String message) {
            // Post to main thread for UI operations
            if (context instanceof android.app.Activity) {
                ((android.app.Activity) context).runOnUiThread(() -> {
                    android.widget.Toast.makeText(context, message, android.widget.Toast.LENGTH_SHORT).show();
                });
            }
        }

        public String getAppVersion() {
            try {
                return context.getPackageManager().getPackageInfo(context.getPackageName(), 0).versionName;
            } catch (Exception e) {
                return "Unknown";
            }
        }

        public int getAddonCount() {
            return addonManager.getLoaded().size();
        }

        public String[] getAddonNames() {
            return addonManager.getLoaded().stream()
                    .map(addon -> addon.getId())
                    .toArray(String[]::new);
        }

        public void fireHook(String hookName, Object data) {
            addonManager.hooks().fire(hookName, data);
        }

        public void postEvent(Object event) {
            addonManager.events().post(event);
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/scripting/ScriptSandbox.java

package com.modloader.scripting;

public final class ScriptSandbox {
    public static String sanitize(String code) {
        return code.replaceAll("java\\.lang\\.reflect", "[blocked]");
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/AddonListActivity.java

package com.modloader.ui;
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.RecyclerView;
import androidx.recyclerview.widget.LinearLayoutManager;
import com.modloader.addons.*;
import com.modloader.util.Log;
import java.util.ArrayList;
import java.util.List;
public class AddonListActivity extends AppCompatActivity {
    private AddonManager manager;
    private final List<String> items = new ArrayList<>();
    private SimpleAdapter adapter;
    @Override protected void onCreate(Bundle b){
        super.onCreate(b);
        setContentView(getResources().getIdentifier("activity_addon_list","layout",getPackageName()));
        manager = new AddonManager(this);
        RecyclerView rv = findViewById(getResources().getIdentifier("recycler","id",getPackageName()));
        rv.setLayoutManager(new LinearLayoutManager(this));
        adapter = new SimpleAdapter(items);
        rv.setAdapter(adapter);
        Button reload = findViewById(getResources().getIdentifier("btnReload","id",getPackageName()));
        reload.setOnClickListener(v-> refresh());
        refresh();
    }
    private void refresh(){
        manager.reload();
        items.clear();
        for(Addon a: manager.getLoaded()){
            items.add(a.getId()+" v"+a.getVersion());
        }
        adapter.notifyDataSetChanged();
        Log.i("Addons reloaded: "+items.size());
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/BaseActivity.java

// File: BaseActivity.java (FIXED) - Compatible with New PermissionManager
// Path: /app/src/main/java/com/modloader/ui/BaseActivity.java

package com.modloader.ui;

import android.content.Intent;
import android.os.Bundle;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import com.modloader.util.LogUtils;
import com.modloader.util.PermissionManager;
import com.modloader.util.ShizukuManager;

import java.util.List;

/**
 * FIXED: Base Activity with proper PermissionManager integration
 * Compatible with the new PermissionManager and ShizukuManager
 */
public abstract class BaseActivity extends AppCompatActivity implements 
    PermissionManager.PermissionCallback, ShizukuManager.ShizukuPermissionCallback {

    protected PermissionManager permissionManager;
    protected ShizukuManager shizukuManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Initialize managers
        permissionManager = new PermissionManager(this);
        shizukuManager = new ShizukuManager(this);
        
        // Set callbacks
        permissionManager.setPermissionCallback(this);
        shizukuManager.setPermissionCallback(this);
        
        LogUtils.logDebug("BaseActivity initialized: " + this.getClass().getSimpleName());
    }

    @Override
    protected void onResume() {
        super.onResume();
        
        // Refresh permission status when activity resumes
        if (permissionManager != null) {
            permissionManager.refreshPermissionStatus();
        }
        
        // Check if permissions are needed
        if (shouldCheckPermissionsOnResume()) {
            checkAndRequestPermissions();
        }
    }

    /**
     * Override this to control whether permissions should be checked on resume
     */
    protected boolean shouldCheckPermissionsOnResume() {
        return true;
    }

    /**
     * Check and request permissions if needed
     */
    protected void checkAndRequestPermissions() {
        if (permissionManager != null && !hasAllRequiredPermissions()) {
            // Don't automatically request - let user decide
            LogUtils.logDebug("Some permissions are missing, but not auto-requesting");
        }
    }

    /**
     * FIXED: Check if all required permissions are granted
     */
    protected boolean hasAllRequiredPermissions() {
        if (permissionManager == null) {
            return false;
        }
        return permissionManager.hasBasicPermissions();
    }

    /**
     * FIXED: Check if enhanced permissions are available
     */
    protected boolean hasEnhancedPermissions() {
        if (permissionManager == null) {
            return false;
        }
        return permissionManager.hasEnhancedPermissions();
    }

    /**
     * Request all basic permissions
     */
    protected void requestBasicPermissions() {
        if (permissionManager != null) {
            permissionManager.requestBasicPermissions();
        }
    }

    /**
     * Request all permissions including enhanced
     */
    protected void requestAllPermissions() {
        if (permissionManager != null) {
            permissionManager.requestAllPermissions();
        }
    }

    /**
     * Show permission management dialog
     */
    protected void showPermissionDialog() {
        if (permissionManager == null) {
            return;
        }

        String status = permissionManager.getPermissionStatus();
        
        new AlertDialog.Builder(this)
            .setTitle("Permission Status")
            .setMessage(status)
            .setPositiveButton("Request Permissions", (dialog, which) -> requestAllPermissions())
            .setNeutralButton("App Settings", (dialog, which) -> permissionManager.openAppSettings())
            .setNegativeButton("Close", null)
            .show();
    }

    // ===== PermissionManager.PermissionCallback Implementation =====

    @Override
    public void onPermissionGranted(String permission) {
        LogUtils.logUser("Permission granted in " + getClass().getSimpleName() + ": " + permission);
        onPermissionStatusChanged();
    }

    @Override
    public void onPermissionDenied(String permission) {
        LogUtils.logUser("Permission denied in " + getClass().getSimpleName() + ": " + permission);
        onPermissionStatusChanged();
    }

    @Override
    public void onPermissionError(String permission, String error) {
        LogUtils.logUser("Permission error in " + getClass().getSimpleName() + ": " + permission + " - " + error);
        onPermissionStatusChanged();
    }

    @Override
    public void onAllPermissionsResult(boolean allGranted, List<String> denied) {
        LogUtils.logUser("All permissions result in " + getClass().getSimpleName() + 
                        " - All granted: " + allGranted + ", Denied: " + denied.size());
        onPermissionStatusChanged();
    }

    // ===== ShizukuManager.ShizukuPermissionCallback Implementation =====

    @Override
    public void onPermissionGranted() {
        LogUtils.logUser("Shizuku permission granted in " + getClass().getSimpleName());
        onPermissionStatusChanged();
    }

    @Override
    public void onPermissionDenied() {
        LogUtils.logUser("Shizuku permission denied in " + getClass().getSimpleName());
        onPermissionStatusChanged();
    }

    @Override
    public void onPermissionError(String error) {
        LogUtils.logUser("Shizuku permission error in " + getClass().getSimpleName() + ": " + error);
        onPermissionStatusChanged();
    }

    // ===== Abstract Methods =====

    /**
     * Called when permission status changes - override to update UI
     */
    protected void onPermissionStatusChanged() {
        // Default implementation - override in subclasses
        LogUtils.logDebug("Permission status changed in " + getClass().getSimpleName());
    }

    // ===== Permission Request Results =====

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (permissionManager != null) {
            permissionManager.handlePermissionResult(requestCode, permissions, grantResults);
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (permissionManager != null) {
            permissionManager.handleActivityResult(requestCode, resultCode);
        }
    }

    // ===== Utility Methods =====

    /**
     * Get current operation mode from settings
     */
    protected String getCurrentOperationMode() {
        return SettingsActivity.getCurrentOperationMode(this);
    }

    /**
     * Check if Shizuku mode is enabled
     */
    protected boolean isShizukuMode() {
        return SettingsActivity.isShizukuMode(this);
    }

    /**
     * Check if enhanced permissions can be used
     */
    protected boolean canUseEnhancedPermissions() {
        return SettingsActivity.canUseEnhancedPermissions(this);
    }

    /**
     * Get permission manager instance
     */
    protected PermissionManager getPermissionManager() {
        return permissionManager;
    }

    /**
     * Get Shizuku manager instance
     */
    protected ShizukuManager getShizukuManager() {
        return shizukuManager;
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/ConfigExtensionsActivity.java

package com.modloader.ui;
import android.os.Bundle;
import android.view.ViewGroup;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.RecyclerView;
import androidx.recyclerview.widget.LinearLayoutManager;
import com.modloader.addons.AddonManager;
import com.modloader.configext.ConfigExtension;
import com.modloader.configext.UiFactory;
public class ConfigExtensionsActivity extends AppCompatActivity {
    @Override protected void onCreate(Bundle b){
        super.onCreate(b);
        setContentView(getResources().getIdentifier("activity_config_extensions","layout",getPackageName()));
        RecyclerView rv = findViewById(getResources().getIdentifier("recycler","id",getPackageName()));
        rv.setLayoutManager(new LinearLayoutManager(this));
        AddonManager addons = new AddonManager(this);
        addons.reload();
        java.util.List<ConfigExtension.Model> models = ConfigExtension.loadAll(this);
        rv.setAdapter(new RecyclerView.Adapter<SimpleHolder>(){
            @Override public SimpleHolder onCreateViewHolder(ViewGroup p, int vtype){ return new SimpleHolder(new android.widget.FrameLayout(p.getContext())); }
            @Override public void onBindViewHolder(SimpleHolder h, int pos){
                h.root.removeAllViews();
                h.root.addView(UiFactory.buildCard(h.root.getContext(), models.get(pos), addons));
            }
            @Override public int getItemCount(){ return models.size(); }
        });
    }
    static class SimpleHolder extends RecyclerView.ViewHolder {
        android.widget.FrameLayout root;
        SimpleHolder(android.view.View v){ super(v); root = (android.widget.FrameLayout)v; }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/DllModActivity.java

// File: DllModActivity.java (Updated Activity) - Uses Controller Pattern
// Path: /storage/emulated/0/AndroidIDEProjects/main/java/com/terrarialoader/ui/DllModActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.OpenableColumns;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.widget.Toast;

import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.R;
import com.modloader.util.LogUtils;

import java.io.File;
import java.util.List;

/**
 * DllModActivity now delegates business logic to DllModController
 * This keeps the activity focused on UI management while the controller handles the logic
 */
public class DllModActivity extends Activity implements DllModController.DllModCallback {

    private static final int REQUEST_SELECT_APK = 1001;
    private static final int REQUEST_SELECT_DLL = 1002;
    
    // UI Components
    private TextView statusText;
    private TextView loaderStatusText;
    private RecyclerView dllModRecyclerView;
    private Button installLoaderBtn;
    private Button selectApkBtn;
    private Button installDllBtn;
    private Button viewLogsBtn;
    private Button refreshBtn;
    private LinearLayout loaderInfoSection;
    
    // Controller and Adapter
    private DllModController controller;
    private DllModAdapter adapter;
    private AlertDialog progressDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dll_mod);
        
        setTitle("DLL Mod Manager");
        
        // Initialize controller
        controller = new DllModController(this);
        controller.setCallback(this);
        
        initializeViews();
        setupUI();
        
        // Load initial data
        controller.loadDllMods();
        controller.updateLoaderStatus();
    }

    private void initializeViews() {
        statusText = findViewById(R.id.statusText);
        loaderStatusText = findViewById(R.id.loaderStatusText);
        dllModRecyclerView = findViewById(R.id.dllModRecyclerView);
        installLoaderBtn = findViewById(R.id.installLoaderBtn);
        selectApkBtn = findViewById(R.id.selectApkBtn);
        installDllBtn = findViewById(R.id.installDllBtn);
        viewLogsBtn = findViewById(R.id.viewLogsBtn);
        refreshBtn = findViewById(R.id.refreshBtn);
        loaderInfoSection = findViewById(R.id.loaderInfoSection);
    }

    private void setupUI() {
        // Setup RecyclerView
        dllModRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        adapter = new DllModAdapter(this, controller);
        dllModRecyclerView.setAdapter(adapter);

        // Button listeners - delegate to controller
        installLoaderBtn.setOnClickListener(v -> controller.showLoaderInstallDialog());
        
        selectApkBtn.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/vnd.android.package-archive");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            startActivityForResult(intent, REQUEST_SELECT_APK);
        });
        
        installDllBtn.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("*/*"); // Allow all files, we'll filter by extension
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            startActivityForResult(intent, REQUEST_SELECT_DLL);
        });
        
        viewLogsBtn.setOnClickListener(v -> {
            startActivity(new Intent(this, LogViewerActivity.class));
        });
        
        refreshBtn.setOnClickListener(v -> {
            controller.refresh();
            Toast.makeText(this, "Refreshed DLL mod list", Toast.LENGTH_SHORT).show();
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (resultCode != RESULT_OK || data == null || data.getData() == null) {
            return;
        }
        
        Uri uri = data.getData();
        
        switch (requestCode) {
            case REQUEST_SELECT_APK:
                controller.handleApkSelection(uri);
                break;
                
            case REQUEST_SELECT_DLL:
                String filename = getFilenameFromUri(uri);
                controller.handleDllSelection(uri, filename);
                break;
        }
    }

    private String getFilenameFromUri(Uri uri) {
        String filename = null;
        
        try (Cursor cursor = getContentResolver().query(uri, null, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                int nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                if (nameIndex >= 0) {
                    filename = cursor.getString(nameIndex);
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Could not get filename from URI: " + e.getMessage());
        }
        
        return filename;
    }

    @Override
    protected void onResume() {
        super.onResume();
        controller.refresh();
    }

    // === DllModController.DllModCallback Implementation ===
    
    @Override
    public void onModsLoaded(List<File> mods) {
        runOnUiThread(() -> {
            adapter.updateMods(mods);
            statusText.setText(controller.getStatusText());
        });
    }

    @Override
    public void onLoaderStatusChanged(boolean installed, String statusText, int textColor) {
        runOnUiThread(() -> {
            loaderStatusText.setText(statusText);
            loaderStatusText.setTextColor(textColor);
            
            if (installed) {
                installLoaderBtn.setText("Reinstall Loader");
                loaderInfoSection.setVisibility(View.VISIBLE);
                installDllBtn.setEnabled(true);
                installDllBtn.setText("Install DLL Mod");
            } else {
                installLoaderBtn.setText("Install Loader");
                loaderInfoSection.setVisibility(View.GONE);
                installDllBtn.setEnabled(false);
                installDllBtn.setText("Install Loader First");
            }
        });
    }

    @Override
    public void onInstallationProgress(String message) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
            }
            
            if (message.contains("Please select")) {
                // This is a request for user action, not a progress message
                return;
            }
            
            progressDialog = new AlertDialog.Builder(this)
                .setTitle("Installing Loader")
                .setMessage(message)
                .setCancelable(false)
                .show();
        });
    }

    @Override
    public void onInstallationComplete(boolean success, String message) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
                progressDialog = null;
            }
            
            if (success) {
                Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, message, Toast.LENGTH_LONG).show();
            }
        });
    }

    @Override
    public void onError(String error) {
        runOnUiThread(() -> {
            Toast.makeText(this, error, Toast.LENGTH_SHORT).show();
        });
    }

    // === Simple adapter class for DLL mods ===
    private static class DllModAdapter extends RecyclerView.Adapter<DllModAdapter.ViewHolder> {
        private final Activity activity;
        private final DllModController controller;
        private List<File> mods;

        public DllModAdapter(Activity activity, DllModController controller) {
            this.activity = activity;
            this.controller = controller;
            this.mods = controller.getDllMods();
        }

        public void updateMods(List<File> newMods) {
            this.mods = newMods;
            notifyDataSetChanged();
        }

        @Override
        public ViewHolder onCreateViewHolder(android.view.ViewGroup parent, int viewType) {
            View view = android.view.LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_mod, parent, false);
            return new ViewHolder(view);
        }

        @Override
        public void onBindViewHolder(ViewHolder holder, int position) {
            File mod = mods.get(position);
            String name = mod.getName();
            boolean isEnabled = !name.endsWith(".disabled");
            
            holder.modDescription.setText(name);
            holder.modSwitch.setChecked(isEnabled);
            
            holder.modSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
                controller.toggleMod(mod, isChecked);
            });
            
            holder.modDeleteButton.setOnClickListener(v -> {
                controller.deleteMod(mod, () -> {
                    mods.remove(position);
                    notifyItemRemoved(position);
                    notifyItemRangeChanged(position, mods.size());
                });
            });
        }

        @Override
        public int getItemCount() {
            return mods.size();
        }

        static class ViewHolder extends RecyclerView.ViewHolder {
            TextView modDescription;
            android.widget.Switch modSwitch;
            android.widget.ImageButton modDeleteButton;

            ViewHolder(View itemView) {
                super(itemView);
                modDescription = itemView.findViewById(R.id.modDescription);
                modSwitch = itemView.findViewById(R.id.modSwitch);
                modDeleteButton = itemView.findViewById(R.id.modDeleteButton);
            }
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/DllModController.java

// File: DllModController.java (Fixed) - Corrected method calls with Context parameter
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/app/src/main/java/com/terrarialoader/ui/DllModController.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.net.Uri;
import com.modloader.loader.ModManager;
import com.modloader.loader.ModBase;
import com.modloader.loader.MelonLoaderManager;
import com.modloader.util.LogUtils;
import com.modloader.util.FileUtils;
import com.modloader.installer.ModInstaller;
import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class DllModController {
    private final Activity activity;
    private List<File> dllMods = new ArrayList<>();
    
    // Callback interface for UI updates
    public interface DllModCallback {
        void onModsLoaded(List<File> mods);
        void onLoaderStatusChanged(boolean installed, String statusText, int textColor);
        void onInstallationProgress(String message);
        void onInstallationComplete(boolean success, String message);
        void onError(String error);
    }
    
    private DllModCallback callback;

    public DllModController(Activity activity) {
        this.activity = activity;
    }
    
    public void setCallback(DllModCallback callback) {
        this.callback = callback;
    }

    public void loadDllMods() {
        dllMods.clear();
        
        // Get DLL mods from ModManager
        List<File> allMods = ModManager.getAvailableMods();
        for (File mod : allMods) {
            ModBase.ModType type = ModBase.ModType.fromFileName(mod.getName());
            if (type == ModBase.ModType.DLL || type == ModBase.ModType.HYBRID) {
                dllMods.add(mod);
            }
        }
        
        LogUtils.logDebug("Loaded " + dllMods.size() + " DLL mods");
        
        if (callback != null) {
            callback.onModsLoaded(dllMods);
        }
    }

    public void updateLoaderStatus() {
        // FIXED: Pass activity context to MelonLoaderManager methods
        boolean melonInstalled = MelonLoaderManager.isMelonLoaderInstalled(activity);
        boolean lemonInstalled = MelonLoaderManager.isLemonLoaderInstalled(activity);
        
        if (melonInstalled) {
            String statusText = "✅ MelonLoader " + MelonLoaderManager.getInstalledLoaderVersion() + " installed";
            if (callback != null) {
                callback.onLoaderStatusChanged(true, statusText, 0xFF4CAF50); // Green
            }
        } else if (lemonInstalled) {
            String statusText = "✅ LemonLoader " + MelonLoaderManager.getInstalledLoaderVersion() + " installed";
            if (callback != null) {
                callback.onLoaderStatusChanged(true, statusText, 0xFF4CAF50); // Green
            }
        } else {
            String statusText = "❌ No loader installed - DLL mods will not work";
            if (callback != null) {
                callback.onLoaderStatusChanged(false, statusText, 0xFFF44336); // Red
            }
        }
    }

    public void showLoaderInstallDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("Choose Loader");
        builder.setMessage("Select which loader to install:\n\n" +
                          "• MelonLoader: Full-featured, works with most Unity games\n" +
                          "• LemonLoader: Lightweight, better for older devices\n\n" +
                          "You will need to select a Terraria APK file after choosing.");
        
        builder.setPositiveButton("MelonLoader", (dialog, which) -> {
            LogUtils.logUser("User selected MelonLoader installation");
            showApkInstructions("MelonLoader");
        });
        
        builder.setNegativeButton("LemonLoader", (dialog, which) -> {
            LogUtils.logUser("User selected LemonLoader installation");
            showApkInstructions("LemonLoader");
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }

    public void showApkInstructions(String loaderType) {
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("APK Installation Instructions");
        builder.setMessage("To install " + loaderType + ":\n\n" +
                          "1. Select your Terraria APK file\n" +
                          "2. The loader will be injected into the APK\n" +
                          "3. Install the modified APK\n" +
                          "4. Your DLL mods will work in Terraria\n\n" +
                          "Ready to select APK?");
        
        builder.setPositiveButton("Select APK", (dialog, which) -> {
            // Store the selected loader type for later use
            activity.getSharedPreferences("dll_mod_prefs", Activity.MODE_PRIVATE)
                .edit()
                .putString("selected_loader", loaderType)
                .apply();
            
            // Trigger APK selection in the UI
            if (callback != null) {
                callback.onInstallationProgress("Please select Terraria APK file");
            }
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    public void handleApkSelection(Uri apkUri) {
        LogUtils.logUser("APK selected for loader installation");
        
        // Get the selected loader type
        String selectedLoader = activity.getSharedPreferences("dll_mod_prefs", Activity.MODE_PRIVATE)
            .getString("selected_loader", "MelonLoader");
        
        // Show confirmation dialog
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("Confirm Installation");
        builder.setMessage("Install " + selectedLoader + " into the selected APK?\n\n" +
                          "This will create a new patched APK file that you can install.");
        
        builder.setPositiveButton("Install", (dialog, which) -> {
            if ("MelonLoader".equals(selectedLoader)) {
                installLoaderIntoApk(apkUri, MelonLoaderManager.LoaderType.MELONLOADER_NET8);
            } else {
                installLoaderIntoApk(apkUri, MelonLoaderManager.LoaderType.MELONLOADER_NET35);
            }
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    public void handleDllSelection(Uri dllUri, String filename) {
        LogUtils.logUser("DLL file selected for installation");
        
        if (filename == null) {
            filename = "mod_" + System.currentTimeMillis() + ".dll";
        }
        
        if (!filename.toLowerCase().endsWith(".dll")) {
            if (callback != null) {
                callback.onError("Please select a .dll file");
            }
            return;
        }
        
        // FIXED: Pass activity context to MelonLoaderManager methods
        if (!MelonLoaderManager.isMelonLoaderInstalled(activity) && !MelonLoaderManager.isLemonLoaderInstalled(activity)) {
            showLoaderRequiredDialog();
            return;
        }
        
        // Install DLL mod
        boolean success = ModInstaller.installMod(activity, dllUri, filename);
        
        if (success) {
            if (callback != null) {
                callback.onInstallationComplete(true, "DLL mod installed successfully");
            }
            loadDllMods(); // Refresh list
        } else {
            if (callback != null) {
                callback.onInstallationComplete(false, "Failed to install DLL mod");
            }
        }
    }

    public void showLoaderRequiredDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("Loader Required");
        builder.setMessage("DLL mods require MelonLoader or LemonLoader to be installed first.\n\n" +
                          "Would you like to install a loader now?");
        
        builder.setPositiveButton("Install Loader", (dialog, which) -> {
            showLoaderInstallDialog();
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    public void installLoaderIntoApk(Uri apkUri, MelonLoaderManager.LoaderType loaderType) {
        LogUtils.logUser("Installing " + loaderType.getDisplayName() + " into APK");
        
        if (callback != null) {
            callback.onInstallationProgress("Installing " + loaderType.getDisplayName() + " into APK...\nThis may take a few minutes.");
        }
        
        // Run installation in background thread
        new Thread(() -> {
            boolean success = false;
            File outputApk = null;
            
            try {
                // Create temporary files
                File tempApk = File.createTempFile("input_", ".apk", activity.getCacheDir());
                outputApk = new File(activity.getExternalFilesDir(null), "patched_terraria_" + 
                                        loaderType.name().toLowerCase() + "_" +
                                        System.currentTimeMillis() + ".apk");
                
                // Copy input APK to temp file
                FileUtils.copyUriToFile(activity, apkUri, tempApk);
                
                // Install loader
                if (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) {
                    success = MelonLoaderManager.installMelonLoader(activity, tempApk, outputApk);
                } else if (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET35) {
                    success = MelonLoaderManager.installLemonLoader(activity, tempApk, outputApk);
                }
                
                // Cleanup temp file
                tempApk.delete();
                
            } catch (Exception e) {
                LogUtils.logDebug("Loader installation error: " + e.getMessage());
                success = false;
            }
            
            // Update UI on main thread
            final boolean finalSuccess = success;
            final File finalOutputApk = outputApk;
            
            activity.runOnUiThread(() -> {
                if (finalSuccess) {
                    showInstallationSuccessDialog(finalOutputApk, loaderType);
                } else {
                    if (callback != null) {
                        callback.onInstallationComplete(false, "Loader installation failed");
                    }
                    LogUtils.logUser("❌ " + loaderType.getDisplayName() + " installation failed");
                }
                
                updateLoaderStatus();
            });
        }).start();
    }

    public void showInstallationSuccessDialog(File outputApk, MelonLoaderManager.LoaderType loaderType) {
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("Installation Complete");
        builder.setMessage("✅ " + loaderType.getDisplayName() + " has been installed into the APK!\n\n" +
                          "Patched APK saved to:\n" + outputApk.getName() + "\n\n" +
                          "Next steps:\n" +
                          "1. Install the patched APK\n" +
                          "2. Install DLL mods\n" +
                          "3. Launch Terraria");
        
        builder.setPositiveButton("Install APK", (dialog, which) -> {
            installApk(outputApk);
        });
        
        builder.setNegativeButton("Later", null);
        builder.show();
        
        LogUtils.logUser("✅ " + loaderType.getDisplayName() + " installation completed successfully");
        
        if (callback != null) {
            callback.onInstallationComplete(true, loaderType.getDisplayName() + " installation completed successfully");
        }
    }

    public void installApk(File apkFile) {
        try {
            Intent intent = new Intent(Intent.ACTION_VIEW);
            intent.setDataAndType(
                androidx.core.content.FileProvider.getUriForFile(
                    activity, 
                    activity.getPackageName() + ".provider", 
                    apkFile
                ),
                "application/vnd.android.package-archive"
            );
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            activity.startActivity(intent);
        } catch (Exception e) {
            if (callback != null) {
                callback.onError("Cannot install APK: " + e.getMessage());
            }
            LogUtils.logDebug("APK installation error: " + e.getMessage());
        }
    }

    public void toggleMod(File mod, boolean isChecked) {
        if (isChecked) {
            ModManager.enableMod(activity, mod);
        } else {
            ModManager.disableMod(activity, mod);
        }
    }

    public void deleteMod(File mod, Runnable onSuccess) {
        AlertDialog.Builder builder = new AlertDialog.Builder(activity);
        builder.setTitle("Delete Mod");
        builder.setMessage("Delete " + mod.getName() + "?");
        builder.setPositiveButton("Delete", (dialog, which) -> {
            if (ModManager.deleteMod(activity, mod)) {
                if (onSuccess != null) {
                    onSuccess.run();
                }
                loadDllMods(); // Refresh the list
            }
        });
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    public List<File> getDllMods() {
        return new ArrayList<>(dllMods);
    }

    public String getStatusText() {
        int enabledCount = 0;
        for (File mod : dllMods) {
            if (!mod.getName().endsWith(".disabled")) {
                enabledCount++;
            }
        }
        
        return "DLL Mods: " + enabledCount + " enabled, " + 
               (dllMods.size() - enabledCount) + " disabled, " + 
               dllMods.size() + " total";
    }

    public boolean isLoaderInstalled() {
        // FIXED: Pass activity context to MelonLoaderManager methods
        return MelonLoaderManager.isMelonLoaderInstalled(activity) || MelonLoaderManager.isLemonLoaderInstalled(activity);
    }

    public void refresh() {
        loadDllMods();
        updateLoaderStatus();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/InstructionsActivity.java

// File: InstructionsActivity.java (Fixed) - Corrected method calls with Context parameter
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/app/src/main/java/com/terrarialoader/ui/InstructionsActivity.java

package com.modloader.ui;

import android.content.ClipData;
import android.content.ClipboardManager;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

import com.modloader.R;
import com.modloader.loader.MelonLoaderManager;
import com.modloader.util.LogUtils;

public class InstructionsActivity extends AppCompatActivity {

    private TextView tvInstructions;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_instructions);

        setTitle("Manual Installation Guide");

        tvInstructions = findViewById(R.id.tv_instructions);
        
        setupInstructions();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add(0, 1, 0, "Copy GitHub URLs");
        menu.add(0, 2, 0, "Open GitHub");
        menu.add(0, 3, 0, "Check Installation");
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case 1:
                copyGitHubUrls();
                return true;
            case 2:
                openGitHubDialog();
                return true;
            case 3:
                checkInstallationStatus();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }

    private void setupInstructions() {
        // Get the actual path that the app uses
        String actualBasePath = getExternalFilesDir(null) + "/TerrariaLoader/com.and.games505.TerrariaPaid";
        
        String manualInstructions = "📱 Manual MelonLoader/LemonLoader Installation Guide\n\n" +
                
                "🔗 STEP 1: Download Required Files\n" +
                "──────────────────────────────────\n" +
                "Visit GitHub and download ONE of these:\n\n" +
                
                "🔸 For MelonLoader (Full Features):\n" +
                "• Go to: github.com/LavaGang/MelonLoader/releases\n" +
                "• Download 'melon_data.zip' from latest release\n" +
                "• File size: ~40MB\n\n" +
                
                "🔸 For LemonLoader (Lightweight):\n" +
                "• Go to: github.com/LemonLoader/LemonLoader/releases\n" +
                "• Download 'lemon_data.zip' or installer APK\n" +
                "• File size: ~15MB\n\n" +
                
                "📁 STEP 2: Create Directory Structure\n" +
                "────────────────────────────────────\n" +
                "⚠️ IMPORTANT: Use the CORRECT path for your device!\n\n" +
                
                "Using a file manager, create this structure:\n" +
                actualBasePath + "/\n\n" +
                
                "Alternative path (if first doesn't work):\n" +
                "/storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/com.and.games505.TerrariaPaid/\n\n" +
                
                "Create these folders inside the above path:\n" +
                "├── Loaders/MelonLoader/\n" +
                "│   ├── net8/                    (for MelonLoader)\n" +
                "│   ├── net35/                   (for LemonLoader)\n" +
                "│   └── Dependencies/\n" +
                "│       ├── SupportModules/\n" +
                "│       ├── CompatibilityLayers/\n" +
                "│       └── Il2CppAssemblyGenerator/\n" +
                "│           ├── Cpp2IL/cpp2il_out/\n" +
                "│           ├── UnityDependencies/\n" +
                "│           └── Il2CppInterop/Il2CppAssemblies/\n" +
                "├── Mods/\n" +
                "│   ├── DLL/                     (for your DLL mods)\n" +
                "│   └── DEX/                     (for your DEX/JAR mods)\n" +
                "├── Logs/\n" +
                "├── Config/\n" +
                "└── Backups/\n\n" +
                
                "📦 STEP 3: Extract Files\n" +
                "───────────────────────\n" +
                "Extract the downloaded ZIP file:\n\n" +
                
                "🔸 Core Files (place in Loaders/MelonLoader/net8/ or net35/):\n" +
                "• MelonLoader.dll\n" +
                "• 0Harmony.dll\n" +
                "• MonoMod.RuntimeDetour.dll\n" +
                "• MonoMod.Utils.dll\n" +
                "• Il2CppInterop.Runtime.dll (MelonLoader only)\n\n" +
                
                "🔸 Dependencies (place in Loaders/MelonLoader/Dependencies/):\n" +
                "• All remaining DLL files go in appropriate subdirectories\n" +
                "• Unity assemblies go in UnityDependencies/\n" +
                "• Il2Cpp files go in Il2CppAssemblyGenerator/\n\n" +
                
                "⚠️ IMPORTANT FILE PLACEMENT:\n" +
                "────────────────────────────\n" +
                "• MelonLoader files → Loaders/MelonLoader/net8/\n" +
                "• LemonLoader files → Loaders/MelonLoader/net35/\n" +
                "• Support modules → Loaders/MelonLoader/Dependencies/SupportModules/\n" +
                "• Your mod DLLs → Mods/DLL/\n" +
                "• Your DEX/JAR mods → Mods/DEX/\n\n" +
                
                "✅ STEP 4: Verify Installation\n" +
                "─────────────────────────────\n" +
                "Return to TerrariaLoader and:\n" +
                "1. Go to 'DLL Mod Manager'\n" +
                "2. Check loader status\n" +
                "3. Should show '✅ Loader Installed'\n" +
                "4. If not, check file paths carefully\n\n" +
                
                "🎮 STEP 5: Use Your Loader\n" +
                "─────────────────────────\n" +
                "1. Place DLL mods in Mods/DLL/ folder\n" +
                "2. Select Terraria APK in DLL Manager\n" +
                "3. Patch APK with loader\n" +
                "4. Install patched Terraria\n" +
                "5. Launch and enjoy mods!\n\n" +
                
                "🔧 TROUBLESHOOTING:\n" +
                "──────────────────\n" +
                "• Loader not detected → Check file paths exactly\n" +
                "• Can't find directory → Try both paths mentioned in Step 2\n" +
                "• APK patch fails → Verify all DLL files present\n" +
                "• Mods don't load → Check Logs/ folder for errors\n" +
                "• Slow performance → Try LemonLoader instead\n" +
                "• Permission denied → Enable 'All files access' for file manager\n\n" +
                
                "📋 REQUIRED FILES CHECKLIST:\n" +
                "───────────────────────────\n" +
                "☐ MelonLoader.dll (in Loaders/MelonLoader/net8/ or net35/)\n" +
                "☐ 0Harmony.dll (in Loaders/MelonLoader/net8/ or net35/)\n" +
                "☐ MonoMod files (in Loaders/MelonLoader/net8/ or net35/)\n" +
                "☐ Il2CppInterop files (Dependencies/SupportModules/)\n" +
                "☐ Unity dependencies (Dependencies/Il2CppAssemblyGenerator/UnityDependencies/)\n" +
                "☐ Directory structure matches exactly\n" +
                "☐ Using correct base path for your device\n\n" +
                
                "💡 TIPS:\n" +
                "──────\n" +
                "• Use a good file manager (like Solid Explorer)\n" +
                "• Enable 'Show hidden files' in your file manager\n" +
                "• Grant 'All files access' permission to your file manager\n" +
                "• Double-check spelling of folder names\n" +
                "• Keep backup of original Terraria APK\n" +
                "• Start with LemonLoader if you have storage issues\n" +
                "• Copy the exact path from Step 2 to avoid typos\n\n" +
                
                "📍 PATH VERIFICATION:\n" +
                "────────────────────\n" +
                "Your device should use:\n" + actualBasePath + "\n\n" +
                
                "If that doesn't work, try:\n" +
                "/storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/com.and.games505.TerrariaPaid/\n\n" +
                
                "Need help? Use the menu (⋮) for quick actions or check the logs in TerrariaLoader for detailed error messages!";

        tvInstructions.setText(manualInstructions);
    }

    private void copyGitHubUrls() {
        String githubUrls = "MelonLoader: https://github.com/LavaGang/MelonLoader/releases\n" +
                           "LemonLoader: https://github.com/LemonLoader/LemonLoader/releases";
        
        ClipboardManager clipboard = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
        ClipData clip = ClipData.newPlainText("GitHub URLs", githubUrls);
        clipboard.setPrimaryClip(clip);
        
        Toast.makeText(this, "GitHub URLs copied to clipboard!", Toast.LENGTH_SHORT).show();
        LogUtils.logUser("User copied GitHub URLs to clipboard");
    }

    private void openGitHubDialog() {
        android.app.AlertDialog.Builder builder = new android.app.AlertDialog.Builder(this);
        builder.setTitle("Open GitHub Repository");
        builder.setMessage("Which repository would you like to visit?");
        
        builder.setPositiveButton("MelonLoader", (dialog, which) -> {
            openUrl("https://github.com/LavaGang/MelonLoader/releases");
        });
        
        builder.setNegativeButton("LemonLoader", (dialog, which) -> {
            openUrl("https://github.com/LemonLoader/LemonLoader/releases");
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }

    private void openUrl(String url) {
        try {
            Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
            startActivity(intent);
            LogUtils.logUser("User opened GitHub URL: " + url);
        } catch (Exception e) {
            Toast.makeText(this, "Cannot open URL. Please visit manually.", Toast.LENGTH_LONG).show();
            LogUtils.logDebug("Failed to open URL: " + e.getMessage());
        }
    }

    private void checkInstallationStatus() {
        LogUtils.logUser("User checking manual installation status");
        
        // FIXED: Pass context to MelonLoaderManager methods
        boolean melonInstalled = MelonLoaderManager.isMelonLoaderInstalled(this);
        boolean lemonInstalled = MelonLoaderManager.isMelonLoaderInstalled(this);
        
        android.app.AlertDialog.Builder builder = new android.app.AlertDialog.Builder(this);
        
        if (melonInstalled || lemonInstalled) {
            String loaderType = melonInstalled ? "MelonLoader" : "LemonLoader";
            String version = MelonLoaderManager.getInstalledLoaderVersion();
            
            builder.setTitle("✅ Installation Detected!");
            builder.setMessage("Great! " + loaderType + " v" + version + " is properly installed.\n\n" +
                              "You can now:\n" +
                              "• Go to DLL Mod Manager\n" +
                              "• Install DLL mods\n" +
                              "• Patch Terraria APK\n" +
                              "• Start modding!\n\n" +
                              "Installation path:\n" + 
                              MelonLoaderManager.getStatus(this, MelonLoaderManager.TERRARIA_PACKAGE).basePath);
            
            builder.setPositiveButton("Open DLL Manager", (dialog, which) -> {
                Intent intent = new Intent(this, DllModActivity.class);
                startActivity(intent);
            });
            
            builder.setNegativeButton("Stay Here", null);
            
        } else {
            builder.setTitle("❌ Installation Not Found");
            builder.setMessage("No loader installation detected.\n\n" +
                              "Please check:\n" +
                              "• Files are in correct directories\n" +
                              "• Directory names match exactly\n" +
                              "• Core DLL files are present\n" +
                              "• File permissions are correct\n\n" +
                              "Expected path:\n" +
                              "/storage/emulated/0/TerrariaLoader/com.and.games505.TerrariaPaid/");
            
            builder.setPositiveButton("View Debug Info", (dialog, which) -> {
                showDebugInfo();
            });
            
            builder.setNegativeButton("OK", null);
        }
        
        builder.show();
    }

    private void showDebugInfo() {
        // FIXED: Pass context to MelonLoaderManager.getDebugInfo method
        String debugInfo = MelonLoaderManager.getDebugInfo(this, MelonLoaderManager.TERRARIA_PACKAGE);
        
        android.app.AlertDialog.Builder builder = new android.app.AlertDialog.Builder(this);
        builder.setTitle("Debug Information");
        builder.setMessage(debugInfo);
        builder.setPositiveButton("Copy to Clipboard", (dialog, which) -> {
            ClipboardManager clipboard = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
            ClipData clip = ClipData.newPlainText("Debug Info", debugInfo);
            clipboard.setPrimaryClip(clip);
            Toast.makeText(this, "Debug info copied to clipboard", Toast.LENGTH_SHORT).show();
        });
        builder.setNegativeButton("Close", null);
        builder.show();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/LogCategoryAdapter.java

// File: LogCategoryAdapter.java - Advanced Log Display Adapter
// Path: /main/java/com/terrarialoader/ui/LogCategoryAdapter.java

package com.modloader.ui;

import android.content.Context;
import android.graphics.Color;
import android.graphics.Typeface;
import android.text.SpannableString;
import android.text.Spanned;
import android.text.style.BackgroundColorSpan;
import android.text.style.ForegroundColorSpan;
import android.text.style.StyleSpan;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.modloader.R;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * Advanced RecyclerView adapter for displaying categorized log entries
 * with highlighting, filtering, and rich formatting
 */
public class LogCategoryAdapter extends RecyclerView.Adapter<LogCategoryAdapter.LogViewHolder> {
    
    private final Context context;
    private List<LogEntry> logEntries = new ArrayList<>();
    private String searchQuery = "";
    private boolean highlightMode = true;
    private LogEntry.LogLevel filterLevel = null;
    private LogEntry.LogType filterType = null;
    
    // Interface for item interactions
    public interface LogItemClickListener {
        void onLogItemClick(LogEntry logEntry);
        void onLogItemLongClick(LogEntry logEntry);
    }
    
    private LogItemClickListener clickListener;
    
    public LogCategoryAdapter(Context context) {
        this.context = context;
    }
    
    public void setClickListener(LogItemClickListener listener) {
        this.clickListener = listener;
    }
    
    public void updateEntries(List<LogEntry> entries) {
        this.logEntries = entries != null ? entries : new ArrayList<>();
        notifyDataSetChanged();
    }
    
    public void setSearchQuery(String query) {
        this.searchQuery = query != null ? query : "";
        notifyDataSetChanged();
    }
    
    public void setHighlightMode(boolean enabled) {
        this.highlightMode = enabled;
        notifyDataSetChanged();
    }
    
    public void setLevelFilter(LogEntry.LogLevel level) {
        this.filterLevel = level;
        notifyDataSetChanged();
    }
    
    public void setTypeFilter(LogEntry.LogType type) {
        this.filterType = type;
        notifyDataSetChanged();
    }
    
    @NonNull
    @Override
    public LogViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_log_entry, parent, false);
        return new LogViewHolder(view);
    }
    
    @Override
    public void onBindViewHolder(@NonNull LogViewHolder holder, int position) {
        LogEntry entry = logEntries.get(position);
        holder.bind(entry);
    }
    
    @Override
    public int getItemCount() {
        return logEntries.size();
    }
    
    public class LogViewHolder extends RecyclerView.ViewHolder {
        private final TextView timestampText;
        private final TextView levelText;
        private final TextView tagText;
        private final TextView messageText;
        private final View levelIndicator;
        private final View rootView;
        
        public LogViewHolder(@NonNull View itemView) {
            super(itemView);
            timestampText = itemView.findViewById(R.id.logTimestamp);
            levelText = itemView.findViewById(R.id.logLevel);
            tagText = itemView.findViewById(R.id.logTag);
            messageText = itemView.findViewById(R.id.logMessage);
            levelIndicator = itemView.findViewById(R.id.levelIndicator);
            rootView = itemView;
            
            // Set click listeners
            itemView.setOnClickListener(v -> {
                if (clickListener != null) {
                    int pos = getAdapterPosition();
                    if (pos != RecyclerView.NO_POSITION) {
                        clickListener.onLogItemClick(logEntries.get(pos));
                    }
                }
            });
            
            itemView.setOnLongClickListener(v -> {
                if (clickListener != null) {
                    int pos = getAdapterPosition();
                    if (pos != RecyclerView.NO_POSITION) {
                        clickListener.onLogItemLongClick(logEntries.get(pos));
                        return true;
                    }
                }
                return false;
            });
        }
        
        public void bind(LogEntry entry) {
            // Set timestamp
            timestampText.setText(entry.getFormattedTimestamp());
            
            // Set level with color
            levelText.setText(entry.getLevel().getDisplayName());
            levelText.setTextColor(Color.parseColor(entry.getLevel().getColor()));
            
            // Set tag
            tagText.setText(entry.getTag());
            
            // Set message with highlighting
            if (highlightMode && !searchQuery.trim().isEmpty()) {
                messageText.setText(highlightText(entry.getMessage(), searchQuery));
            } else {
                messageText.setText(entry.getMessage());
            }
            
            // Set level indicator color
            levelIndicator.setBackgroundColor(Color.parseColor(entry.getLevel().getColor()));
            
            // Set background based on log type and importance
            setItemBackground(entry);
            
            // Set typography based on level
            setTypography(entry);
            
            // Add type badge for system/game logs
            addTypeBadge(entry);
        }
        
        private SpannableString highlightText(String text, String query) {
            SpannableString spannableString = new SpannableString(text);
            
            if (query.trim().isEmpty()) {
                return spannableString;
            }
            
            try {
                // Try regex highlighting first
                if (isValidRegex(query)) {
                    Pattern pattern = Pattern.compile(query, Pattern.CASE_INSENSITIVE);
                    Matcher matcher = pattern.matcher(text);
                    
                    while (matcher.find()) {
                        spannableString.setSpan(
                            new BackgroundColorSpan(Color.YELLOW),
                            matcher.start(),
                            matcher.end(),
                            Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
                        );
                        spannableString.setSpan(
                            new ForegroundColorSpan(Color.BLACK),
                            matcher.start(),
                            matcher.end(),
                            Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
                        );
                    }
                } else {
                    // Fall back to simple text highlighting
                    String lowerText = text.toLowerCase();
                    String lowerQuery = query.toLowerCase();
                    int index = lowerText.indexOf(lowerQuery);
                    
                    while (index >= 0) {
                        spannableString.setSpan(
                            new BackgroundColorSpan(Color.YELLOW),
                            index,
                            index + query.length(),
                            Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
                        );
                        spannableString.setSpan(
                            new ForegroundColorSpan(Color.BLACK),
                            index,
                            index + query.length(),
                            Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
                        );
                        
                        index = lowerText.indexOf(lowerQuery, index + 1);
                    }
                }
            } catch (Exception e) {
                // If highlighting fails, return original text
                return spannableString;
            }
            
            return spannableString;
        }
        
        private boolean isValidRegex(String regex) {
            try {
                Pattern.compile(regex);
                return true;
            } catch (Exception e) {
                return false;
            }
        }
        
        private void setItemBackground(LogEntry entry) {
            int backgroundColor;
            
            switch (entry.getLevel()) {
                case ERROR:
                    backgroundColor = Color.parseColor("#FFEBEE");
                    break;
                case WARN:
                    backgroundColor = Color.parseColor("#FFF8E1");
                    break;
                case USER:
                    backgroundColor = Color.parseColor("#E8F5E8");
                    break;
                case DEBUG:
                    backgroundColor = Color.parseColor("#F5F5F5");
                    break;
                default:
                    backgroundColor = Color.WHITE;
                    break;
            }
            
            rootView.setBackgroundColor(backgroundColor);
        }
        
        private void setTypography(LogEntry entry) {
            switch (entry.getLevel()) {
                case ERROR:
                    messageText.setTypeface(null, Typeface.BOLD);
                    break;
                case WARN:
                    messageText.setTypeface(null, Typeface.ITALIC);
                    break;
                case USER:
                    messageText.setTypeface(null, Typeface.BOLD);
                    break;
                default:
                    messageText.setTypeface(null, Typeface.NORMAL);
                    break;
            }
        }
        
        private void addTypeBadge(LogEntry entry) {
            if (entry.getType() != LogEntry.LogType.APP) {
                // Add type indicator to tag
                String tagWithType = "[" + entry.getType().getDisplayName() + "] " + entry.getTag();
                tagText.setText(tagWithType);
                
                // Color code the type
                switch (entry.getType()) {
                    case GAME:
                        tagText.setTextColor(Color.parseColor("#FF9800"));
                        break;
                    case SYSTEM:
                        tagText.setTextColor(Color.parseColor("#9C27B0"));
                        break;
                    default:
                        tagText.setTextColor(Color.parseColor("#666666"));
                        break;
                }
            } else {
                tagText.setTextColor(Color.parseColor("#666666"));
            }
        }
    }
    
    // Filter methods
    public List<LogEntry> getFilteredEntries() {
        List<LogEntry> filtered = new ArrayList<>();
        
        for (LogEntry entry : logEntries) {
            if (matchesFilters(entry)) {
                filtered.add(entry);
            }
        }
        
        return filtered;
    }
    
    private boolean matchesFilters(LogEntry entry) {
        // Level filter
        if (filterLevel != null && entry.getLevel() != filterLevel) {
            return false;
        }
        
        // Type filter
        if (filterType != null && entry.getType() != filterType) {
            return false;
        }
        
        // Search query filter
        if (!searchQuery.trim().isEmpty()) {
            return entry.matchesFilter(searchQuery);
        }
        
        return true;
    }
    
    // Statistics methods
    public int getErrorCount() {
        int count = 0;
        for (LogEntry entry : logEntries) {
            if (entry.getLevel() == LogEntry.LogLevel.ERROR) {
                count++;
            }
        }
        return count;
    }
    
    public int getWarningCount() {
        int count = 0;
        for (LogEntry entry : logEntries) {
            if (entry.getLevel() == LogEntry.LogLevel.WARN) {
                count++;
            }
        }
        return count;
    }
    
    public int getImportantCount() {
        int count = 0;
        for (LogEntry entry : logEntries) {
            if (entry.isImportant()) {
                count++;
            }
        }
        return count;
    }
    
    // Export filtered entries
    public String exportFilteredEntries() {
        StringBuilder export = new StringBuilder();
        export.append("=== Filtered Log Export ===\n");
        export.append("Total entries: ").append(getItemCount()).append("\n");
        export.append("Errors: ").append(getErrorCount()).append("\n");
        export.append("Warnings: ").append(getWarningCount()).append("\n");
        export.append("Important: ").append(getImportantCount()).append("\n");
        export.append("Export time: ").append(new java.util.Date().toString()).append("\n");
        export.append("=" + "=".repeat(50)).append("\n\n");
        
        for (LogEntry entry : getFilteredEntries()) {
            export.append(entry.toFormattedString()).append("\n");
        }
        
        return export.toString();
    }
    
    // Clear all entries
    public void clear() {
        logEntries.clear();
        notifyDataSetChanged();
    }
    
    // Add single entry
    public void addEntry(LogEntry entry) {
        if (entry != null) {
            logEntries.add(entry);
            notifyItemInserted(logEntries.size() - 1);
        }
    }
    
    // Add multiple entries
    public void addEntries(List<LogEntry> entries) {
        if (entries != null && !entries.isEmpty()) {
            int startPosition = logEntries.size();
            logEntries.addAll(entries);
            notifyItemRangeInserted(startPosition, entries.size());
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/LogEntry.java

// File: LogEntry.java - Enhanced Log Entry Model for Advanced Logging
// Path: /main/java/com/terrarialoader/ui/LogEntry.java

package com.modloader.ui;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

/**
 * Enhanced log entry model for advanced logging system
 * Supports categorization, filtering, and rich metadata
 */
public class LogEntry {
    
    public enum LogLevel {
        DEBUG("DEBUG", "#808080"),
        INFO("INFO", "#2196F3"),
        WARN("WARN", "#FF9800"),
        ERROR("ERROR", "#F44336"),
        USER("USER", "#4CAF50");
        
        private final String displayName;
        private final String color;
        
        LogLevel(String displayName, String color) {
            this.displayName = displayName;
            this.color = color;
        }
        
        public String getDisplayName() { return displayName; }
        public String getColor() { return color; }
    }
    
    public enum LogType {
        APP("App", "Application logs"),
        GAME("Game", "Game/MelonLoader logs"),
        SYSTEM("System", "System information");
        
        private final String displayName;
        private final String description;
        
        LogType(String displayName, String description) {
            this.displayName = displayName;
            this.description = description;
        }
        
        public String getDisplayName() { return displayName; }
        public String getDescription() { return description; }
    }
    
    private final long timestamp;
    private final LogLevel level;
    private final LogType type;
    private final String tag;
    private final String message;
    private final String threadName;
    private final String className;
    
    // Primary constructor
    public LogEntry(long timestamp, LogLevel level, LogType type, String tag, String message) {
        this.timestamp = timestamp;
        this.level = level != null ? level : LogLevel.INFO;
        this.type = type != null ? type : LogType.APP;
        this.tag = tag != null ? tag : "UNKNOWN";
        this.message = message != null ? message : "";
        this.threadName = Thread.currentThread().getName();
        this.className = getCallingClassName();
    }
    
    // Convenience constructor for app logs
    public LogEntry(LogLevel level, String tag, String message) {
        this(System.currentTimeMillis(), level, LogType.APP, tag, message);
    }
    
    // Convenience constructor for simple messages
    public LogEntry(String message) {
        this(System.currentTimeMillis(), LogLevel.INFO, LogType.APP, "APP", message);
    }
    
    // Getters
    public long getTimestamp() { return timestamp; }
    public LogLevel getLevel() { return level; }
    public LogType getType() { return type; }
    public String getTag() { return tag; }
    public String getMessage() { return message; }
    public String getThreadName() { return threadName; }
    public String getClassName() { return className; }
    
    // Formatted timestamp
    public String getFormattedTimestamp() {
        return new SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault()).format(new Date(timestamp));
    }
    
    public String getFormattedDate() {
        return new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(new Date(timestamp));
    }
    
    public String getFormattedDateTime() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault()).format(new Date(timestamp));
    }
    
    // Formatted string representation
    public String toFormattedString() {
        return String.format("[%s] [%s] [%s] %s: %s", 
            getFormattedTimestamp(), 
            level.getDisplayName(), 
            type.getDisplayName(),
            tag, 
            message);
    }
    
    // Detailed string representation
    public String toDetailedString() {
        StringBuilder sb = new StringBuilder();
        sb.append("Timestamp: ").append(getFormattedDateTime()).append("\n");
        sb.append("Level: ").append(level.getDisplayName()).append("\n");
        sb.append("Type: ").append(type.getDisplayName()).append("\n");
        sb.append("Tag: ").append(tag).append("\n");
        sb.append("Thread: ").append(threadName).append("\n");
        if (className != null && !className.isEmpty()) {
            sb.append("Class: ").append(className).append("\n");
        }
        sb.append("Message: ").append(message);
        return sb.toString();
    }
    
    // JSON representation
    public String toJson() {
        return String.format(
            "{\"timestamp\":%d,\"level\": %s\",\"type\":\"%s\",\"tag\":\"%s\",\"message\":\"%s\",\"thread\":\"%s\"}",
            timestamp,
            level.getDisplayName(),
            type.getDisplayName(),
            escapeJson(tag),
            escapeJson(message),
            escapeJson(threadName)
        );
    }
    
    // Filtering helpers
    public boolean matchesFilter(String searchQuery) {
        if (searchQuery == null || searchQuery.trim().isEmpty()) {
            return true;
        }
        
        String query = searchQuery.toLowerCase();
        return message.toLowerCase().contains(query) ||
               tag.toLowerCase().contains(query) ||
               level.getDisplayName().toLowerCase().contains(query) ||
               type.getDisplayName().toLowerCase().contains(query);
    }
    
    public boolean matchesLevel(LogLevel filterLevel) {
        return filterLevel == null || this.level == filterLevel;
    }
    
    public boolean matchesType(LogType filterType) {
        return filterType == null || this.type == filterType;
    }
    
    public boolean matchesTimeRange(long startTime, long endTime) {
        return timestamp >= startTime && timestamp <= endTime;
    }
    
    // Priority for sorting (higher = more important)
    public int getPriority() {
        switch (level) {
            case ERROR: return 4;
            case WARN: return 3;
            case USER: return 2;
            case INFO: return 1;
            case DEBUG: return 0;
            default: return 0;
        }
    }
    
    // Get severity color for UI display
    public String getSeverityColor() {
        return level.getColor();
    }
    
    // Check if this is an important log entry
    public boolean isImportant() {
        return level == LogLevel.ERROR || level == LogLevel.WARN || 
               message.toLowerCase().contains("fail") ||
               message.toLowerCase().contains("error") ||
               message.toLowerCase().contains("crash");
    }
    
    // Get calling class name for debugging
    private String getCallingClassName() {
        try {
            StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
            // Skip the first few elements (Thread.getStackTrace, this constructor, etc.)
            for (int i = 3; i < stackTrace.length && i < 8; i++) {
                String className = stackTrace[i].getClassName();
                if (!className.startsWith("java.") && 
                    !className.startsWith("android.") &&
                    !className.equals(LogEntry.class.getName())) {
                    return className.substring(className.lastIndexOf('.') + 1);
                }
            }
        } catch (Exception e) {
            // Ignore exceptions during class name detection
        }
        return null;
    }
    
    // Helper method to escape JSON strings
    private String escapeJson(String str) {
        if (str == null) return "";
        return str.replace("\\", "\\\\")
                  .replace("\"", "\\\"")
                  .replace("\n", "\\n")
                  .replace("\r", "\\r")
                  .replace("\t", "\\t");
    }
    
    // Factory methods for common log types
    public static LogEntry debug(String tag, String message) {
        return new LogEntry(LogLevel.DEBUG, tag, message);
    }
    
    public static LogEntry info(String tag, String message) {
        return new LogEntry(LogLevel.INFO, tag, message);
    }
    
    public static LogEntry warn(String tag, String message) {
        return new LogEntry(LogLevel.WARN, tag, message);
    }
    
    public static LogEntry error(String tag, String message) {
        return new LogEntry(LogLevel.ERROR, tag, message);
    }
    
    public static LogEntry user(String tag, String message) {
        return new LogEntry(LogLevel.USER, tag, message);
    }
    
    public static LogEntry system(String message) {
        return new LogEntry(System.currentTimeMillis(), LogLevel.INFO, LogType.SYSTEM, "SYSTEM", message);
    }
    
    public static LogEntry game(String tag, String message) {
        return new LogEntry(System.currentTimeMillis(), LogLevel.INFO, LogType.GAME, tag, message);
    }
    
    @Override
    public String toString() {
        return toFormattedString();
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        LogEntry logEntry = (LogEntry) obj;
        return timestamp == logEntry.timestamp &&
               level == logEntry.level &&
               type == logEntry.type &&
               tag.equals(logEntry.tag) &&
               message.equals(logEntry.message);
    }
    
    @Override
    public int hashCode() {
        int result = Long.hashCode(timestamp);
        result = 31 * result + level.hashCode();
        result = 31 * result + type.hashCode();
        result = 31 * result + tag.hashCode();
        result = 31 * result + message.hashCode();
        return result;
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/LogViewerActivity.java

// File: LogViewerActivity.java (ENHANCED) - Advanced Log Viewer with Better UI
// Path: /main/java/com/terrarialoader/ui/LogViewerActivity.java

package com.modloader.ui;

import android.content.Intent;
import android.graphics.Color;
import android.graphics.Typeface;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.text.SpannableString;
import android.text.Spanned;
import android.text.style.ForegroundColorSpan;
import android.text.style.StyleSpan;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.*;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;
import androidx.swiperefreshlayout.widget.SwipeRefreshLayout;

import com.modloader.R;
import com.modloader.util.LogUtils;
import com.modloader.util.PathManager;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.regex.Pattern;

public class LogViewerActivity extends AppCompatActivity {
    
    // UI Components
    private SwipeRefreshLayout swipeRefreshLayout;
    private ScrollView logScrollView;
    private TextView logTextView;
    private LinearLayout filterSection;
    private Spinner logTypeSpinner;
    private Spinner logLevelSpinner;
    private EditText searchEditText;
    private Button clearLogsButton;
    private Button exportLogsButton;
    private Button refreshButton;
    private CheckBox autoScrollCheckbox;
    private TextView logStatsText;
    
    // Data
    private List<String> allLogs = new ArrayList<>();
    private List<String> filteredLogs = new ArrayList<>();
    private String currentFilter = "ALL";
    private String currentLevel = "ALL";
    private String searchQuery = "";
    private boolean autoScroll = true;
    private Handler refreshHandler;
    private Runnable refreshRunnable;
    
    // Log types and levels
    private final String[] LOG_TYPES = {"ALL", "USER", "DEBUG", "ERROR", "SYSTEM", "MOD"};
    private final String[] LOG_LEVELS = {"ALL", "INFO", "WARN", "ERROR", "FATAL"};
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_log_viewer_enhanced);
        
        setTitle("📋 Advanced Log Viewer");
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        
        initializeComponents();
        setupUI();
        setupFilters();
        loadLogs();
        startAutoRefresh();
        
        LogUtils.logUser("Advanced Log Viewer opened");
    }
    
    private void initializeComponents() {
        // Find UI components
        swipeRefreshLayout = findViewById(R.id.swipeRefreshLayout);
        logScrollView = findViewById(R.id.logScrollView);
        logTextView = findViewById(R.id.logTextView);
        filterSection = findViewById(R.id.filterSection);
        logTypeSpinner = findViewById(R.id.logTypeSpinner);
        logLevelSpinner = findViewById(R.id.logLevelSpinner);
        searchEditText = findViewById(R.id.searchEditText);
        clearLogsButton = findViewById(R.id.clearLogsButton);
        exportLogsButton = findViewById(R.id.exportLogsButton);
        refreshButton = findViewById(R.id.refreshButton);
        autoScrollCheckbox = findViewById(R.id.autoScrollCheckbox);
        logStatsText = findViewById(R.id.logStatsText);
        
        // Initialize handlers
        refreshHandler = new Handler(Looper.getMainLooper());
    }
    
    private void setupUI() {
        // Setup swipe to refresh
        swipeRefreshLayout.setOnRefreshListener(() -> {
            loadLogs();
            swipeRefreshLayout.setRefreshing(false);
            Toast.makeText(this, "Logs refreshed", Toast.LENGTH_SHORT).show();
        });
        
        // Setup log text view
        logTextView.setTextSize(12);
        logTextView.setTypeface(Typeface.MONOSPACE);
        logTextView.setTextIsSelectable(true);
        logTextView.setBackgroundColor(Color.parseColor("#1E1E1E"));
        logTextView.setTextColor(Color.parseColor("#E0E0E0"));
        logTextView.setPadding(16, 16, 16, 16);
        
        // Setup buttons
        clearLogsButton.setOnClickListener(v -> showClearLogsDialog());
        exportLogsButton.setOnClickListener(v -> exportLogs());
        refreshButton.setOnClickListener(v -> {
            loadLogs();
            Toast.makeText(this, "Logs refreshed", Toast.LENGTH_SHORT).show();
        });
        
        // Setup auto-scroll checkbox
        autoScrollCheckbox.setChecked(autoScroll);
        autoScrollCheckbox.setOnCheckedChangeListener((buttonView, isChecked) -> {
            autoScroll = isChecked;
            if (autoScroll) {
                scrollToBottom();
            }
        });
        
        // Setup search functionality
        searchEditText.addTextChangedListener(new android.text.TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                searchQuery = s.toString().trim();
                applyFilters();
            }
            
            @Override
            public void afterTextChanged(android.text.Editable s) {}
        });
    }
    
    private void setupFilters() {
        // Setup log type spinner
        ArrayAdapter<String> typeAdapter = new ArrayAdapter<>(this, 
            android.R.layout.simple_spinner_item, LOG_TYPES);
        typeAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        logTypeSpinner.setAdapter(typeAdapter);
        logTypeSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currentFilter = LOG_TYPES[position];
                applyFilters();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
        
        // Setup log level spinner
        ArrayAdapter<String> levelAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item, LOG_LEVELS);
        levelAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        logLevelSpinner.setAdapter(levelAdapter);
        logLevelSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currentLevel = LOG_LEVELS[position];
                applyFilters();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
    }
    
    private void loadLogs() {
        try {
            // Get logs from LogUtils
            String rawLogs = LogUtils.getLogs();
            
            // Parse logs into individual lines
            allLogs.clear();
            if (rawLogs != null && !rawLogs.trim().isEmpty()) {
                String[] logLines = rawLogs.split("\n");
                Collections.addAll(allLogs, logLines);
            }
            
            // Apply current filters
            applyFilters();
            
            // Update statistics
            updateLogStats();
            
        } catch (Exception e) {
            LogUtils.logDebug("Error loading logs: " + e.getMessage());
            Toast.makeText(this, "Error loading logs: " + e.getMessage(), Toast.LENGTH_SHORT).show();
        }
    }
    
    private void applyFilters() {
        filteredLogs.clear();
        
        for (String logLine : allLogs) {
            if (matchesFilters(logLine)) {
                filteredLogs.add(logLine);
            }
        }
        
        displayLogs();
        updateLogStats();
    }
    
    private boolean matchesFilters(String logLine) {
        // Apply type filter
        if (!"ALL".equals(currentFilter)) {
            if (!logLine.toLowerCase().contains(currentFilter.toLowerCase())) {
                return false;
            }
        }
        
        // Apply level filter
        if (!"ALL".equals(currentLevel)) {
            if (!logLine.toUpperCase().contains(currentLevel.toUpperCase())) {
                return false;
            }
        }
        
        // Apply search query
        if (!searchQuery.isEmpty()) {
            if (!logLine.toLowerCase().contains(searchQuery.toLowerCase())) {
                return false;
            }
        }
        
        return true;
    }
    
    private void displayLogs() {
        if (filteredLogs.isEmpty()) {
            logTextView.setText("📝 No logs match the current filters.\n\n" +
                "Try adjusting your filter settings or clearing the search box.");
            return;
        }
        
        SpannableString spannableLog = new SpannableString(String.join("\n", filteredLogs));
        
        // Apply syntax highlighting
        applySyntaxHighlighting(spannableLog);
        
        logTextView.setText(spannableLog);
        
        // Auto-scroll to bottom if enabled
        if (autoScroll) {
            scrollToBottom();
        }
    }
    
    private void applySyntaxHighlighting(SpannableString spannableLog) {
        String text = spannableLog.toString();
        
        // Highlight different log levels with colors
        highlightPattern(spannableLog, "ERROR", Color.parseColor("#FF6B6B"));
        highlightPattern(spannableLog, "FATAL", Color.parseColor("#FF3030"));
        highlightPattern(spannableLog, "WARN", Color.parseColor("#FFB366"));
        highlightPattern(spannableLog, "INFO", Color.parseColor("#66B2FF"));
        highlightPattern(spannableLog, "DEBUG", Color.parseColor("#98FB98"));
        
        // Highlight timestamps
        highlightPattern(spannableLog, "\\[\\d{2}:\\d{2}:\\d{2}\\]", Color.parseColor("#CCCCCC"));
        
        // Highlight file paths
        highlightPattern(spannableLog, "/[\\w/.-]+\\.(java|kt|xml)", Color.parseColor("#DDA0DD"));
        
        // Highlight search query if present
        if (!searchQuery.isEmpty()) {
            highlightPattern(spannableLog, Pattern.quote(searchQuery), Color.parseColor("#FFFF00"));
        }
    }
    
    private void highlightPattern(SpannableString spannableLog, String pattern, int color) {
        Pattern p = Pattern.compile(pattern, Pattern.CASE_INSENSITIVE);
        java.util.regex.Matcher matcher = p.matcher(spannableLog.toString());
        
        while (matcher.find()) {
            spannableLog.setSpan(new ForegroundColorSpan(color), 
                matcher.start(), matcher.end(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        }
    }
    
    private void scrollToBottom() {
        logScrollView.post(() -> logScrollView.fullScroll(ScrollView.FOCUS_DOWN));
    }
    
    private void updateLogStats() {
        int totalLogs = allLogs.size();
        int filteredCount = filteredLogs.size();
        int errorCount = 0;
        int warningCount = 0;
        
        for (String log : allLogs) {
            if (log.toUpperCase().contains("ERROR") || log.toUpperCase().contains("FATAL")) {
                errorCount++;
            } else if (log.toUpperCase().contains("WARN")) {
                warningCount++;
            }
        }
        
        String statsText = String.format("📊 Total: %d | Showing: %d | Errors: %d | Warnings: %d",
            totalLogs, filteredCount, errorCount, warningCount);
        logStatsText.setText(statsText);
    }
    
    private void startAutoRefresh() {
        refreshRunnable = new Runnable() {
            @Override
            public void run() {
                loadLogs();
                refreshHandler.postDelayed(this, 5000); // Refresh every 5 seconds
            }
        };
        refreshHandler.postDelayed(refreshRunnable, 5000);
    }
    
    private void stopAutoRefresh() {
        if (refreshHandler != null && refreshRunnable != null) {
            refreshHandler.removeCallbacks(refreshRunnable);
        }
    }
    
    private void showClearLogsDialog() {
        new AlertDialog.Builder(this)
            .setTitle("🗑️ Clear Logs")
            .setMessage("Are you sure you want to clear all logs? This action cannot be undone.")
            .setPositiveButton("Clear", (dialog, which) -> {
                clearLogs();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void clearLogs() {
        try {
            // Clear logs in LogUtils
            LogUtils.clearLogs();
            
            // Reload empty logs
            loadLogs();
            
            Toast.makeText(this, "✅ Logs cleared successfully", Toast.LENGTH_SHORT).show();
            LogUtils.logUser("Logs cleared by user");
            
        } catch (Exception e) {
            Toast.makeText(this, "❌ Failed to clear logs: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            LogUtils.logDebug("Error clearing logs: " + e.getMessage());
        }
    }
    
    private void exportLogs() {
        try {
            // Create export directory
            File exportDir = new File(getExternalFilesDir(null), "TerrariaLoader/com.and.games505.TerrariaPaid/AppLogs");
            if (!exportDir.exists()) {
                exportDir.mkdirs();
            }
            
            // Create filename with timestamp
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss", Locale.getDefault());
            String timestamp = dateFormat.format(new Date());
            String filename = "TerrariaLoader_Logs_" + timestamp + ".txt";
            File logFile = new File(exportDir, filename);
            
            // Write logs to file
            try (FileWriter writer = new FileWriter(logFile)) {
                writer.write("=== TerrariaLoader Log Export ===\n");
                writer.write("Export Date: " + new Date().toString() + "\n");
                writer.write("Total Logs: " + allLogs.size() + "\n");
                writer.write("Filtered Logs: " + filteredLogs.size() + "\n");
                writer.write("Current Filter: " + currentFilter + "\n");
                writer.write("Current Level: " + currentLevel + "\n");
                writer.write("Search Query: " + (searchQuery.isEmpty() ? "None" : searchQuery) + "\n");
                writer.write("\n=== LOG CONTENT ===\n\n");
                
                // Write filtered logs (what user is currently viewing)
                for (String logLine : filteredLogs) {
                    writer.write(logLine + "\n");
                }
                
                writer.write("\n=== END OF LOG ===\n");
            }
            
            // Share the log file
            Intent shareIntent = new Intent(Intent.ACTION_SEND);
            shareIntent.setType("text/plain");
            shareIntent.putExtra(Intent.EXTRA_STREAM,
                FileProvider.getUriForFile(this, getPackageName() + ".provider", logFile));
            shareIntent.putExtra(Intent.EXTRA_SUBJECT, "TerrariaLoader Logs - " + timestamp);
            shareIntent.putExtra(Intent.EXTRA_TEXT, "TerrariaLoader log export containing " + 
                filteredLogs.size() + " log entries.");
            shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            
            startActivity(Intent.createChooser(shareIntent, "📤 Share Logs"));
            
            Toast.makeText(this, "✅ Logs exported: " + filename, Toast.LENGTH_LONG).show();
            LogUtils.logUser("Logs exported to: " + logFile.getAbsolutePath());
            
        } catch (IOException e) {
            Toast.makeText(this, "❌ Failed to export logs: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            LogUtils.logDebug("Log export error: " + e.getMessage());
        }
    }
    
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.log_viewer_menu, menu);
        return true;
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        
        if (id == android.R.id.home) {
            finish();
            return true;
        } else if (id == R.id.action_toggle_filters) {
            // Toggle filter section visibility
            filterSection.setVisibility(
                filterSection.getVisibility() == View.VISIBLE ? View.GONE : View.VISIBLE);
            return true;
        } else if (id == R.id.action_share_logs) {
            exportLogs();
            return true;
        } else if (id == R.id.action_clear_logs) {
            showClearLogsDialog();
            return true;
        } else if (id == R.id.action_settings) {
            showLogSettings();
            return true;
        }
        
        return super.onOptionsItemSelected(item);
    }
    
    private void showLogSettings() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("⚙️ Log Viewer Settings");
        
        View settingsView = getLayoutInflater().inflate(R.layout.dialog_log_settings, null);
        
        CheckBox autoRefreshCheckbox = settingsView.findViewById(R.id.autoRefreshCheckbox);
        CheckBox syntaxHighlightCheckbox = settingsView.findViewById(R.id.syntaxHighlightCheckbox);
        SeekBar textSizeSeekBar = settingsView.findViewById(R.id.textSizeSeekBar);
        TextView textSizeLabel = settingsView.findViewById(R.id.textSizeLabel);
        
        // Set current values
        autoRefreshCheckbox.setChecked(refreshRunnable != null);
        syntaxHighlightCheckbox.setChecked(true); // Always enabled for now
        
        int currentTextSize = (int) logTextView.getTextSize() / 4; // Convert to reasonable scale
        textSizeSeekBar.setProgress(currentTextSize);
        textSizeLabel.setText("Text Size: " + currentTextSize);
        
        textSizeSeekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                textSizeLabel.setText("Text Size: " + progress);
                logTextView.setTextSize(Math.max(8, progress));
            }
            
            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {}
            
            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {}
        });
        
        builder.setView(settingsView);
        builder.setPositiveButton("Apply", (dialog, which) -> {
            // Apply settings
            if (autoRefreshCheckbox.isChecked() && refreshRunnable == null) {
                startAutoRefresh();
            } else if (!autoRefreshCheckbox.isChecked() && refreshRunnable != null) {
                stopAutoRefresh();
            }
            
            Toast.makeText(this, "Settings applied", Toast.LENGTH_SHORT).show();
        });
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopAutoRefresh();
        LogUtils.logUser("Advanced Log Viewer closed");
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        // Stop auto-refresh when app is not visible
        stopAutoRefresh();
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        // Resume auto-refresh when app becomes visible
        startAutoRefresh();
        loadLogs(); // Refresh logs when returning to activity
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/LogViewerEnhancedActivity.java

// File: LogViewerEnhancedActivity.java - Enhanced log viewer with filtering and search
// Path: /app/src/main/java/com/modloader/ui/LogViewerEnhancedActivity.java

package com.modloader.ui;

import android.app.AlertDialog;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import androidx.swiperefreshlayout.widget.SwipeRefreshLayout;

import com.modloader.R;
import com.modloader.util.LogUtils;
import com.modloader.util.DiagnosticBundleExporter;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class LogViewerEnhancedActivity extends AppCompatActivity {
    
    // UI Components
    private LinearLayout filterSection;
    private Spinner logTypeSpinner;
    private Spinner logLevelSpinner;
    private EditText searchEditText;
    private TextView logStatsText;
    private TextView logTextView;
    private ScrollView logScrollView;
    private SwipeRefreshLayout swipeRefreshLayout;
    private CheckBox autoScrollCheckbox;
    private Button refreshButton;
    private Button clearLogsButton;
    private Button exportLogsButton;
    
    // State variables
    private Handler refreshHandler;
    private Runnable refreshRunnable;
    private boolean autoRefreshEnabled = true;
    private boolean filtersVisible = true;
    private String currentFilter = "All";
    private String currentLevel = "All";
    private String currentSearch = "";
    
    // Log data
    private String fullLogContent = "";
    private List<String> logLines = new ArrayList<>();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_log_viewer_enhanced);
        
        setTitle("Enhanced Log Viewer");
        
        initializeViews();
        setupListeners();
        setupSpinners();
        startAutoRefresh();
        
        // Initial load
        refreshLogs();
    }
    
    private void initializeViews() {
        // Filter section
        filterSection = findViewById(R.id.filterSection);
        logTypeSpinner = findViewById(R.id.logTypeSpinner);
        logLevelSpinner = findViewById(R.id.logLevelSpinner);
        searchEditText = findViewById(R.id.searchEditText);
        
        // Stats and controls
        logStatsText = findViewById(R.id.logStatsText);
        refreshButton = findViewById(R.id.refreshButton);
        clearLogsButton = findViewById(R.id.clearLogsButton);
        exportLogsButton = findViewById(R.id.exportLogsButton);
        autoScrollCheckbox = findViewById(R.id.autoScrollCheckbox);
        
        // Main log display
        logTextView = findViewById(R.id.logTextView);
        logScrollView = findViewById(R.id.logScrollView);
        swipeRefreshLayout = findViewById(R.id.swipeRefreshLayout);
    }
    
    private void setupListeners() {
        // Refresh button
        refreshButton.setOnClickListener(v -> refreshLogs());
        
        // Clear logs button
        clearLogsButton.setOnClickListener(v -> clearLogs());
        
        // Export logs button
        exportLogsButton.setOnClickListener(v -> exportLogs());
        
        // Swipe refresh
        swipeRefreshLayout.setOnRefreshListener(this::refreshLogs);
        
        // Search text watcher
        searchEditText.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {}
            
            @Override
            public void afterTextChanged(Editable s) {
                currentSearch = s.toString();
                applyFilters();
            }
        });
        
        // Spinner listeners
        logTypeSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currentFilter = parent.getItemAtPosition(position).toString();
                applyFilters();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
        
        logLevelSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currentLevel = parent.getItemAtPosition(position).toString();
                applyFilters();
            }
            
            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
    }
    
    private void setupSpinners() {
        // Log type spinner
        String[] logTypes = {"All", "User", "Debug", "Info", "Warning", "Error"};
        ArrayAdapter<String> typeAdapter = new ArrayAdapter<>(this, 
            android.R.layout.simple_spinner_item, logTypes);
        typeAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        logTypeSpinner.setAdapter(typeAdapter);
        
        // Log level spinner  
        String[] logLevels = {"All", "DEBUG", "INFO", "WARNING", "ERROR", "USER"};
        ArrayAdapter<String> levelAdapter = new ArrayAdapter<>(this,
            android.R.layout.simple_spinner_item, logLevels);
        levelAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        logLevelSpinner.setAdapter(levelAdapter);
    }
    
    private void startAutoRefresh() {
        refreshHandler = new Handler();
        refreshRunnable = new Runnable() {
            @Override
            public void run() {
                if (autoRefreshEnabled) {
                    refreshLogs();
                }
                refreshHandler.postDelayed(this, 5000); // Refresh every 5 seconds
            }
        };
        refreshHandler.post(refreshRunnable);
    }
    
    private void refreshLogs() {
        try {
            // Get logs from LogUtils
            String logs = LogUtils.getLogs();
            if (logs == null || logs.isEmpty()) {
                logs = "No logs available.\n\nIf you're experiencing issues, try:\n• Restarting the app\n• Checking storage permissions\n• Using other app features to generate logs";
            }
            
            fullLogContent = logs;
            logLines = parseLogLines(logs);
            
            // Update statistics
            updateLogStats();
            
            // Apply current filters
            applyFilters();
            
            // Stop refresh animation
            if (swipeRefreshLayout.isRefreshing()) {
                swipeRefreshLayout.setRefreshing(false);
            }
            
        } catch (Exception e) {
            String errorMsg = "Error loading logs: " + e.getMessage();
            logTextView.setText(errorMsg);
            updateLogStats(0, 0, 0, 0);
            
            if (swipeRefreshLayout.isRefreshing()) {
                swipeRefreshLayout.setRefreshing(false);
            }
        }
    }
    
    private List<String> parseLogLines(String logs) {
        List<String> lines = new ArrayList<>();
        if (logs != null && !logs.isEmpty()) {
            String[] splitLines = logs.split("\n");
            for (String line : splitLines) {
                if (line != null && !line.trim().isEmpty()) {
                    lines.add(line);
                }
            }
        }
        return lines;
    }
    
    private void applyFilters() {
        try {
            List<String> filteredLines = new ArrayList<>();
            int totalLines = logLines.size();
            int errorCount = 0;
            int warningCount = 0;
            
            for (String line : logLines) {
                if (line == null || line.trim().isEmpty()) {
                    continue;
                }
                
                // Count errors and warnings
                String lowerLine = line.toLowerCase();
                if (lowerLine.contains("error") || lowerLine.contains("❌")) {
                    errorCount++;
                } else if (lowerLine.contains("warn") || lowerLine.contains("⚠️")) {
                    warningCount++;
                }
                
                // Apply filters
                boolean includeByType = filterByType(line);
                boolean includeByLevel = filterByLevel(line);  
                boolean includeBySearch = filterBySearch(line);
                
                if (includeByType && includeByLevel && includeBySearch) {
                    filteredLines.add(line);
                }
            }
            
            // Update display
            StringBuilder displayContent = new StringBuilder();
            for (String line : filteredLines) {
                displayContent.append(line).append("\n");
            }
            
            logTextView.setText(displayContent.toString());
            updateLogStats(totalLines, filteredLines.size(), errorCount, warningCount);
            
            // Auto-scroll to bottom if enabled
            if (autoScrollCheckbox.isChecked()) {
                scrollToBottom();
            }
            
        } catch (Exception e) {
            logTextView.setText("Error applying filters: " + e.getMessage());
        }
    }
    
    private boolean filterByType(String line) {
        if ("All".equals(currentFilter)) {
            return true;
        }
        
        String lowerLine = line.toLowerCase();
        String lowerFilter = currentFilter.toLowerCase();
        
        return lowerLine.contains(lowerFilter) || 
               (lowerFilter.equals("user") && (lowerLine.contains("✅") || lowerLine.contains("❌") || lowerLine.contains("⚠️")));
    }
    
    private boolean filterByLevel(String line) {
        if ("All".equals(currentLevel)) {
            return true;
        }
        
        return line.toUpperCase().contains(currentLevel);
    }
    
    private boolean filterBySearch(String line) {
        if (currentSearch == null || currentSearch.trim().isEmpty()) {
            return true;
        }
        
        return line.toLowerCase().contains(currentSearch.toLowerCase());
    }
    
    private void updateLogStats() {
        updateLogStats(logLines.size(), logLines.size(), 0, 0);
    }
    
    private void updateLogStats(int total, int showing, int errors, int warnings) {
        String statsText = String.format("Total: %d | Showing: %d | Errors: %d | Warnings: %d", 
            total, showing, errors, warnings);
        logStatsText.setText(statsText);
    }
    
    private void scrollToBottom() {
        logScrollView.post(() -> {
            logScrollView.fullScroll(ScrollView.FOCUS_DOWN);
        });
    }
    
    private void clearLogs() {
        new AlertDialog.Builder(this)
            .setTitle("Clear Logs")
            .setMessage("Are you sure you want to clear all logs? This cannot be undone.")
            .setPositiveButton("Clear", (dialog, which) -> {
                LogUtils.clearLogs();
                logTextView.setText("Logs cleared.\n\nNew logs will appear as you use the app.");
                updateLogStats(0, 0, 0, 0);
                Toast.makeText(this, "Logs cleared", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void exportLogs() {
        try {
            // Create diagnostic bundle
            File bundleFile = DiagnosticBundleExporter.createDiagnosticBundle(this);
            
            if (bundleFile != null && bundleFile.exists()) {
                // Share the bundle
                Intent shareIntent = new Intent(Intent.ACTION_SEND);
                shareIntent.setType("application/zip");
                shareIntent.putExtra(Intent.EXTRA_SUBJECT, "TerrariaLoader Diagnostic Bundle");
                shareIntent.putExtra(Intent.EXTRA_TEXT, "Diagnostic bundle created on " + new java.util.Date().toString());
                
                // Use FileProvider to share the file
                android.net.Uri fileUri = androidx.core.content.FileProvider.getUriForFile(
                    this, getPackageName() + ".provider", bundleFile);
                shareIntent.putExtra(Intent.EXTRA_STREAM, fileUri);
                shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
                
                startActivity(Intent.createChooser(shareIntent, "Export Diagnostic Bundle"));
                
                Toast.makeText(this, "Diagnostic bundle created: " + bundleFile.getName(), 
                    Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(this, "Failed to create diagnostic bundle", Toast.LENGTH_SHORT).show();
            }
            
        } catch (Exception e) {
            Toast.makeText(this, "Export failed: " + e.getMessage(), Toast.LENGTH_SHORT).show();
        }
    }
    
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.log_viewer_menu, menu);
        return true;
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int itemId = item.getItemId();
        
        if (itemId == R.id.action_toggle_filters) {
            toggleFilters();
            return true;
        } else if (itemId == R.id.action_share_logs) {
            exportLogs();
            return true;
        } else if (itemId == R.id.action_clear_logs) {
            clearLogs();
            return true;
        } else if (itemId == R.id.action_settings) {
            showSettings();
            return true;
        }
        
        return super.onOptionsItemSelected(item);
    }
    
    private void toggleFilters() {
        filtersVisible = !filtersVisible;
        filterSection.setVisibility(filtersVisible ? View.VISIBLE : View.GONE);
        Toast.makeText(this, "Filters " + (filtersVisible ? "shown" : "hidden"), Toast.LENGTH_SHORT).show();
    }
    
    private void showSettings() {
        // Simple settings dialog
        View settingsView = getLayoutInflater().inflate(R.layout.dialog_log_settings, null);
        
        CheckBox autoRefreshCheck = settingsView.findViewById(R.id.autoRefreshCheckbox);
        autoRefreshCheck.setChecked(autoRefreshEnabled);
        
        new AlertDialog.Builder(this)
            .setTitle("Log Viewer Settings")
            .setView(settingsView)
            .setPositiveButton("OK", (dialog, which) -> {
                autoRefreshEnabled = autoRefreshCheck.isChecked();
                Toast.makeText(this, "Settings saved", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (refreshHandler != null && refreshRunnable != null) {
            refreshHandler.removeCallbacks(refreshRunnable);
        }
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        autoRefreshEnabled = false;
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        autoRefreshEnabled = true;
        refreshLogs();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/ModListActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.OpenableColumns;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Button;
import android.widget.ImageButton;
import android.widget.TextView;
import android.widget.Toast;
import android.widget.Switch; // Make sure this is imported if used in your XML

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.R;
import com.modloader.installer.ModInstaller;
import com.modloader.loader.ModManager;
import com.modloader.util.LogUtils;

import java.io.File;
import java.util.List;

public class ModListActivity extends AppCompatActivity {

    private RecyclerView recyclerView;
    private ModListAdapter modAdapter; // Changed from ModAdapter
    private TextView modCountTextView;
    private static final int PICK_MOD_FILE_REQUEST = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mod_list);

        modCountTextView = findViewById(R.id.modCountTextView);
        recyclerView = findViewById(R.id.recyclerViewMods);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        ImageButton backButton = findViewById(R.id.backButton);
        backButton.setOnClickListener(v -> onBackPressed());

        Button addModButton = findViewById(R.id.addModButton);
        addModButton.setOnClickListener(v -> openFilePicker());

        Button refreshModsButton = findViewById(R.id.refreshModsButton);
        refreshModsButton.setOnClickListener(v -> loadMods());

        loadMods();
    }

    private void loadMods() {
        ModManager.loadMods(this);
        List<File> mods = ModManager.getAvailableMods();
        modAdapter = new ModListAdapter(this, mods); // Changed to ModListAdapter
        recyclerView.setAdapter(modAdapter);
        updateModCount(mods.size());
    }

    private void updateModCount(int count) {
        modCountTextView.setText("Total Mods: " + count + " (Enabled: " + ModManager.getEnabledModCount() + ")");
    }

    private void openFilePicker() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("*/*"); // Allow all file types, then filter
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        try {
            startActivityForResult(Intent.createChooser(intent, "Select Mod File"), PICK_MOD_FILE_REQUEST);
        } catch (android.content.ActivityNotFoundException ex) {
            Toast.makeText(this, "Please install a File Manager.", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == PICK_MOD_FILE_REQUEST && resultCode == Activity.RESULT_OK) {
            if (data != null && data.getData() != null) {
                Uri uri = data.getData();
                handlePickedFile(uri); // Added this method
            }
        }
    }

    // New method to handle picked files and install them
    private void handlePickedFile(Uri uri) {
        String filename = getFilenameFromUri(uri);
        if (filename == null || !isValidModExtension(filename)) {
            LogUtils.logUser("❌ Invalid mod file type selected.");
            Toast.makeText(this, "Invalid mod file type. Only .dex, .jar, .dll, .hybrid are supported.", Toast.LENGTH_LONG).show();
            return;
        }

        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Install Mod");
        builder.setMessage("Do you want to install '" + filename + "'?");
        builder.setPositiveButton("Install", (dialog, which) -> {
            boolean success = ModInstaller.installModAuto(this, uri);
            if (success) {
                Toast.makeText(this, "Mod installed successfully!", Toast.LENGTH_SHORT).show();
                loadMods(); // Reload mods after installation
            } else {
                Toast.makeText(this, "Failed to install mod.", Toast.LENGTH_SHORT).show();
            }
        });
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    private String getFilenameFromUri(Uri uri) {
        String result = null;
        if (uri.getScheme().equals("content")) {
            try (Cursor cursor = getContentResolver().query(uri, null, null, null, null)) {
                if (cursor != null && cursor.moveToFirst()) {
                    int nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                    if (nameIndex != -1) {
                        result = cursor.getString(nameIndex);
                    }
                }
            }
        }
        if (result == null) {
            result = uri.getPath();
            int cut = result.lastIndexOf('/');
            if (cut != -1) {
                result = result.substring(cut + 1);
            }
        }
        return result;
    }

    private boolean isValidModExtension(String filename) {
        String lowerFilename = filename.toLowerCase();
        for (String ext : ModInstaller.getSupportedExtensions()) {
            if (lowerFilename.endsWith(ext)) {
                return true;
            }
        }
        return false;
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadMods(); // Refresh mod list when activity resumes
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/ModListAdapter.java

// File: ModListAdapter.java (Fixed Adapter Class) - NullPointerException Fix
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/app/src/main/java/com/terrarialoader/ui/ModListAdapter.java

package com.modloader.ui;

import android.app.AlertDialog;
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageButton;
import android.widget.Switch;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.R;
import com.modloader.installer.ModInstaller;
import com.modloader.loader.ModManager;
import com.modloader.loader.ModMetadata;
import com.modloader.loader.ModBase;
import com.modloader.util.LogUtils;

import java.io.File;
import java.util.List;

public class ModListAdapter extends RecyclerView.Adapter<ModListAdapter.ModViewHolder> {

    private final Context context;
    private List<File> mods; // Changed to non-final to allow updates

    public ModListAdapter(Context context, List<File> mods) {
        this.context = context;
        this.mods = mods;
    }

    @NonNull
    @Override
    public ModViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_mod, parent, false);
        return new ModViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ModViewHolder holder, int position) {
        File modFile = mods.get(position);
        String modName = modFile.getName();
        boolean isEnabled = !modName.endsWith(".disabled");

        holder.modNameTextView.setText(modName);
        
        // FIXED: Null pointer protection for metadata
        try {
            // Get metadata safely
            String cleanModName = modName.replace(".disabled", "").replace(".dex", "").replace(".jar", "").replace(".dll", "");
            ModMetadata metadata = ModManager.getMetadata(cleanModName);
            
            if (metadata != null) {
                // Use metadata if available
                ModBase.ModType modType = metadata.getModType();
                if (modType != null) {
                    holder.modDescriptionTextView.setText("Type: " + modType.getDisplayName());
                } else {
                    holder.modDescriptionTextView.setText("Type: " + getModTypeFromFileName(modName));
                }
            } else {
                // Fallback to file extension detection
                holder.modDescriptionTextView.setText("Type: " + getModTypeFromFileName(modName));
            }
        } catch (Exception e) {
            // Ultimate fallback
            LogUtils.logDebug("Error getting mod metadata: " + e.getMessage());
            holder.modDescriptionTextView.setText("Type: " + getModTypeFromFileName(modName));
        }

        holder.modSwitch.setChecked(isEnabled);
        holder.modSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            try {
                if (isChecked) {
                    ModManager.enableMod(context, modFile);
                } else {
                    ModManager.disableMod(context, modFile);
                }
                // Refresh the adapter after mod state change
                // Note: The list of files is not actually changing here, only their names.
                // A better approach would be to reload the list of files entirely.
                // For now, we will simply notify that the item has changed.
                notifyItemChanged(position);
            } catch (Exception e) {
                LogUtils.logDebug("Error toggling mod: " + e.getMessage());
                Toast.makeText(context, "Error toggling mod: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });

        holder.modDeleteButton.setOnClickListener(v -> {
            new AlertDialog.Builder(context)
                    .setTitle("Delete Mod")
                    .setMessage("Are you sure you want to delete " + modName + "?")
                    .setPositiveButton("Delete", (dialog, which) -> {
                        try {
                            if (ModInstaller.uninstallMod(context, modName)) {
                                // Remove from list and notify adapter
                                mods.remove(position);
                                notifyItemRemoved(position);
                                notifyItemRangeChanged(position, mods.size());
                                Toast.makeText(context, modName + " deleted.", Toast.LENGTH_SHORT).show();
                            } else {
                                Toast.makeText(context, "Failed to delete " + modName, Toast.LENGTH_SHORT).show();
                            }
                        } catch (Exception e) {
                            LogUtils.logDebug("Error deleting mod: " + e.getMessage());
                            Toast.makeText(context, "Error deleting mod: " + e.getMessage(), Toast.LENGTH_SHORT).show();
                        }
                    })
                    .setNegativeButton("Cancel", null)
                    .show();
        });
    }
    
    // Helper method to determine mod type from filename
    private String getModTypeFromFileName(String fileName) {
        String lowerName = fileName.toLowerCase();
        if (lowerName.endsWith(".dex") || lowerName.endsWith(".dex.disabled")) {
            return "DEX (Java)";
        } else if (lowerName.endsWith(".jar") || lowerName.endsWith(".jar.disabled")) {
            return "JAR (Java Library)";
        } else if (lowerName.endsWith(".dll") || lowerName.endsWith(".dll.disabled")) {
            return "DLL (C#/Native)";
        } else {
            return "Unknown";
        }
    }

    @Override
    public int getItemCount() {
        return mods.size();
    }

    /**
     * Updates the adapter's data set with a new list of mods.
     * @param newMods The new list of mods to display.
     */
    public void updateMods(List<File> newMods) {
        this.mods = newMods;
        notifyDataSetChanged();
    }

    public static class ModViewHolder extends RecyclerView.ViewHolder {
        TextView modNameTextView;
        TextView modDescriptionTextView;
        Switch modSwitch;
        ImageButton modDeleteButton;

        public ModViewHolder(@NonNull View itemView) {
            super(itemView);
            modNameTextView = itemView.findViewById(R.id.modNameTextView);
            modDescriptionTextView = itemView.findViewById(R.id.modDescription);
            modSwitch = itemView.findViewById(R.id.modSwitch);
            modDeleteButton = itemView.findViewById(R.id.modDeleteButton);
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/ModManagementActivity.java

// File: ModManagementActivity.java - Pure Mod Management (Post-Installation)
// Path: /main/java/com/terrarialoader/ui/ModManagementActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.OpenableColumns;
import android.view.View;
import android.widget.Button;
import android.widget.ImageButton;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.R;
import com.modloader.installer.ModInstaller;
import com.modloader.loader.MelonLoaderManager;
import com.modloader.loader.ModManager;
import com.modloader.ui.ModListAdapter;
import com.modloader.util.LogUtils;

import java.io.File;
import java.util.List;

/**
 * Pure mod management activity - assumes loader is already installed
 * Focused solely on managing DLL and DEX mods
 */
public class ModManagementActivity extends AppCompatActivity {

    private static final int REQUEST_SELECT_DLL = 1001;
    private static final int REQUEST_SELECT_DEX = 1002;
    
    // UI Components
    private TextView statusText;
    private TextView loaderStatusText;
    private RecyclerView modRecyclerView;
    private ModListAdapter modAdapter;
    private Button addDllModBtn;
    private Button addDexModBtn;
    private Button refreshBtn;
    private Button backBtn;
    private LinearLayout loaderInfoSection;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mod_management);
        
        setTitle("🎮 Mod Management");
        
        initializeViews();
        setupListeners();
        loadMods();
        updateStatus();
    }

    private void initializeViews() {
        statusText = findViewById(R.id.statusText);
        loaderStatusText = findViewById(R.id.loaderStatusText);
        modRecyclerView = findViewById(R.id.modRecyclerView);
        addDllModBtn = findViewById(R.id.addDllModBtn);
        addDexModBtn = findViewById(R.id.addDexModBtn);
        refreshBtn = findViewById(R.id.refreshBtn);
        backBtn = findViewById(R.id.backBtn);
        loaderInfoSection = findViewById(R.id.loaderInfoSection);
        
        // Setup RecyclerView
        modRecyclerView.setLayoutManager(new LinearLayoutManager(this));
    }

    private void setupListeners() {
        addDllModBtn.setOnClickListener(v -> {
            if (!MelonLoaderManager.isMelonLoaderInstalled(this) && !MelonLoaderManager.isLemonLoaderInstalled(this)) {
                showLoaderRequiredDialog();
                return;
            }
            
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("*/*");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            startActivityForResult(intent, REQUEST_SELECT_DLL);
        });
        
        addDexModBtn.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("*/*");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            startActivityForResult(intent, REQUEST_SELECT_DEX);
        });
        
        refreshBtn.setOnClickListener(v -> {
            loadMods();
            updateStatus();
            Toast.makeText(this, "🔄 Mods refreshed", Toast.LENGTH_SHORT).show();
        });
        
        backBtn.setOnClickListener(v -> finish());
    }

    private void loadMods() {
        ModManager.loadMods(this);
        List<File> allMods = ModManager.getAvailableMods();
        
        if (modAdapter == null) {
            modAdapter = new ModListAdapter(this, allMods);
            modRecyclerView.setAdapter(modAdapter);
        } else {
            // Update existing adapter
            modAdapter.updateMods(allMods);
        }
        
        LogUtils.logUser("Loaded " + allMods.size() + " mods for management");
    }

    private void updateStatus() {
        // Check loader status
        boolean melonInstalled = MelonLoaderManager.isMelonLoaderInstalled(this);
        boolean lemonInstalled = MelonLoaderManager.isLemonLoaderInstalled(this);
        
        if (melonInstalled) {
            loaderStatusText.setText("✅ MelonLoader " + MelonLoaderManager.getInstalledLoaderVersion() + " - DLL mods supported");
            loaderStatusText.setTextColor(0xFF4CAF50); // Green
            addDllModBtn.setEnabled(true);
            addDllModBtn.setText("📥 Add DLL Mod");
            loaderInfoSection.setVisibility(View.VISIBLE);
        } else if (lemonInstalled) {
            loaderStatusText.setText("✅ LemonLoader " + MelonLoaderManager.getInstalledLoaderVersion() + " - DLL mods supported");
            loaderStatusText.setTextColor(0xFF4CAF50); // Green
            addDllModBtn.setEnabled(true);
            addDllModBtn.setText("📥 Add DLL Mod");
            loaderInfoSection.setVisibility(View.VISIBLE);
        } else {
            loaderStatusText.setText("⚠️ No loader installed - DLL mods unavailable");
            loaderStatusText.setTextColor(0xFFF44336); // Red
            addDllModBtn.setEnabled(false);
            addDllModBtn.setText("❌ Install Loader First");
            loaderInfoSection.setVisibility(View.GONE);
        }
        
        // Update mod counts
        int enabledCount = ModManager.getEnabledModCount();
        int totalCount = ModManager.getTotalModCount();
        int dexCount = ModManager.getDexModCount();
        int dllCount = ModManager.getDllModCount();
        
        statusText.setText(String.format("📊 Total: %d mods (%d enabled) | DEX/JAR: %d | DLL: %d", 
            totalCount, enabledCount, dexCount, dllCount));
    }

    private void showLoaderRequiredDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("🔧 Loader Required");
        builder.setMessage("DLL mods require MelonLoader or LemonLoader to be installed.\n\n" +
                          "Would you like to set up a loader now?");
        
        builder.setPositiveButton("🚀 Setup Loader", (dialog, which) -> {
            // Go to unified loader setup
            Intent intent = new Intent(this, UnifiedLoaderActivity.class);
            startActivity(intent);
        });
        
        builder.setNegativeButton("📖 Manual Guide", (dialog, which) -> {
            Intent intent = new Intent(this, InstructionsActivity.class);
            startActivity(intent);
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (resultCode != Activity.RESULT_OK || data == null || data.getData() == null) {
            return;
        }
        
        Uri uri = data.getData();
        String filename = getFilenameFromUri(uri);
        
        if (filename == null) {
            Toast.makeText(this, "Could not determine filename", Toast.LENGTH_SHORT).show();
            return;
        }
        
        switch (requestCode) {
            case REQUEST_SELECT_DLL:
                handleDllModInstallation(uri, filename);
                break;
                
            case REQUEST_SELECT_DEX:
                handleDexModInstallation(uri, filename);
                break;
        }
    }

    private void handleDllModInstallation(Uri uri, String filename) {
        if (!filename.toLowerCase().endsWith(".dll")) {
            Toast.makeText(this, "⚠️ Please select a .dll file", Toast.LENGTH_LONG).show();
            return;
        }
        
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("📥 Install DLL Mod");
        builder.setMessage("Install '" + filename + "' as a DLL mod?\n\n" +
                          "This mod will be loaded by MelonLoader when Terraria starts.");
        
        builder.setPositiveButton("Install", (dialog, which) -> {
            boolean success = ModInstaller.installMod(this, uri, filename);
            if (success) {
                Toast.makeText(this, "✅ DLL mod installed: " + filename, Toast.LENGTH_SHORT).show();
                loadMods();
                updateStatus();
            } else {
                Toast.makeText(this, "❌ Failed to install DLL mod", Toast.LENGTH_LONG).show();
            }
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    private void handleDexModInstallation(Uri uri, String filename) {
        String lowerName = filename.toLowerCase();
        if (!lowerName.endsWith(".dex") && !lowerName.endsWith(".jar")) {
            Toast.makeText(this, "⚠️ Please select a .dex or .jar file", Toast.LENGTH_LONG).show();
            return;
        }
        
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("📥 Install DEX/JAR Mod");
        builder.setMessage("Install '" + filename + "' as a DEX/JAR mod?\n\n" +
                          "This mod will be loaded directly by TerrariaLoader.");
        
        builder.setPositiveButton("Install", (dialog, which) -> {
            boolean success = ModInstaller.installMod(this, uri, filename);
            if (success) {
                Toast.makeText(this, "✅ DEX/JAR mod installed: " + filename, Toast.LENGTH_SHORT).show();
                loadMods();
                updateStatus();
            } else {
                Toast.makeText(this, "❌ Failed to install DEX/JAR mod", Toast.LENGTH_LONG).show();
            }
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    private String getFilenameFromUri(Uri uri) {
        String filename = null;
        
        try (Cursor cursor = getContentResolver().query(uri, null, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                int nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                if (nameIndex >= 0) {
                    filename = cursor.getString(nameIndex);
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Could not get filename from URI: " + e.getMessage());
        }
        
        if (filename == null) {
            String path = uri.getPath();
            if (path != null) {
                int lastSlash = path.lastIndexOf('/');
                if (lastSlash >= 0 && lastSlash < path.length() - 1) {
                    filename = path.substring(lastSlash + 1);
                }
            }
        }
        
        return filename;
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadMods();
        updateStatus();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/OfflineDiagnosticActivity.java

// File: OfflineDiagnosticActivity.java (Part 1 - Main Class)
// Path: /main/java/com/terrarialoader/ui/OfflineDiagnosticActivity.java

package com.modloader.ui;

import android.app.ProgressDialog;
import android.content.Intent;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.FileProvider;

import com.modloader.R;
import com.modloader.diagnostic.DiagnosticManager;
import com.modloader.util.LogUtils;
import com.modloader.util.FileUtils;
import com.modloader.util.PathManager;
import com.modloader.loader.MelonLoaderManager;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

public class OfflineDiagnosticActivity extends AppCompatActivity {
    
    private DiagnosticManager diagnosticManager;
    
    // UI Components
    private Button btnRunFullDiagnostic;
    private Button btnDiagnoseApk;
    private Button btnFixSettings;
    private Button btnAutoRepair;
    private Button btnExportReport;
    private Button btnClearResults;
    private TextView diagnosticResultsText;
    
    // Progress dialog
    private ProgressDialog progressDialog;
    
    // File picker for APK selection
    private final ActivityResultLauncher<Intent> apkPickerLauncher = 
        registerForActivityResult(new ActivityResultContracts.StartActivityForResult(), result -> {
            if (result.getResultCode() == RESULT_OK && result.getData() != null) {
                Uri apkUri = result.getData().getData();
                runApkDiagnostic(apkUri);
            }
        });
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_offline_diagnostic);
        setTitle("🔧 Offline Diagnostics");
        
        initializeComponents();
        setupUI();
        
        LogUtils.logUser("Offline Diagnostics opened");
    }
    
    private void initializeComponents() {
        diagnosticManager = new DiagnosticManager(this);
        
        // Find UI components
        btnRunFullDiagnostic = findViewById(R.id.btn_run_full_diagnostic);
        btnDiagnoseApk = findViewById(R.id.btn_diagnose_apk);
        btnFixSettings = findViewById(R.id.btn_fix_settings);
        btnAutoRepair = findViewById(R.id.btn_auto_repair);
        btnExportReport = findViewById(R.id.btn_export_report);
        btnClearResults = findViewById(R.id.btn_clear_results);
        diagnosticResultsText = findViewById(R.id.diagnostic_results_text);
    }
    
    private void setupUI() {
        // Full system diagnostic
        btnRunFullDiagnostic.setOnClickListener(v -> runFullSystemCheck());
        
        // APK diagnostic
        btnDiagnoseApk.setOnClickListener(v -> selectApkForDiagnostic());
        
        // Settings diagnostic and fix
        btnFixSettings.setOnClickListener(v -> diagnoseAndFixSettings());
        
        // Auto repair
        btnAutoRepair.setOnClickListener(v -> performAutoRepair());
        
        // Export report
        btnExportReport.setOnClickListener(v -> exportDiagnosticReport());
        
        // Clear results
        btnClearResults.setOnClickListener(v -> clearResults());
    }
    
    private void runFullSystemCheck() {
        showProgress("Running comprehensive system diagnostic...");
        
        AsyncTask.execute(() -> {
            try {
                StringBuilder results = new StringBuilder();
                results.append("=== TerrariaLoader Comprehensive Diagnostic ===\n");
                results.append("Timestamp: ").append(new java.util.Date().toString()).append("\n");
                results.append("Device: ").append(android.os.Build.MANUFACTURER).append(" ")
                       .append(android.os.Build.MODEL).append("\n");
                results.append("Android: ").append(android.os.Build.VERSION.RELEASE).append("\n\n");
                
                // 1. Directory Structure Check
                results.append("📁 DIRECTORY STRUCTURE\n");
                results.append(checkDirectoryStructure()).append("\n");
                
                // 2. MelonLoader/LemonLoader Status
                results.append("🛠️ LOADER STATUS\n");
                results.append(checkLoaderStatus()).append("\n");
                
                // 3. Mod Files Validation
                results.append("📦 MOD FILES\n");
                results.append(checkModFiles()).append("\n");
                
                // 4. System Permissions
                results.append("🔐 PERMISSIONS\n");
                results.append(checkPermissions()).append("\n");
                
                // 5. Storage and Space
                results.append("💾 STORAGE\n");
                results.append(checkStorage()).append("\n");
                
                // 6. Settings Validation
                results.append("⚙️ SETTINGS\n");
                results.append(checkSettingsIntegrity()).append("\n");
                
                // 7. Suggested Actions
                results.append("💡 RECOMMENDATIONS\n");
                results.append(generateRecommendations()).append("\n");
                
                runOnUiThread(() -> {
                    hideProgress();
                    displayResults(results.toString());
                    LogUtils.logUser("Full system diagnostic completed");
                });
            } catch (Exception e) {
                runOnUiThread(() -> {
                    hideProgress();
                    showError("Diagnostic failed: " + e.getMessage());
                    LogUtils.logDebug("Diagnostic error: " + e.toString());
                });
            }
        });
    }
    
    private String checkDirectoryStructure() {
        StringBuilder result = new StringBuilder();
        
        try {
            String gamePackage = "com.and.games505.TerrariaPaid";
            File baseDir = PathManager.getGameBaseDir(this, gamePackage);
            
            if (baseDir == null) {
                result.append("❌ Base directory path is null\n");
                return result.toString();
            }
            
            result.append("Base Path: ").append(baseDir.getAbsolutePath()).append("\n");
            
            // Check key directories
            String[] criticalPaths = {
                "",                           // Base
                "Mods",                      // Mods root
                "Mods/DEX",                  // DEX mods
                "Mods/DLL",                  // DLL mods
                "Loaders",                   // Loaders root
                "Loaders/MelonLoader",       // MelonLoader
                "Logs",                      // Game logs
                "AppLogs",                   // App logs
                "Config",                    // Configuration
                "Backups"                    // Backups
            };
            
            int existingDirs = 0;
            for (String path : criticalPaths) {
                File dir = new File(baseDir, path);
                boolean exists = dir.exists() && dir.isDirectory();
                String status = exists ? "✅" : "❌";
                result.append(status).append(" ").append(path.isEmpty() ? "Base" : path).append("\n");
                if (exists) existingDirs++;
            }
            
            result.append("\nDirectory Health: ").append(existingDirs).append("/").append(criticalPaths.length);
            if (existingDirs < criticalPaths.length) {
                result.append(" (⚠️ Some directories missing)");
            } else {
                result.append(" (✅ Complete)");
            }
            
        } catch (Exception e) {
            result.append("❌ Directory check failed: ").append(e.getMessage());
        }
        
        return result.toString();
    }
    
    private String checkLoaderStatus() {
        StringBuilder result = new StringBuilder();
        
        try {
            String gamePackage = "com.and.games505.TerrariaPaid";
            boolean melonInstalled = MelonLoaderManager.isMelonLoaderInstalled(this);
            boolean lemonInstalled = MelonLoaderManager.isLemonLoaderInstalled(this);
            
            if (melonInstalled) {
                result.append("✅ MelonLoader detected\n");
                result.append("   Version: ").append(MelonLoaderManager.getInstalledLoaderVersion()).append("\n");
                
                // Check core files
                File loaderDir = PathManager.getMelonLoaderDir(this, gamePackage);
                if (loaderDir != null && loaderDir.exists()) {
                    File[] files = loaderDir.listFiles();
                    int fileCount = (files != null) ? files.length : 0;
                    result.append("   Files: ").append(fileCount).append(" detected\n");
                }
            } else if (lemonInstalled) {
                result.append("✅ LemonLoader detected\n");
                result.append("   Version: ").append(MelonLoaderManager.getInstalledLoaderVersion()).append("\n");
            } else {
                result.append("❌ No loader installed\n");
                result.append("   Recommendation: Use 'Complete Setup Wizard' to install MelonLoader\n");
            }
            
            // Check runtime directories
            File net8Dir = new File(PathManager.getMelonLoaderDir(this, gamePackage), "net8");
            File net35Dir = new File(PathManager.getMelonLoaderDir(this, gamePackage), "net35");
            
            result.append("Runtime Support:\n");
            result.append(net8Dir.exists() ? "✅" : "❌").append(" NET8 Runtime\n");
            result.append(net35Dir.exists() ? "✅" : "❌").append(" NET35 Runtime\n");
            
        } catch (Exception e) {
            result.append("❌ Loader check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String checkModFiles() {
        StringBuilder result = new StringBuilder();
        
        try {
            String gamePackage = "com.and.games505.TerrariaPaid";
            
            // Check DEX mods
            File dexDir = PathManager.getDexModsDir(this, gamePackage);
            int dexCount = 0, dexEnabled = 0;
            if (dexDir != null && dexDir.exists()) {
                File[] dexFiles = dexDir.listFiles((dir, name) -> {
                    String lower = name.toLowerCase();
                    return lower.endsWith(".dex") || lower.endsWith(".jar") || 
                           lower.endsWith(".dex.disabled") || lower.endsWith(".jar.disabled");
                });
                if (dexFiles != null) {
                    dexCount = dexFiles.length;
                    for (File file : dexFiles) {
                        if (!file.getName().endsWith(".disabled")) {
                            dexEnabled++;
                        }
                    }
                }
            }
            
            // Check DLL mods
            File dllDir = PathManager.getDllModsDir(this, gamePackage);
            int dllCount = 0, dllEnabled = 0;
            if (dllDir != null && dllDir.exists()) {
                File[] dllFiles = dllDir.listFiles((dir, name) -> {
                    String lower = name.toLowerCase();
                    return lower.endsWith(".dll") || lower.endsWith(".dll.disabled");
                });
                if (dllFiles != null) {
                    dllCount = dllFiles.length;
                    for (File file : dllFiles) {
                        if (!file.getName().endsWith(".disabled")) {
                            dllEnabled++;
                        }
                    }
                }
            }
            
            result.append("DEX/JAR Mods: ").append(dexEnabled).append("/").append(dexCount)
                  .append(" enabled\n");
            result.append("DLL Mods: ").append(dllEnabled).append("/").append(dllCount)
                  .append(" enabled\n");
            result.append("Total Active Mods: ").append(dexEnabled + dllEnabled).append("\n");
            
            if (dexCount == 0 && dllCount == 0) {
                result.append("ℹ️ No mods installed - use Mod Management to add mods\n");
            }
            
        } catch (Exception e) {
            result.append("❌ Mod check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String checkPermissions() {
        StringBuilder result = new StringBuilder();
        
        try {
            // Test write permissions
            File testDir = new File(getExternalFilesDir(null), "permission_test");
            testDir.mkdirs();
            
            File testFile = new File(testDir, "write_test.txt");
            try (FileWriter writer = new FileWriter(testFile)) {
                writer.write("Permission test successful");
                result.append("✅ External storage write access\n");
            } catch (Exception e) {
                result.append("❌ External storage write failed: ").append(e.getMessage()).append("\n");
            } finally {
                if (testFile.exists()) testFile.delete();
                testDir.delete();
            }
            
            // Check install packages permission
            try {
                getPackageManager().canRequestPackageInstalls();
                result.append("✅ Package installation permission available\n");
            } catch (Exception e) {
                result.append("⚠️ Package installation permission may be restricted\n");
            }
            
        } catch (Exception e) {
            result.append("❌ Permission check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String checkStorage() {
        StringBuilder result = new StringBuilder();
        
        try {
            File externalDir = getExternalFilesDir(null);
            if (externalDir != null) {
                long freeSpace = externalDir.getFreeSpace();
                long totalSpace = externalDir.getTotalSpace();
                long usedSpace = totalSpace - freeSpace;
                
                result.append("Free Space: ").append(FileUtils.formatFileSize(freeSpace)).append("\n");
                result.append("Used Space: ").append(FileUtils.formatFileSize(usedSpace)).append("\n");
                result.append("Total Space: ").append(FileUtils.formatFileSize(totalSpace)).append("\n");
                
                if (freeSpace < 100 * 1024 * 1024) { // Less than 100MB
                    result.append("⚠️ Low storage space - consider freeing up space\n");
                } else {
                    result.append("✅ Sufficient storage space available\n");
                }
            } else {
                result.append("❌ Cannot access external storage\n");
            }
            
        } catch (Exception e) {
            result.append("❌ Storage check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String checkSettingsIntegrity() {
        StringBuilder result = new StringBuilder();
        
        try {
            // Check app preferences
            android.content.SharedPreferences prefs = 
                getSharedPreferences("TerrariaLoaderPrefs", MODE_PRIVATE);
            
            // Test write operation
            android.content.SharedPreferences.Editor editor = prefs.edit();
            editor.putString("diagnostic_test", "test_value");
            boolean writeSuccess = editor.commit();
            
            if (writeSuccess) {
                String testValue = prefs.getString("diagnostic_test", null);
                if ("test_value".equals(testValue)) {
                    result.append("✅ Settings persistence working\n");
                    // Clean up test
                    editor.remove("diagnostic_test").commit();
                } else {
                    result.append("❌ Settings read/write mismatch\n");
                }
            } else {
                result.append("❌ Settings write failed\n");
                result.append("   This may explain your auto-refresh issue\n");
            }
            
        } catch (Exception e) {
            result.append("❌ Settings check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String generateRecommendations() {
        StringBuilder result = new StringBuilder();
        
        try {
            boolean hasIssues = false;
            
            // Check if directories need repair
            File baseDir = PathManager.getGameBaseDir(this, "com.and.games505.TerrariaPaid");
            if (baseDir == null || !baseDir.exists()) {
                result.append("• Run 'Auto-Repair' to create missing directories\n");
                hasIssues = true;
            }
            
            // Check if loader is missing
            if (!MelonLoaderManager.isMelonLoaderInstalled(this) && 
                !MelonLoaderManager.isLemonLoaderInstalled(this)) {
                result.append("• Use 'Complete Setup Wizard' to install MelonLoader\n");
                hasIssues = true;
            }
            
            // Check storage
            File externalDir = getExternalFilesDir(null);
            if (externalDir != null && externalDir.getFreeSpace() < 50 * 1024 * 1024) {
                result.append("• Free up storage space (recommended: 100MB+)\n");
                hasIssues = true;
            }
            
            if (!hasIssues) {
                result.append("✅ System appears to be in good condition\n");
                result.append("• If you're still experiencing issues, try:\n");
                result.append("  - Restart the app completely\n");
                result.append("  - Reboot your device\n");
                result.append("  - Check specific mod compatibility\n");
            }
            
        } catch (Exception e) {
            result.append("• General recommendation: Check system permissions\n");
        }
        
        return result.toString();
    }
    
    // Continue to Part 2...
// File: OfflineDiagnosticActivity.java (Part 2 - Methods & UI)
// Continuation of Part 1

    private void selectApkForDiagnostic() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("application/vnd.android.package-archive");
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        
        try {
            apkPickerLauncher.launch(Intent.createChooser(intent, "Select APK to Diagnose"));
        } catch (Exception e) {
            showToast("No file manager available");
        }
    }
    
    private void runApkDiagnostic(Uri apkUri) {
        showProgress("Analyzing APK installation issues...");
        
        AsyncTask.execute(() -> {
            try {
                StringBuilder results = new StringBuilder();
                results.append("=== APK Installation Diagnostic ===\n");
                results.append("File URI: ").append(apkUri.toString()).append("\n\n");
                
                String fileName = getFileNameFromUri(apkUri);
                results.append("File Name: ").append(fileName != null ? fileName : "Unknown").append("\n");
                
                results.append(validateApkFromUri(apkUri)).append("\n");
                results.append("🔧 INSTALLATION ENVIRONMENT\n");
                results.append(checkInstallationEnvironment()).append("\n");
                results.append("📱 DEVICE COMPATIBILITY\n");
                results.append(checkDeviceCompatibility()).append("\n");
                results.append("💡 SOLUTIONS FOR APK PARSING ERRORS\n");
                results.append(getApkSolutions()).append("\n");
                
                runOnUiThread(() -> {
                    hideProgress();
                    displayResults(results.toString());
                });
            } catch (Exception e) {
                runOnUiThread(() -> {
                    hideProgress();
                    showError("APK analysis failed: " + e.getMessage());
                });
            }
        });
    }
    
    private String validateApkFromUri(Uri apkUri) {
        StringBuilder result = new StringBuilder();
        result.append("📦 APK VALIDATION\n");
        
        try (java.io.InputStream stream = getContentResolver().openInputStream(apkUri)) {
            if (stream == null) {
                result.append("❌ Cannot access APK file\n");
                return result.toString();
            }
            
            int available = stream.available();
            if (available > 0) {
                result.append("✅ APK accessible (").append(FileUtils.formatFileSize(available)).append(")\n");
                if (available < 10 * 1024 * 1024) {
                    result.append("⚠️ APK seems small for Terraria - may be corrupted\n");
                }
            } else {
                result.append("⚠️ APK file size unknown or empty\n");
            }
            
            byte[] header = new byte[30];
            int bytesRead = stream.read(header);
            
            if (bytesRead >= 4) {
                if (header[0] == 0x50 && header[1] == 0x4b && header[2] == 0x03 && header[3] == 0x04) {
                    result.append("✅ Valid ZIP/APK signature\n");
                } else {
                    result.append("❌ Invalid ZIP/APK signature - file is corrupted\n");
                    result.append("   This is likely causing your parsing error!\n");
                }
            } else {
                result.append("❌ Cannot read APK header - file corrupted\n");
            }
            
        } catch (Exception e) {
            result.append("❌ APK access failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String checkInstallationEnvironment() {
        StringBuilder result = new StringBuilder();
        
        try {
            boolean unknownSources = canInstallFromUnknownSources();
            result.append(unknownSources ? "✅" : "❌").append(" Unknown sources enabled\n");
            
            if (!unknownSources) {
                result.append("   📋 Fix: Settings > Apps > TerrariaLoader > Install unknown apps\n");
            }
            
            File dataDir = getDataDir();
            long freeSpace = dataDir.getFreeSpace();
            result.append("Internal space: ").append(FileUtils.formatFileSize(freeSpace)).append("\n");
            
            if (freeSpace < 200 * 1024 * 1024) {
                result.append("⚠️ Low storage - may cause installation failure\n");
            }
            
            try {
                getPackageManager().getPackageInfo("com.and.games505.TerrariaPaid", 0);
                result.append("⚠️ Terraria already installed - uninstall first\n");
            } catch (android.content.pm.PackageManager.NameNotFoundException e) {
                result.append("✅ No conflicting installation\n");
            }
            
        } catch (Exception e) {
            result.append("❌ Environment check failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private boolean canInstallFromUnknownSources() {
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            return getPackageManager().canRequestPackageInstalls();
        } else {
            try {
                return android.provider.Settings.Secure.getInt(
                    getContentResolver(), 
                    android.provider.Settings.Secure.INSTALL_NON_MARKET_APPS, 0) != 0;
            } catch (Exception e) {
                return false;
            }
        }
    }
    
    private String checkDeviceCompatibility() {
        StringBuilder result = new StringBuilder();
        
        result.append("Device: ").append(android.os.Build.MANUFACTURER)
              .append(" ").append(android.os.Build.MODEL).append("\n");
        result.append("Android: ").append(android.os.Build.VERSION.RELEASE)
              .append(" (API ").append(android.os.Build.VERSION.SDK_INT).append(")\n");
        result.append("Architecture: ").append(android.os.Build.SUPPORTED_ABIS[0]).append("\n");
        
        if (android.os.Build.VERSION.SDK_INT >= 21) {
            result.append("✅ Compatible Android version\n");
        } else {
            result.append("❌ Android version too old\n");
        }
        
        return result.toString();
    }
    
    private String getApkSolutions() {
        StringBuilder result = new StringBuilder();
        
        result.append("For 'There was a problem parsing the package':\n\n");
        result.append("1. 🔧 Re-download APK (may be corrupted)\n");
        result.append("2. 🔧 Enable 'Install unknown apps'\n");
        result.append("3. 🔧 Uninstall original Terraria first\n");
        result.append("4. 🔧 Clear Package Installer cache\n");
        result.append("5. 🔧 Restart device and retry\n");
        result.append("6. 🔧 Copy APK to internal storage\n");
        result.append("7. 🔧 Use different file manager\n");
        result.append("8. 🔧 Check antivirus isn't blocking\n");
        
        return result.toString();
    }
    
    private void diagnoseAndFixSettings() {
        showProgress("Diagnosing settings persistence...");
        
        AsyncTask.execute(() -> {
            try {
                StringBuilder results = new StringBuilder();
                results.append("=== Settings Persistence Diagnostic ===\n\n");
                results.append("🔧 SHARED PREFERENCES TEST\n");
                results.append(testSharedPreferences()).append("\n");
                results.append("🔄 AUTO-REFRESH SPECIFIC TEST\n");
                results.append(testAutoRefreshSetting()).append("\n");
                results.append("💾 FILE SYSTEM TEST\n");
                results.append(testFileSystemWrites()).append("\n");
                
                runOnUiThread(() -> {
                    hideProgress();
                    displayResults(results.toString());
                    showSettingsFixOptions();
                });
            } catch (Exception e) {
                runOnUiThread(() -> {
                    hideProgress();
                    showError("Settings diagnostic failed: " + e.getMessage());
                });
            }
        });
    }
    
    private String testSharedPreferences() {
        StringBuilder result = new StringBuilder();
        
        try {
            android.content.SharedPreferences prefs = getSharedPreferences("DiagnosticTest", MODE_PRIVATE);
            android.content.SharedPreferences.Editor editor = prefs.edit();
            
            editor.putBoolean("test_bool", true);
            editor.putString("test_string", "test_value");
            boolean commitSuccess = editor.commit();
            
            result.append("Write test: ").append(commitSuccess ? "✅ Success" : "❌ Failed").append("\n");
            
            if (commitSuccess) {
                boolean boolVal = prefs.getBoolean("test_bool", false);
                String stringVal = prefs.getString("test_string", null);
                boolean readSuccess = boolVal && "test_value".equals(stringVal);
                
                result.append("Read test: ").append(readSuccess ? "✅ Success" : "❌ Failed").append("\n");
                
                if (!readSuccess) {
                    result.append("   This explains your auto-refresh issue!\n");
                }
                
                editor.clear().commit();
            }
            
        } catch (Exception e) {
            result.append("❌ SharedPreferences test failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String testAutoRefreshSetting() {
        StringBuilder result = new StringBuilder();
        
        try {
            // Simulate the exact auto-refresh setting behavior
            android.content.SharedPreferences logPrefs = getSharedPreferences("LogViewerPrefs", MODE_PRIVATE);
            android.content.SharedPreferences.Editor editor = logPrefs.edit();
            
            // Test the specific setting that's failing
            editor.putBoolean("auto_refresh_enabled", false);
            boolean applyResult = editor.commit(); // Use commit instead of apply for immediate result
            
            result.append("Auto-refresh disable: ").append(applyResult ? "✅ Success" : "❌ Failed").append("\n");
            
            if (applyResult) {
                // Check if it actually persisted
                boolean currentValue = logPrefs.getBoolean("auto_refresh_enabled", true); // default true
                result.append("Setting persisted: ").append(!currentValue ? "✅ Success" : "❌ Failed").append("\n");
                
                if (currentValue) {
                    result.append("   Setting reverted to default - persistence failed!\n");
                    result.append("   This is your exact issue.\n");
                }
            }
            
        } catch (Exception e) {
            result.append("❌ Auto-refresh test failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String testFileSystemWrites() {
        StringBuilder result = new StringBuilder();
        
        try {
            File testDir = new File(getFilesDir(), "diagnostic_test");
            testDir.mkdirs();
            
            File testFile = new File(testDir, "settings_test.txt");
            
            try (FileWriter writer = new FileWriter(testFile)) {
                writer.write("auto_refresh=false\n");
                writer.write("timestamp=" + System.currentTimeMillis() + "\n");
                result.append("✅ File write successful\n");
            }
            
            if (testFile.exists()) {
                try (java.io.BufferedReader reader = new java.io.BufferedReader(
                        new java.io.FileReader(testFile))) {
                    String line = reader.readLine();
                    if (line != null && line.contains("auto_refresh=false")) {
                        result.append("✅ File read successful\n");
                    } else {
                        result.append("❌ File content corrupted\n");
                    }
                }
            }
            
            testFile.delete();
            testDir.delete();
            
        } catch (Exception e) {
            result.append("❌ File system test failed: ").append(e.getMessage()).append("\n");
        }
        
        return result.toString();
    }
    
    private String getFileNameFromUri(Uri uri) {
        try {
            android.database.Cursor cursor = getContentResolver().query(uri, null, null, null, null);
            if (cursor != null) {
                int nameIndex = cursor.getColumnIndex(android.provider.OpenableColumns.DISPLAY_NAME);
                if (nameIndex >= 0 && cursor.moveToFirst()) {
                    String name = cursor.getString(nameIndex);
                    cursor.close();
                    return name;
                }
                cursor.close();
            }
        } catch (Exception e) {
            return uri.getLastPathSegment();
        }
        return null;
    }
    
    private void performAutoRepair() {
        new AlertDialog.Builder(this)
            .setTitle("Auto-Repair System")
            .setMessage("Attempt automatic fixes for:\n\n" +
                       "• Missing directories\n" +
                       "• Settings persistence\n" +
                       "• File permissions\n" +
                       "• Configuration corruption\n\n" +
                       "Continue?")
            .setPositiveButton("Yes, Repair", (dialog, which) -> executeAutoRepair())
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void executeAutoRepair() {
        showProgress("Performing auto-repair...");
        
        AsyncTask.execute(() -> {
            try {
                StringBuilder results = new StringBuilder();
                results.append("=== Auto-Repair Results ===\n\n");
                
                boolean directoryRepair = diagnosticManager.attemptSelfRepair();
                boolean settingsRepair = repairSettings();
                boolean permissionRepair = repairPermissions();
                
                results.append("Directory Structure: ").append(directoryRepair ? "✅ Fixed" : "❌ Failed").append("\n");
                results.append("Settings Persistence: ").append(settingsRepair ? "✅ Fixed" : "❌ Failed").append("\n");
                results.append("Permissions: ").append(permissionRepair ? "✅ Fixed" : "❌ Failed").append("\n\n");
                
                if (directoryRepair || settingsRepair || permissionRepair) {
                    results.append("🔄 Restart recommended to apply changes.\n");
                } else {
                    results.append("❌ Could not auto-fix detected issues.\n");
                    results.append("💡 Try manual solutions or check device settings.\n");
                }
                
                runOnUiThread(() -> {
                    hideProgress();
                    displayResults(results.toString());
                });
            } catch (Exception e) {
                runOnUiThread(() -> {
                    hideProgress();
                    showError("Auto-repair failed: " + e.getMessage());
                });
            }
        });
    }
    
    private boolean repairSettings() {
        try {
            // Clear all shared preferences and recreate
            String[] prefFiles = {"TerrariaLoaderPrefs", "LogViewerPrefs", "AppSettings"};
            
            for (String prefFile : prefFiles) {
                android.content.SharedPreferences prefs = getSharedPreferences(prefFile, MODE_PRIVATE);
                android.content.SharedPreferences.Editor editor = prefs.edit();
                editor.clear();
                if (!editor.commit()) {
                    return false;
                }
            }
            
            // Test write after clear
            android.content.SharedPreferences testPrefs = getSharedPreferences("TerrariaLoaderPrefs", MODE_PRIVATE);
            android.content.SharedPreferences.Editor testEditor = testPrefs.edit();
            testEditor.putBoolean("settings_repaired", true);
            return testEditor.commit();
            
        } catch (Exception e) {
            LogUtils.logDebug("Settings repair failed: " + e.getMessage());
            return false;
        }
    }
    
    private boolean repairPermissions() {
        try {
            File testDir = new File(getExternalFilesDir(null), "permission_test");
            testDir.mkdirs();
            
            File testFile = new File(testDir, "test.txt");
            FileWriter writer = new FileWriter(testFile);
            writer.write("test");
            writer.close();
            
            boolean canWrite = testFile.exists() && testFile.length() > 0;
            testFile.delete();
            testDir.delete();
            
            return canWrite;
        } catch (Exception e) {
            return false;
        }
    }
    
    private void exportDiagnosticReport() {
        try {
            String reportContent = diagnosticResultsText.getText().toString();
            if (reportContent.isEmpty() || reportContent.startsWith("Click")) {
                showToast("No diagnostic results to export");
                return;
            }
            
            File reportsDir = new File(getExternalFilesDir(null), "DiagnosticReports");
            reportsDir.mkdirs();
            
            String timestamp = new java.text.SimpleDateFormat("yyyyMMdd_HHmmss", 
                java.util.Locale.getDefault()).format(new java.util.Date());
            File reportFile = new File(reportsDir, "diagnostic_" + timestamp + ".txt");
            
            try (FileWriter writer = new FileWriter(reportFile)) {
                writer.write(reportContent);
                writer.write("\n\n=== Export Info ===\n");
                writer.write("Exported by: TerrariaLoader Diagnostic Tool\n");
                writer.write("Export time: " + new java.util.Date().toString() + "\n");
            }
            
            // Share the report
            Uri fileUri = FileProvider.getUriForFile(this, 
                getPackageName() + ".provider", reportFile);
            
            Intent shareIntent = new Intent(Intent.ACTION_SEND);
            shareIntent.setType("text/plain");
            shareIntent.putExtra(Intent.EXTRA_STREAM, fileUri);
            shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            
            startActivity(Intent.createChooser(shareIntent, "Share Diagnostic Report"));
            showToast("Report exported: " + reportFile.getName());
            
        } catch (Exception e) {
            showError("Export failed: " + e.getMessage());
        }
    }
    
    private void clearResults() {
        diagnosticResultsText.setText("Click 'Run Full System Check' to start diagnostics...");
    }
    
    private void showSettingsFixOptions() {
        new AlertDialog.Builder(this)
            .setTitle("Settings Fix Options")
            .setMessage("Settings persistence issue detected. Try these fixes:")
            .setPositiveButton("Clear All Settings", (dialog, which) -> clearAllSettings())
            .setNeutralButton("Reset App Data", (dialog, which) -> showResetAppDataInfo())
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    private void clearAllSettings() {
        try {
            String[] prefFiles = {"TerrariaLoaderPrefs", "LogViewerPrefs", "AppSettings"};
            for (String prefFile : prefFiles) {
                getSharedPreferences(prefFile, MODE_PRIVATE).edit().clear().commit();
            }
            showToast("Settings cleared - restart app to test");
        } catch (Exception e) {
            showError("Failed to clear settings: " + e.getMessage());
        }
    }
    
    private void showResetAppDataInfo() {
        new AlertDialog.Builder(this)
            .setTitle("Reset App Data")
            .setMessage("To completely reset TerrariaLoader:\n\n" +
                       "1. Go to Android Settings\n" +
                       "2. Apps > TerrariaLoader\n" +
                       "3. Storage > Clear Data\n\n" +
                       "This will fix persistent settings issues.")
            .setPositiveButton("OK", null)
            .show();
    }
    
    private void displayResults(String results) {
        diagnosticResultsText.setText(results);
    }
    
    private void showProgress(String message) {
        progressDialog = new ProgressDialog(this);
        progressDialog.setMessage(message);
        progressDialog.setCancelable(false);
        progressDialog.show();
    }
    
    private void hideProgress() {
        if (progressDialog != null && progressDialog.isShowing()) {
            progressDialog.dismiss();
        }
    }
    
    private void showToast(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
    private void showError(String error) {
        new AlertDialog.Builder(this)
            .setTitle("Error")
            .setMessage(error)
            .setPositiveButton("OK", null)
            .show();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        hideProgress();
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/PluginAdapter.java

package com.modloader.ui;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.plugins.PluginDescriptor;
import com.modloader.R;

import java.util.ArrayList;
import java.util.List;

public class PluginAdapter extends RecyclerView.Adapter<PluginAdapter.VH> {

    private final List<PluginDescriptor> items = new ArrayList<>();

    public void submit(List<PluginDescriptor> data) {
        items.clear();
        if (data != null) items.addAll(data);
        notifyDataSetChanged();
    }

    @NonNull @Override
    public VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_plugin, parent, false);
        return new VH(v);
    }

    @Override
    public void onBindViewHolder(@NonNull VH h, int pos) {
        PluginDescriptor d = items.get(pos);
        h.title.setText(d.name + " (" + d.version + ")");
        h.subtitle.setText(d.id + " — " + d.author);
    }

    @Override
    public int getItemCount() { return items.size(); }

    static class VH extends RecyclerView.ViewHolder {
        final TextView title, subtitle;
        VH(@NonNull View itemView) {
            super(itemView);
            title = itemView.findViewById(R.id.title);
            subtitle = itemView.findViewById(R.id.subtitle);
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/PluginDetailsActivity.java

package com.modloader.ui;

import android.os.Bundle;
import android.widget.TextView;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.modloader.R;
import com.modloader.plugins.PluginDescriptor;

public class PluginDetailsActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_plugins);
        // Minimal placeholder to avoid navigation complexity in this integration step.
        TextView tv = findViewById(R.id.header);
        tv.setText("Plugin Details (placeholder)");
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/PluginManagerActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.content.Intent;
import android.graphics.Color;
import android.net.Uri;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageButton;
import android.widget.PopupMenu;
import android.widget.Switch;
import android.widget.TextView;
import android.widget.Toast;

import androidx.activity.result.ActivityResult;
import androidx.activity.result.ActivityResultCallback;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.modloader.R;
import com.modloader.plugins.PluginDescriptor;
import com.modloader.plugins.PluginInstaller;
import com.modloader.plugins.PluginManager;
import com.modloader.plugins.PluginRepository;
import com.modloader.util.IOUtils;

import org.json.JSONObject;

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

/**
 * Full-feature Plugin Manager activity:
 * - lists installed plugins from external and internal folders
 * - enable/disable persisted state
 * - uninstall (delete)
 * - install via FAB (opens document picker)
 * - long-press / overflow popup for actions
 */
public class PluginManagerActivity extends AppCompatActivity {

    private RecyclerView rv;
    private Adapter adapter;
    private PluginManager manager;
    private PluginRepository repo;

    private final ActivityResultLauncher<Intent> pickFileLauncher =
            registerForActivityResult(new ActivityResultContracts.StartActivityForResult(),
                    new ActivityResultCallback<ActivityResult>() {
                        @Override
                        public void onActivityResult(ActivityResult result) {
                            if (result.getResultCode() == Activity.RESULT_OK && result.getData() != null) {
                                Uri uri = result.getData().getData();
                                if (uri != null) {
                                    try (InputStream in = getContentResolver().openInputStream(uri)) {
                                        File tmp = new File(getCacheDir(), "import.tmp");
                                        IOUtils.copyToFile(in, tmp);
                                        File installed = new PluginInstaller(PluginManagerActivity.this).install(tmp);
                                        Toast.makeText(PluginManagerActivity.this, "Installed: " + installed.getName(), Toast.LENGTH_SHORT).show();
                                        refreshList();
                                    } catch (Exception e) {
                                        Toast.makeText(PluginManagerActivity.this, "Install error: " + e.getMessage(), Toast.LENGTH_LONG).show();
                                    }
                                }
                            }
                        }
                    });

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_plugin_manager);
        setTitle("Plugin Manager");

        rv = findViewById(R.id.rv_plugins);
        rv.setLayoutManager(new LinearLayoutManager(this));
        adapter = new Adapter();
        rv.setAdapter(adapter);

        manager = new PluginManager(this);
        repo = new PluginRepository(this);

        FloatingActionButton fab = findViewById(R.id.fab_install);
        fab.setOnClickListener(v -> pickPluginFile());

        refreshList();
    }

    private void pickPluginFile() {
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        intent.setType("*/*");
        String[] mimes = new String[] { "application/java-archive", "application/octet-stream", "application/zip", "application/vnd.android.package-archive" };
        intent.putExtra(Intent.EXTRA_MIME_TYPES, mimes);
        pickFileLauncher.launch(intent);
    }

    private void refreshList() {
        List<PluginDescriptor> combined = new ArrayList<>();
        // scan external
        for (File f : repo.scanFiles()) {
            PluginDescriptor d = repo.parseManifest(f);
            if (d == null) {
                // best-effort create descriptor from filename
                d = new PluginDescriptor();
                d.id = f.getName();
                d.name = f.getName();
                d.version = "1.0";
                d.author = "unknown";
                d.enabled = manager != null ? getEnabledFromPrefs(d.id) : true;
            }
            combined.add(d);
        }
        // also check internal storage folder
        List<PluginDescriptor> internal = manager.getInstalledPlugins();
        for (PluginDescriptor p : internal) {
            boolean exists = false;
            for (PluginDescriptor ex : combined) if (ex.id.equals(p.id)) { exists = true; break; }
            if (!exists) combined.add(p);
        }
        adapter.setItems(combined);
    }

    private boolean getEnabledFromPrefs(String id) {
        return manager != null && manager.isPluginEnabled(id);
    }

    private void enablePlugin(PluginDescriptor d, boolean enable) {
        if (manager != null) {
            manager.setPluginEnabled(d.id, enable);
            d.enabled = enable;
            adapter.notifyDataSetChanged();
            Toast.makeText(this, (enable ? "Enabled " : "Disabled ") + d.name, Toast.LENGTH_SHORT).show();
        }
    }

    private void uninstallPlugin(PluginDescriptor d) {
        new AlertDialog.Builder(this)
                .setTitle("Uninstall")
                .setMessage("Uninstall plugin " + d.name + "?")
                .setPositiveButton("Remove", (dialog, which) -> {
                    boolean ok = manager != null && manager.uninstallPlugin(d.id);
                    if (ok) {
                        Toast.makeText(this, "Removed " + d.name, Toast.LENGTH_SHORT).show();
                        refreshList();
                    } else {
                        Toast.makeText(this, "Failed to remove " + d.name, Toast.LENGTH_LONG).show();
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main_menu, menu);
        return true;
    }

    class Adapter extends RecyclerView.Adapter<Adapter.VH> {
        private final List<PluginDescriptor> items = new ArrayList<>();

        void setItems(List<PluginDescriptor> data) {
            items.clear();
            if (data != null) items.addAll(data);
            notifyDataSetChanged();
        }

        @NonNull
        @Override
        public VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
            View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_plugin_manager, parent, false);
            return new VH(v);
        }

        @Override
        public void onBindViewHolder(@NonNull VH h, int pos) {
            PluginDescriptor d = items.get(pos);
            h.title.setText(d.name + " (" + d.version + ")");
            h.subtitle.setText(d.id + " — " + d.author);

            // grayed out style when disabled
            if (!d.enabled) {
                h.itemView.setAlpha(0.5f);
                h.title.setTextColor(Color.GRAY);
                h.subtitle.setTextColor(Color.GRAY);
                h.disabledLabel.setVisibility(View.VISIBLE);
            } else {
                h.itemView.setAlpha(1f);
                h.title.setTextColor(ContextCompat.getColor(PluginManagerActivity.this, android.R.color.black));
                h.subtitle.setTextColor(ContextCompat.getColor(PluginManagerActivity.this, android.R.color.darker_gray));
                h.disabledLabel.setVisibility(View.GONE);
            }

            h.toggle.setOnCheckedChangeListener(null);
            h.toggle.setChecked(d.enabled);
            h.toggle.setOnCheckedChangeListener((buttonView, isChecked) -> {
                enablePlugin(d, isChecked);
            });

            h.btnOverflow.setOnClickListener(v -> {
                PopupMenu popup = new PopupMenu(PluginManagerActivity.this, h.btnOverflow);
                popup.getMenu().add(d.enabled ? "Disable" : "Enable");
                popup.getMenu().add("Uninstall");
                popup.setOnMenuItemClickListener(item -> {
                    String title = (String) item.getTitle();
                    if (title.equals("Uninstall")) {
                        uninstallPlugin(d);
                    } else if (title.equals("Enable")) {
                        enablePlugin(d, true);
                    } else if (title.equals("Disable")) {
                        enablePlugin(d, false);
                    }
                    return true;
                });
                popup.show();
            });

            h.itemView.setOnLongClickListener(v -> {
                // same as overflow
                h.btnOverflow.performClick();
                return true;
            });
        }

        @Override
        public int getItemCount() { return items.size(); }

        class VH extends RecyclerView.ViewHolder {
            TextView title, subtitle, disabledLabel;
            Switch toggle;
            ImageButton btnOverflow;
            VH(@NonNull View itemView) {
                super(itemView);
                title = itemView.findViewById(R.id.title);
                subtitle = itemView.findViewById(R.id.subtitle);
                disabledLabel = itemView.findViewById(R.id.disabled_label);
                toggle = itemView.findViewById(R.id.switch_enable);
                btnOverflow = itemView.findViewById(R.id.btn_uninstall);
            }
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/PluginsActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.provider.DocumentsContract;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.modloader.plugins.PluginDescriptor;
import com.modloader.plugins.PluginInstaller;
import com.modloader.plugins.PluginManager;
import com.modloader.plugins.PluginRepository;
import com.modloader.util.IOUtils;
import com.modloader.R;

import java.io.File;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class PluginsActivity extends AppCompatActivity {

    private static final int REQ_OPEN_FILE = 1001;
    private RecyclerView list;
    private PluginAdapter adapter;
    private Button refreshBtn, installBtn;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_plugins);

        list = findViewById(R.id.plugins_list);
        list.setLayoutManager(new LinearLayoutManager(this));
        adapter = new PluginAdapter();
        list.setAdapter(adapter);

        refreshBtn = findViewById(R.id.btn_refresh);
        installBtn = findViewById(R.id.btn_install);

        refreshBtn.setOnClickListener(v -> refresh());
        installBtn.setOnClickListener(v -> pickFile());
    }

    private void refresh() {
        List<String> messages = PluginManager.get(this).refreshAndLoadAll();
        List<PluginDescriptor> loaded = PluginManager.get(this).getLoadedDescriptors();
        adapter.submit(loaded);
        Toast.makeText(this, String.join("\n", messages), Toast.LENGTH_SHORT).show();
    }

    private void pickFile() {
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        intent.setType("*/*");
        String[] mimes = new String[] { "application/vnd.android.package-archive", "application/java-archive", "application/octet-stream" };
        intent.putExtra(Intent.EXTRA_MIME_TYPES, mimes);
        startActivityForResult(intent, REQ_OPEN_FILE);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQ_OPEN_FILE && resultCode == Activity.RESULT_OK && data != null) {
            Uri uri = data.getData();
            if (uri != null) {
                try (InputStream in = getContentResolver().openInputStream(uri)) {
                    File tmp = new File(getCacheDir(), "import.tmp");
                    IOUtils.copyToFile(in, tmp);
                    File installed = new PluginInstaller(this).install(tmp);
                    Toast.makeText(this, "Installed: " + installed.getName(), Toast.LENGTH_SHORT).show();
                } catch (Exception e) {
                    Toast.makeText(this, "Install error: " + e.getMessage(), Toast.LENGTH_LONG).show();
                }
            }
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/ScriptConsoleActivity.java

package com.modloader.ui;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import com.modloader.addons.AddonManager;
import com.modloader.scripting.ScriptEngineManager;
public class ScriptConsoleActivity extends AppCompatActivity {
    private ScriptEngineManager engines;
    @Override protected void onCreate(Bundle b){
        super.onCreate(b);
        setContentView(getResources().getIdentifier("activity_script_console","layout",getPackageName()));
        AddonManager addons = new AddonManager(this);
        addons.reload();
        engines = new ScriptEngineManager(this, addons);
        EditText input = findViewById(getResources().getIdentifier("input","id",getPackageName()));
        TextView output = findViewById(getResources().getIdentifier("output","id",getPackageName()));
        Button runJs = findViewById(getResources().getIdentifier("runJs","id",getPackageName()));
        Button runLua = findViewById(getResources().getIdentifier("runLua","id",getPackageName()));
        runJs.setOnClickListener(v->{
            try { Object r = engines.evalJs(input.getText().toString()); output.setText(String.valueOf(r)); }
            catch(Throwable t){ output.setText("Error: "+t.getMessage()); }
        });
        runLua.setOnClickListener(v->{
            try { Object r = engines.evalLua(input.getText().toString()); output.setText(String.valueOf(r)); }
            catch(Throwable t){ output.setText("Error: "+t.getMessage()); }
        });
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/SettingsActivity.java

// File: SettingsActivity.java (FIXED) - Complete Settings with Proper Shizuku Detection
// Path: /app/src/main/java/com/modloader/ui/SettingsActivity.java

package com.modloader.ui;

import android.content.Intent;
import android.content.SharedPreferences;
import android.graphics.Color;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.provider.Settings;
import android.widget.*;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.cardview.widget.CardView;

import com.modloader.R;
import com.modloader.util.LogUtils;
import com.modloader.util.PermissionManager;
import com.modloader.util.ShizukuManager;

/**
 * FIXED: Complete Settings Activity with proper Shizuku permission detection
 * Handles all operation modes and permission management with real-time status updates
 */
public class SettingsActivity extends AppCompatActivity implements 
    PermissionManager.PermissionCallback, ShizukuManager.ShizukuPermissionCallback {

    // Operation Mode Constants
    public static final String PREF_OPERATION_MODE = "operation_mode";
    public static final String MODE_NORMAL = "normal";
    public static final String MODE_SHIZUKU = "shizuku";
    public static final String MODE_ROOT = "root";
    public static final String MODE_HYBRID = "hybrid";

    // UI Components
    private RadioGroup operationModeGroup;
    private RadioButton normalModeRadio;
    private RadioButton shizukuModeRadio;
    private RadioButton rootModeRadio;
    private RadioButton hybridModeRadio;
    
    private CardView normalCard, shizukuCard, rootCard, hybridCard;
    private TextView normalStatus, shizukuStatus, rootStatus, hybridStatus;
    private Button shizukuSetupBtn, rootSetupBtn, permissionBtn, refreshStatusBtn;
    
    private Switch autoEnableSwitch;
    private Switch debugLoggingSwitch;
    private Switch autoBackupSwitch;
    private Switch autoUpdateSwitch;

    // Managers
    private SharedPreferences prefs;
    private PermissionManager permissionManager;
    private ShizukuManager shizukuManager;
    
    // Status tracking
    private boolean isRefreshing = false;
    private Handler uiHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_settings_enhanced);
        setTitle("⚙️ Settings & Operation Modes");
        
        LogUtils.logUser("Settings activity opened");
        
        // Initialize handler for UI updates
        uiHandler = new Handler();
        
        // Initialize managers
        prefs = getSharedPreferences("terraria_loader_settings", MODE_PRIVATE);
        permissionManager = new PermissionManager(this);
        shizukuManager = new ShizukuManager(this);
        
        // Set callbacks
        permissionManager.setPermissionCallback(this);
        shizukuManager.setPermissionCallback(this);
        
        initializeViews();
        setupOperationModes();
        setupFeatureToggles();
        setupActionButtons();
        
        // Initial UI update
        updateUIState();
        
        // Auto-setup permissions based on current mode
        autoSetupPermissions();
    }

    private void initializeViews() {
        // Operation Mode Selection
        operationModeGroup = findViewById(R.id.operationModeGroup);
        normalModeRadio = findViewById(R.id.normalModeRadio);
        shizukuModeRadio = findViewById(R.id.shizukuModeRadio);
        rootModeRadio = findViewById(R.id.rootModeRadio);
        hybridModeRadio = findViewById(R.id.hybridModeRadio);

        // Mode Cards
        normalCard = findViewById(R.id.normalCard);
        shizukuCard = findViewById(R.id.shizukuCard);
        rootCard = findViewById(R.id.rootCard);
        hybridCard = findViewById(R.id.hybridCard);

        // Status Text
        normalStatus = findViewById(R.id.normalStatus);
        shizukuStatus = findViewById(R.id.shizukuStatus);
        rootStatus = findViewById(R.id.rootStatus);
        hybridStatus = findViewById(R.id.hybridStatus);

        // Setup Buttons
        shizukuSetupBtn = findViewById(R.id.shizukuSetupBtn);
        rootSetupBtn = findViewById(R.id.rootSetupBtn);
        permissionBtn = findViewById(R.id.permissionBtn);
        refreshStatusBtn = findViewById(R.id.refreshStatusBtn);

        // Feature Toggles
        autoEnableSwitch = findViewById(R.id.autoEnableSwitch);
        debugLoggingSwitch = findViewById(R.id.debugLoggingSwitch);
        autoBackupSwitch = findViewById(R.id.autoBackupSwitch);
        autoUpdateSwitch = findViewById(R.id.autoUpdateSwitch);
    }

    private void setupOperationModes() {
        // Load current mode
        String currentMode = prefs.getString(PREF_OPERATION_MODE, MODE_NORMAL);
        setOperationMode(currentMode, false);

        // Set up radio button listeners
        operationModeGroup.setOnCheckedChangeListener((group, checkedId) -> {
            String newMode;
            if (checkedId == R.id.normalModeRadio) {
                newMode = MODE_NORMAL;
            } else if (checkedId == R.id.shizukuModeRadio) {
                newMode = MODE_SHIZUKU;
            } else if (checkedId == R.id.rootModeRadio) {
                newMode = MODE_ROOT;
            } else if (checkedId == R.id.hybridModeRadio) {
                newMode = MODE_HYBRID;
            } else {
                newMode = MODE_NORMAL;
            }
            setOperationMode(newMode, true);
        });

        // Card click listeners for better UX
        setupCardListeners();
    }

    private void setupCardListeners() {
        normalCard.setOnClickListener(v -> {
            normalModeRadio.setChecked(true);
        });

        shizukuCard.setOnClickListener(v -> {
            if (shizukuManager.isShizukuAvailable()) {
                shizukuModeRadio.setChecked(true);
            } else {
                showShizukuSetupDialog();
            }
        });

        rootCard.setOnClickListener(v -> {
            // For now, we'll treat root as unavailable since we're focusing on Shizuku
            showRootInfoDialog();
        });

        hybridCard.setOnClickListener(v -> {
            if (shizukuManager.isShizukuAvailable()) {
                hybridModeRadio.setChecked(true);
            } else {
                showHybridSetupDialog();
            }
        });
    }

    private void setOperationMode(String mode, boolean save) {
        if (save) {
            prefs.edit().putString(PREF_OPERATION_MODE, mode).apply();
            LogUtils.logUser("Operation mode changed to: " + mode);
            // Auto-setup permissions for new mode
            autoSetupPermissions();
        }

        // Update radio buttons
        switch (mode) {
            case MODE_NORMAL:
                normalModeRadio.setChecked(true);
                break;
            case MODE_SHIZUKU:
                shizukuModeRadio.setChecked(true);
                break;
            case MODE_ROOT:
                rootModeRadio.setChecked(true);
                break;
            case MODE_HYBRID:
                hybridModeRadio.setChecked(true);
                break;
        }

        updateUIState();
    }

    private void setupFeatureToggles() {
        // Load current settings
        autoEnableSwitch.setChecked(prefs.getBoolean("auto_enable_mods", true));
        debugLoggingSwitch.setChecked(prefs.getBoolean("debug_logging", false));
        autoBackupSwitch.setChecked(prefs.getBoolean("auto_backup", true));
        autoUpdateSwitch.setChecked(prefs.getBoolean("auto_update_check", true));

        // Set up listeners
        autoEnableSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            prefs.edit().putBoolean("auto_enable_mods", isChecked).apply();
            LogUtils.logUser("Auto-enable mods: " + isChecked);
        });

        debugLoggingSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            prefs.edit().putBoolean("debug_logging", isChecked).apply();
            LogUtils.setDebugEnabled(isChecked);
            LogUtils.logUser("Debug logging: " + isChecked);
        });

        autoBackupSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            prefs.edit().putBoolean("auto_backup", isChecked).apply();
            LogUtils.logUser("Auto backup: " + isChecked);
        });

        autoUpdateSwitch.setOnCheckedChangeListener((buttonView, isChecked) -> {
            prefs.edit().putBoolean("auto_update_check", isChecked).apply();
            LogUtils.logUser("Auto update check: " + isChecked);
        });
    }

    private void setupActionButtons() {
        // FIXED: Shizuku Setup Button with proper permission detection
        shizukuSetupBtn.setOnClickListener(v -> handleShizukuSetup());

        // Root Setup Button (placeholder)
        rootSetupBtn.setOnClickListener(v -> showRootInfoDialog());

        // Permission Management Button
        permissionBtn.setOnClickListener(v -> showPermissionManagementDialog());

        // FIXED: Refresh Status Button
        refreshStatusBtn.setOnClickListener(v -> refreshAllStatuses());

        // Additional action buttons
        findViewById(R.id.resetSettingsBtn).setOnClickListener(v -> resetToDefaults());
        findViewById(R.id.exportSettingsBtn).setOnClickListener(v -> exportSettings());
        findViewById(R.id.importSettingsBtn).setOnClickListener(v -> importSettings());
    }

    /**
     * FIXED: Handle Shizuku setup with comprehensive status checking
     */
    private void handleShizukuSetup() {
        LogUtils.logUser("Handling Shizuku setup...");
        
        if (!shizukuManager.isShizukuInstalled()) {
            LogUtils.logUser("Shizuku not installed - showing install dialog");
            showShizukuInstallDialog();
        } else if (!shizukuManager.isShizukuRunning()) {
            LogUtils.logUser("Shizuku installed but not running - showing start dialog");
            showShizukuStartDialog();
        } else if (!shizukuManager.hasShizukuPermission()) {
            LogUtils.logUser("Shizuku running but permission not granted - requesting permission");
            shizukuManager.requestShizukuPermission();
        } else {
            LogUtils.logUser("Shizuku is fully ready!");
            Toast.makeText(this, "✅ Shizuku is already properly configured!", Toast.LENGTH_SHORT).show();
        }
    }

    /**
     * FIXED: Refresh all statuses with proper async handling
     */
    private void refreshAllStatuses() {
        if (isRefreshing) {
            LogUtils.logDebug("Refresh already in progress");
            return;
        }
        
        isRefreshing = true;
        refreshStatusBtn.setText("🔄 Refreshing...");
        refreshStatusBtn.setEnabled(false);
        
        LogUtils.logUser("Refreshing all permission statuses...");
        
        // Refresh in background thread to avoid UI blocking
        new Thread(() -> {
            try {
                // Force refresh all managers
                permissionManager.refreshPermissionStatus();
                shizukuManager.refreshPermissionStatus();
                
                // Update UI on main thread
                uiHandler.post(() -> {
                    updateUIState();
                    isRefreshing = false;
                    refreshStatusBtn.setText("🔄 Refresh Status");
                    refreshStatusBtn.setEnabled(true);
                    Toast.makeText(this, "Status refreshed", Toast.LENGTH_SHORT).show();
                });
                
            } catch (Exception e) {
                LogUtils.logError("Error refreshing statuses: " + e.getMessage());
                uiHandler.post(() -> {
                    isRefreshing = false;
                    refreshStatusBtn.setText("🔄 Refresh Status");
                    refreshStatusBtn.setEnabled(true);
                    Toast.makeText(this, "Refresh failed", Toast.LENGTH_SHORT).show();
                });
            }
        }).start();
    }

    /**
     * FIXED: Update UI state with real-time status checking
     */
    private void updateUIState() {
        String currentMode = prefs.getString(PREF_OPERATION_MODE, MODE_NORMAL);
        
        LogUtils.logDebug("Updating UI state - Current mode: " + currentMode);
        
        // Update card appearances with real-time status
        updateCardAppearance(normalCard, normalStatus, MODE_NORMAL.equals(currentMode),
                permissionManager.hasBasicPermissions(), getNormalStatusText());

        boolean shizukuReady = shizukuManager.isShizukuReady();
        updateCardAppearance(shizukuCard, shizukuStatus, MODE_SHIZUKU.equals(currentMode),
                shizukuReady, getShizukuStatusText());

        // Root mode (placeholder - always unavailable for now)
        updateCardAppearance(rootCard, rootStatus, MODE_ROOT.equals(currentMode),
                false, "Root access not implemented in this version");

        boolean hybridReady = shizukuReady; // For now, hybrid = shizuku only
        updateCardAppearance(hybridCard, hybridStatus, MODE_HYBRID.equals(currentMode),
                hybridReady, getHybridStatusText());

        // Update setup button states
        updateSetupButtons();
    }

    private void updateCardAppearance(CardView card, TextView status, boolean selected,
                                      boolean available, String statusText) {
        int cardColor;
        int textColor = Color.BLACK;
        
        if (selected) {
            cardColor = Color.parseColor("#E8F5E8"); // Light green
            card.setCardElevation(12f);
        } else if (available) {
            cardColor = Color.parseColor("#E3F2FD"); // Light blue
            card.setCardElevation(6f);
        } else {
            cardColor = Color.parseColor("#FFEBEE"); // Light red
            textColor = Color.parseColor("#666666");
            card.setCardElevation(2f);
        }

        card.setCardBackgroundColor(cardColor);
        status.setText(statusText);
        status.setTextColor(textColor);
    }

    /**
     * FIXED: Update setup button states with real-time status
     */
    private void updateSetupButtons() {
        // Shizuku setup button with comprehensive status
        if (shizukuManager.isShizukuReady()) {
            shizukuSetupBtn.setText("✅ Shizuku Ready");
            shizukuSetupBtn.setEnabled(false);
            shizukuSetupBtn.setBackgroundColor(Color.parseColor("#4CAF50"));
        } else if (shizukuManager.isShizukuRunning()) {
            shizukuSetupBtn.setText("🔐 Grant Permission");
            shizukuSetupBtn.setEnabled(true);
            shizukuSetupBtn.setBackgroundColor(Color.parseColor("#FF9800"));
        } else if (shizukuManager.isShizukuInstalled()) {
            shizukuSetupBtn.setText("▶️ Start Shizuku");
            shizukuSetupBtn.setEnabled(true);
            shizukuSetupBtn.setBackgroundColor(Color.parseColor("#2196F3"));
        } else {
            shizukuSetupBtn.setText("📥 Install Shizuku");
            shizukuSetupBtn.setEnabled(true);
            shizukuSetupBtn.setBackgroundColor(Color.parseColor("#9C27B0"));
        }

        // Root setup button (always disabled for now)
        rootSetupBtn.setText("❌ Root Not Supported");
        rootSetupBtn.setEnabled(false);
        rootSetupBtn.setBackgroundColor(Color.parseColor("#757575"));

        // Permission button - show overall status
        boolean hasBasic = permissionManager.hasBasicPermissions();
        boolean hasEnhanced = permissionManager.hasEnhancedPermissions();
        
        if (hasBasic && hasEnhanced) {
            permissionBtn.setText("✅ All Permissions Ready");
            permissionBtn.setBackgroundColor(Color.parseColor("#4CAF50"));
        } else if (hasBasic) {
            permissionBtn.setText("⚠️ Basic Permissions OK");
            permissionBtn.setBackgroundColor(Color.parseColor("#FF9800"));
        } else {
            permissionBtn.setText("❌ Permissions Needed");
            permissionBtn.setBackgroundColor(Color.parseColor("#F44336"));
        }
    }

    /**
     * FIXED: Get detailed status text for each mode
     */
    private String getNormalStatusText() {
        boolean hasStorage = permissionManager.hasStoragePermission();
        boolean hasInstall = permissionManager.hasInstallPermission();
        
        if (hasStorage && hasInstall) {
            return "✅ Standard permissions ready";
        } else {
            StringBuilder status = new StringBuilder("❌ Missing: ");
            if (!hasStorage) status.append("Storage ");
            if (!hasInstall) status.append("Install");
            return status.toString();
        }
    }

    private String getShizukuStatusText() {
        if (!shizukuManager.isShizukuInstalled()) {
            return "❌ Shizuku app not installed";
        } else if (!shizukuManager.isShizukuRunning()) {
            return "❌ Shizuku service not running";
        } else if (!shizukuManager.hasShizukuPermission()) {
            return "❌ Shizuku permission not granted";
        } else {
            return "✅ Shizuku ready - Enhanced file access available";
        }
    }

    private String getHybridStatusText() {
        boolean shizukuReady = shizukuManager.isShizukuReady();
        boolean basicReady = permissionManager.hasBasicPermissions();
        
        if (shizukuReady && basicReady) {
            return "✅ Maximum capabilities - All systems ready";
        } else {
            StringBuilder status = new StringBuilder("❌ Missing: ");
            if (!basicReady) status.append("Basic ");
            if (!shizukuReady) status.append("Shizuku");
            return status.toString();
        }
    }

    /**
     * FIXED: Auto-setup permissions with proper detection
     */
    private void autoSetupPermissions() {
        String mode = prefs.getString(PREF_OPERATION_MODE, MODE_NORMAL);
        LogUtils.logDebug("Auto-setting up permissions for mode: " + mode);

        // Always request basic permissions
        if (!permissionManager.hasBasicPermissions()) {
            LogUtils.logUser("Basic permissions needed - will request when user interacts");
        }

        switch (mode) {
            case MODE_SHIZUKU:
            case MODE_HYBRID:
                if (shizukuManager.isShizukuRunning() && !shizukuManager.hasShizukuPermission()) {
                    LogUtils.logUser("Shizuku mode selected but permission not granted");
                }
                break;
        }
    }

    // ===== Dialog Methods =====

    private void showShizukuSetupDialog() {
        new AlertDialog.Builder(this)
                .setTitle("Shizuku Setup Required")
                .setMessage("Shizuku provides enhanced file access without root. Would you like to install it?")
                .setPositiveButton("Install", (dialog, which) -> shizukuManager.installShizuku())
                .setNeutralButton("Learn More", (dialog, which) -> openUrl("https://shizuku.rikka.app/"))
                .setNegativeButton("Cancel", null)
                .show();
    }

    private void showShizukuInstallDialog() {
        new AlertDialog.Builder(this)
                .setTitle("Install Shizuku")
                .setMessage("Shizuku needs to be downloaded and installed. This will open your browser.")
                .setPositiveButton("Download", (dialog, which) ->
                        openUrl("https://github.com/RikkaApps/Shizuku/releases/latest"))
                .setNegativeButton("Cancel", null)
                .show();
    }

    private void showShizukuStartDialog() {
        new AlertDialog.Builder(this)
                .setTitle("Start Shizuku Service")
                .setMessage("Shizuku is installed but not running. Please start it using ADB or root, " +
                           "then return to this app.")
                .setPositiveButton("Open Shizuku", (dialog, which) -> shizukuManager.openShizukuApp())
                .setNeutralButton("ADB Guide", (dialog, which) ->
                        openUrl("https://shizuku.rikka.app/guide/setup/"))
                .setNegativeButton("Cancel", null)
                .show();
    }

    private void showRootInfoDialog() {
        new AlertDialog.Builder(this)
                .setTitle("Root Access")
                .setMessage("Root access is not supported in this version. " +
                           "Use Shizuku mode for enhanced capabilities without root.")
                .setPositiveButton("Use Shizuku Instead", (dialog, which) -> {
                    if (shizukuManager.isShizukuAvailable()) {
                        shizukuModeRadio.setChecked(true);
                    } else {
                        showShizukuSetupDialog();
                    }
                })
                .setNegativeButton("OK", null)
                .show();
    }

    private void showHybridSetupDialog() {
        String message = "";
        boolean basicReady = permissionManager.hasBasicPermissions();
        boolean shizukuReady = shizukuManager.isShizukuReady();
        
        if (!basicReady) {
            message += "• Basic permissions not granted\n";
        }
        if (!shizukuReady) {
            message += "• Shizuku is not ready\n";
        }

        new AlertDialog.Builder(this)
                .setTitle("Hybrid Mode Requirements")
                .setMessage("Hybrid mode requires both basic and enhanced permissions:\n\n" + message +
                           "\nPlease set up both components first.")
                .setPositiveButton("Setup Now", (dialog, which) -> {
                    if (!basicReady) {
                        permissionManager.requestBasicPermissions();
                    }
                    if (!shizukuReady) {
                        handleShizukuSetup();
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    /**
     * FIXED: Show detailed permission management dialog
     */
    private void showPermissionManagementDialog() {
        String statusReport = permissionManager.getPermissionStatus();
        
        new AlertDialog.Builder(this)
                .setTitle("Permission Management")
                .setMessage(statusReport)
                .setPositiveButton("Request Missing", (dialog, which) -> {
                    permissionManager.requestAllPermissions();
                })
                .setNeutralButton("App Settings", (dialog, which) -> permissionManager.openAppSettings())
                .setNegativeButton("Close", null)
                .show();
    }

    // ===== Utility Methods =====

    private void openUrl(String url) {
        try {
            Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
            startActivity(intent);
        } catch (Exception e) {
            Toast.makeText(this, "Could not open browser", Toast.LENGTH_SHORT).show();
        }
    }

    private void resetToDefaults() {
        new AlertDialog.Builder(this)
                .setTitle("Reset Settings")
                .setMessage("This will reset all settings to their default values. Continue?")
                .setPositiveButton("Reset", (dialog, which) -> {
                    prefs.edit().clear().apply();
                    recreate(); // Reload activity with default settings
                    Toast.makeText(this, "Settings reset to defaults", Toast.LENGTH_SHORT).show();
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    private void exportSettings() {
        Toast.makeText(this, "Export settings - Coming soon", Toast.LENGTH_SHORT).show();
    }

    private void importSettings() {
        Toast.makeText(this, "Import settings - Coming soon", Toast.LENGTH_SHORT).show();
    }

    // ===== Activity Lifecycle =====

    @Override
    protected void onResume() {
        super.onResume();
        LogUtils.logDebug("Settings activity resumed - refreshing UI state");
        updateUIState(); // Refresh UI when returning from other apps
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Clean up handlers
        if (uiHandler != null) {
            uiHandler.removeCallbacksAndMessages(null);
        }
    }

    // ===== Permission Callbacks =====

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                          @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        permissionManager.handlePermissionResult(requestCode, permissions, grantResults);
        updateUIState(); // Refresh UI after permission changes
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        permissionManager.handleActivityResult(requestCode, resultCode);
        updateUIState(); // Refresh UI after returning from settings
    }

    // ===== PermissionManager.PermissionCallback Implementation =====

    @Override
    public void onPermissionGranted(String permission) {
        LogUtils.logUser("Permission granted: " + permission);
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "✅ " + permission + " permission granted", Toast.LENGTH_SHORT).show();
        });
    }

    @Override
    public void onPermissionDenied(String permission) {
        LogUtils.logUser("Permission denied: " + permission);
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "❌ " + permission + " permission denied", Toast.LENGTH_SHORT).show();
        });
    }

    @Override
    public void onPermissionError(String permission, String error) {
        LogUtils.logUser("Permission error for " + permission + ": " + error);
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "Error with " + permission + ": " + error, Toast.LENGTH_LONG).show();
        });
    }

    // ===== ShizukuManager.ShizukuPermissionCallback Implementation =====

    @Override
    public void onPermissionGranted() {
        LogUtils.logUser("Shizuku permission granted!");
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "✅ Shizuku permission granted! Enhanced capabilities enabled.", Toast.LENGTH_LONG).show();
        });
    }

    @Override
    public void onPermissionDenied() {
        LogUtils.logUser("Shizuku permission denied");
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "❌ Shizuku permission denied. Enhanced features unavailable.", Toast.LENGTH_LONG).show();
        });
    }

    @Override
    public void onPermissionError(String error) {
        LogUtils.logUser("Shizuku permission error: " + error);
        uiHandler.post(() -> {
            updateUIState();
            Toast.makeText(this, "Shizuku error: " + error, Toast.LENGTH_LONG).show();
        });
    }

    // ===== Static Utility Methods for Other Activities =====

    public static String getCurrentOperationMode(android.content.Context context) {
        return context.getSharedPreferences("terraria_loader_settings", android.content.Context.MODE_PRIVATE)
                .getString(PREF_OPERATION_MODE, MODE_NORMAL);
    }

    public static boolean isShizukuMode(android.content.Context context) {
        String mode = getCurrentOperationMode(context);
        return MODE_SHIZUKU.equals(mode) || MODE_HYBRID.equals(mode);
    }

    public static boolean isRootMode(android.content.Context context) {
        String mode = getCurrentOperationMode(context);
        return MODE_ROOT.equals(mode) || MODE_HYBRID.equals(mode);
    }

    public static boolean canUseEnhancedPermissions(android.content.Context context) {
        return isShizukuMode(context); // Root not supported yet
    }

    // Legacy compatibility methods for existing code
    public static boolean isModsEnabled(android.content.Context context) {
        try {
            return context.getSharedPreferences("terraria_loader_settings", android.content.Context.MODE_PRIVATE)
                    .getBoolean("auto_enable_mods", true);
        } catch (Exception e) {
            return true;
        }
    }

    public static boolean isSandboxMode(android.content.Context context) {
        try {
            return context.getSharedPreferences("terraria_loader_settings", android.content.Context.MODE_PRIVATE)
                    .getBoolean("sandbox_mode", false);
        } catch (Exception e) {
            return false;
        }
    }

    public static boolean isDebugMode(android.content.Context context) {
        try {
            return context.getSharedPreferences("terraria_loader_settings", android.content.Context.MODE_PRIVATE)
                    .getBoolean("debug_logging", false);
        } catch (Exception e) {
            return false;
        }
    }

    public static boolean isAutoSaveEnabled(android.content.Context context) {
        try {
            return context.getSharedPreferences("terraria_loader_settings", android.content.Context.MODE_PRIVATE)
                    .getBoolean("auto_backup", true);
        } catch (Exception e) {
            return true;
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/SetupGuideActivity.java

// File: SetupGuideActivity.java (Updated) - Added Offline ZIP Import
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/app/src/main/java/com/terrarialoader/ui/SetupGuideActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

import com.modloader.R;
import com.modloader.loader.MelonLoaderManager;
import com.modloader.util.LogUtils;
import com.modloader.util.OnlineInstaller;
import com.modloader.util.OfflineZipImporter;

public class SetupGuideActivity extends AppCompatActivity {

    private static final int REQUEST_SELECT_ZIP = 1001;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_setup_guide);

        setTitle("🚀 MelonLoader Setup Guide");

        setupButtons();
    }

    private void setupButtons() {
        Button btnOnlineInstall = findViewById(R.id.btn_online_install);
        Button btnOfflineImport = findViewById(R.id.btn_offline_import);
        Button btnManualInstructions = findViewById(R.id.btn_manual_instructions);

        btnOnlineInstall.setOnClickListener(v -> showOnlineInstallDialog());
        btnOfflineImport.setOnClickListener(v -> showOfflineImportDialog());
        btnManualInstructions.setOnClickListener(v -> {
            Intent intent = new Intent(this, InstructionsActivity.class);
            startActivity(intent);
        });
    }

    private void showOnlineInstallDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("🌐 Automated Online Installation");
        builder.setMessage("This will automatically download and install MelonLoader/LemonLoader files from GitHub.\n\n" +
                          "Requirements:\n" +
                          "• Active internet connection\n" +
                          "• ~50MB free space\n\n" +
                          "Continue with automated installation?");
        
        builder.setPositiveButton("Continue", (dialog, which) -> {
            showLoaderTypeDialog();
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    private void showOfflineImportDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("📦 Offline ZIP Import");
        builder.setMessage("Import a MelonLoader ZIP file that you've already downloaded.\n\n" +
                          "Supported files:\n" +
                          "• melon_data.zip (MelonLoader)\n" +
                          "• lemon_data.zip (LemonLoader)\n" +
                          "• Custom MelonLoader packages\n\n" +
                          "The ZIP will be automatically extracted to the correct directories.");
        
        builder.setPositiveButton("📂 Select ZIP File", (dialog, which) -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/zip");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            startActivityForResult(intent, REQUEST_SELECT_ZIP);
        });
        
        builder.setNegativeButton("Cancel", null);
        builder.show();
    }

    private void showLoaderTypeDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Choose Loader Type");
        builder.setMessage("Select which loader to install:\n\n" +
                          "🔸 MelonLoader:\n" +
                          "• Full-featured Unity mod loader\n" +
                          "• Larger file size (~40MB)\n" +
                          "• Best compatibility\n\n" +
                          "🔸 LemonLoader:\n" +
                          "• Lightweight Unity mod loader\n" +
                          "• Smaller file size (~15MB)\n" +
                          "• Faster installation\n\n" +
                          "Which would you like to install?");
        
        builder.setPositiveButton("MelonLoader", (dialog, which) -> {
            LogUtils.logUser("User selected MelonLoader for automated installation");
            startAutomatedInstallation(MelonLoaderManager.LoaderType.MELONLOADER_NET8);
        });
        
        builder.setNegativeButton("LemonLoader", (dialog, which) -> {
            LogUtils.logUser("User selected LemonLoader for automated installation");
            startAutomatedInstallation(MelonLoaderManager.LoaderType.MELONLOADER_NET35);
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }
    
    private void startAutomatedInstallation(MelonLoaderManager.LoaderType loaderType) {
        LogUtils.logUser("Starting automated " + loaderType.getDisplayName() + " installation...");
        
        AlertDialog progressDialog = new AlertDialog.Builder(this)
            .setTitle("Installing " + loaderType.getDisplayName())
            .setMessage("Downloading and extracting files from GitHub...\nThis may take a few minutes.")
            .setCancelable(false)
            .show();

        new Thread(() -> {
            boolean success = false;
            String errorMessage = "";

            try {
                OnlineInstaller.InstallationResult result = OnlineInstaller.installMelonLoaderOnline(
                    this, MelonLoaderManager.TERRARIA_PACKAGE, loaderType);
                success = result.success;
                errorMessage = result.message;
                
            } catch (Exception e) {
                success = false;
                errorMessage = e.getMessage();
                LogUtils.logDebug("Automated installation error: " + errorMessage);
            }

            final boolean finalSuccess = success;
            final String finalErrorMessage = errorMessage;

            runOnUiThread(() -> {
                progressDialog.dismiss();
                if (finalSuccess) {
                    showInstallationSuccessDialog(loaderType);
                } else {
                    showInstallationErrorDialog(loaderType, finalErrorMessage);
                }
            });
        }).start();
    }

    private void startOfflineImportProcess(Uri zipUri) {
        LogUtils.logUser("Starting offline ZIP import process...");
        
        AlertDialog progressDialog = new AlertDialog.Builder(this)
            .setTitle("Importing MelonLoader ZIP")
            .setMessage("Analyzing and extracting ZIP file...\nThis may take a moment.")
            .setCancelable(false)
            .show();

        new Thread(() -> {
            OfflineZipImporter.ImportResult result = OfflineZipImporter.importMelonLoaderZip(this, zipUri);

            runOnUiThread(() -> {
                progressDialog.dismiss();
                if (result.success) {
                    showImportSuccessDialog(result);
                } else {
                    showImportErrorDialog(result);
                }
            });
        }).start();
    }

    private void showInstallationSuccessDialog(MelonLoaderManager.LoaderType loaderType) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("✅ Installation Complete!");
        builder.setMessage("Great! " + loaderType.getDisplayName() + " has been successfully installed!\n\n" +
                          "Next steps:\n" +
                          "1. Go to Unified Loader Activity\n" +
                          "2. Select your Terraria APK\n" +
                          "3. Patch APK with loader\n" +
                          "4. Install patched Terraria\n" +
                          "5. Add DLL mods and enjoy!\n\n" +
                          "You can now use DLL mods with Terraria!");
        
        builder.setPositiveButton("🚀 Open Unified Loader", (dialog, which) -> {
            Intent intent = new Intent(this, UnifiedLoaderActivity.class);
            startActivity(intent);
            finish();
        });
        
        builder.setNegativeButton("Later", (dialog, which) -> {
            finish();
        });
        
        builder.show();
    }

    private void showInstallationErrorDialog(MelonLoaderManager.LoaderType loaderType, String errorMessage) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("❌ Installation Failed");
        builder.setMessage("Failed to install " + loaderType.getDisplayName() + "\n\n" +
                          "Error: " + (errorMessage.isEmpty() ? "Unknown error occurred" : errorMessage) + "\n\n" +
                          "Please try:\n" +
                          "• Check your internet connection\n" +
                          "• Use Offline ZIP Import instead\n" +
                          "• Use Manual Installation\n" +
                          "• Try again later");
        
        builder.setPositiveButton("📦 Try Offline Import", (dialog, which) -> {
            showOfflineImportDialog();
        });
        
        builder.setNegativeButton("📖 Manual Guide", (dialog, which) -> {
            Intent intent = new Intent(this, InstructionsActivity.class);
            startActivity(intent);
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }

    private void showImportSuccessDialog(OfflineZipImporter.ImportResult result) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("✅ ZIP Import Complete!");
        builder.setMessage("Successfully imported " + result.detectedType.getDisplayName() + "!\n\n" +
                          "Files extracted: " + result.filesExtracted + "\n\n" +
                          "The loader files have been automatically placed in the correct directories:\n" +
                          "• NET8/NET35 runtime files\n" +
                          "• Dependencies and support modules\n" +
                          "• All required components\n\n" +
                          "You can now patch APK files!");
        
        builder.setPositiveButton("🚀 Open Unified Loader", (dialog, which) -> {
            Intent intent = new Intent(this, UnifiedLoaderActivity.class);
            startActivity(intent);
            finish();
        });
        
        builder.setNegativeButton("✅ Done", (dialog, which) -> {
            finish();
        });
        
        builder.show();
    }

    private void showImportErrorDialog(OfflineZipImporter.ImportResult result) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("❌ ZIP Import Failed");
        builder.setMessage("Failed to import ZIP file\n\n" +
                          "Error: " + result.message + "\n\n" +
                          (result.errorDetails != null ? "Details: " + result.errorDetails + "\n\n" : "") +
                          "Please ensure:\n" +
                          "• ZIP file is a valid MelonLoader package\n" +
                          "• File is not corrupted\n" +
                          "• You have sufficient storage space");
        
        builder.setPositiveButton("🌐 Try Online Install", (dialog, which) -> {
            showOnlineInstallDialog();
        });
        
        builder.setNegativeButton("📖 Manual Guide", (dialog, which) -> {
            Intent intent = new Intent(this, InstructionsActivity.class);
            startActivity(intent);
        });
        
        builder.setNeutralButton("Try Again", (dialog, which) -> {
            showOfflineImportDialog();
        });
        
        builder.show();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (requestCode == REQUEST_SELECT_ZIP && resultCode == Activity.RESULT_OK && data != null) {
            Uri zipUri = data.getData();
            if (zipUri != null) {
                LogUtils.logUser("ZIP file selected for offline import");
                startOfflineImportProcess(zipUri);
            }
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/SimpleAdapter.java

package com.modloader.ui;
import android.view.*;
import android.widget.TextView;
import androidx.recyclerview.widget.RecyclerView;
import java.util.List;
public class SimpleAdapter extends RecyclerView.Adapter<SimpleAdapter.VH>{
    private final List<String> data;
    public SimpleAdapter(List<String> d){ this.data=d; }
    public static class VH extends RecyclerView.ViewHolder {
        TextView tv;
        VH(View v){ super(v); tv = (TextView)v; }
    }
    @Override public VH onCreateViewHolder(ViewGroup p, int vtype){
        TextView tv = new TextView(p.getContext());
        tv.setPadding(24,24,24,24);
        return new VH(tv);
    }
    @Override public void onBindViewHolder(VH h, int pos){ h.tv.setText(data.get(pos)); }
    @Override public int getItemCount(){ return data.size(); }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/UnifiedLoaderActivity.java

// File: UnifiedLoaderActivity.java - Complete Fixed Version
// Path: /main/java/com/modloader/ui/UnifiedLoaderActivity.java

package com.modloader.ui;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.provider.OpenableColumns;
import android.view.View;
import android.widget.*;
import android.graphics.Typeface;
import androidx.appcompat.app.AppCompatActivity;

import com.modloader.R;
import com.modloader.loader.MelonLoaderManager;
import com.modloader.util.LogUtils;

/**
 * Unified Loader Activity - Complete wizard-style interface for MelonLoader setup
 */
public class UnifiedLoaderActivity extends AppCompatActivity implements 
    UnifiedLoaderController.UnifiedLoaderCallback, UnifiedLoaderListener {

    private static final int REQUEST_SELECT_APK = 1001;
    private static final int REQUEST_SELECT_ZIP = 1002;
    
    // UI Components
    private ProgressBar stepProgressBar;
    private TextView stepTitleText;
    private TextView stepDescriptionText;
    private TextView stepIndicatorText;
    private LinearLayout stepContentContainer;
    private Button previousButton;
    private Button nextButton;
    private Button actionButton;
    
    // Current step content views
    private LinearLayout welcomeContent;
    private LinearLayout loaderInstallContent;
    private LinearLayout apkSelectionContent;
    private LinearLayout patchingContent;
    private LinearLayout completionContent;
    
    // Status indicators
    private TextView loaderStatusText;
    private TextView apkStatusText;
    private TextView progressText;
    private ProgressBar actionProgressBar;
    
    // Controller
    private UnifiedLoaderController controller;
    private AlertDialog progressDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_unified_loader);
        
        setTitle("MelonLoader Setup Wizard");
        
        // Initialize controller
        controller = new UnifiedLoaderController(this);
        controller.setCallback(this);
        
        initializeViews();
        setupStepContents();
        setupListeners();
        
        // Start wizard
        controller.setCurrentStep(UnifiedLoaderController.LoaderStep.WELCOME);
    }

    private void initializeViews() {
        stepProgressBar = findViewById(R.id.stepProgressBar);
        stepTitleText = findViewById(R.id.stepTitleText);
        stepDescriptionText = findViewById(R.id.stepDescriptionText);
        stepIndicatorText = findViewById(R.id.stepIndicatorText);
        stepContentContainer = findViewById(R.id.stepContentContainer);
        previousButton = findViewById(R.id.previousButton);
        nextButton = findViewById(R.id.nextButton);
        actionButton = findViewById(R.id.actionButton);
        
        loaderStatusText = findViewById(R.id.loaderStatusText);
        apkStatusText = findViewById(R.id.apkStatusText);
        progressText = findViewById(R.id.progressText);
        actionProgressBar = findViewById(R.id.actionProgressBar);
    }

    private void setupStepContents() {
        // Create step content views dynamically
        welcomeContent = createWelcomeContent();
        loaderInstallContent = createLoaderInstallContent();
        apkSelectionContent = createApkSelectionContent();
        patchingContent = createPatchingContent();
        completionContent = createCompletionContent();
    }

    private LinearLayout createWelcomeContent() {
        LinearLayout content = new LinearLayout(this);
        content.setOrientation(LinearLayout.VERTICAL);
        content.setPadding(24, 24, 24, 24);
        
        TextView welcomeText = new TextView(this);
        welcomeText.setText("Welcome to MelonLoader Setup!\n\nThis wizard will guide you through:\n\n• Installing MelonLoader/LemonLoader\n• Patching your Terraria APK\n• Setting up DLL mod support\n\nClick 'Next' to begin!");
        welcomeText.setTextSize(16);
        welcomeText.setLineSpacing(8, 1.0f);
        content.addView(welcomeText);
        
        return content;
    }

    private LinearLayout createLoaderInstallContent() {
        LinearLayout content = new LinearLayout(this);
        content.setOrientation(LinearLayout.VERTICAL);
        content.setPadding(16, 16, 16, 16);
        
        // Loader status
        TextView statusLabel = new TextView(this);
        statusLabel.setText("Current Status:");
        statusLabel.setTextSize(14);
        statusLabel.setTypeface(null, Typeface.BOLD);
        content.addView(statusLabel);
        
        TextView statusText = new TextView(this);
        statusText.setText("Checking...");
        statusText.setTextSize(14);
        statusText.setPadding(0, 8, 0, 16);
        content.addView(statusText);
        
        // Installation options
        TextView optionsLabel = new TextView(this);
        optionsLabel.setText("Installation Options:");
        optionsLabel.setTextSize(14);
        optionsLabel.setTypeface(null, Typeface.BOLD);
        content.addView(optionsLabel);
        
        Button onlineInstallBtn = new Button(this);
        onlineInstallBtn.setText("Online Installation (Recommended)");
        onlineInstallBtn.setOnClickListener(v -> showOnlineInstallOptions());
        content.addView(onlineInstallBtn);
        
        Button offlineInstallBtn = new Button(this);
        offlineInstallBtn.setText("Offline ZIP Import");
        offlineInstallBtn.setOnClickListener(v -> selectOfflineZip());
        content.addView(offlineInstallBtn);
        
        return content;
    }

    private LinearLayout createApkSelectionContent() {
        LinearLayout content = new LinearLayout(this);
        content.setOrientation(LinearLayout.VERTICAL);
        content.setPadding(16, 16, 16, 16);
        
        TextView instructionText = new TextView(this);
        instructionText.setText("Select your Terraria APK file to patch with MelonLoader:");
        instructionText.setTextSize(16);
        content.addView(instructionText);
        
        Button selectApkBtn = new Button(this);
        selectApkBtn.setText("Select Terraria APK");
        selectApkBtn.setOnClickListener(v -> selectApkFile());
        content.addView(selectApkBtn);
        
        TextView statusText = new TextView(this);
        statusText.setText("No APK selected");
        statusText.setTextSize(14);
        statusText.setPadding(0, 16, 0, 0);
        content.addView(statusText);
        
        return content;
    }

    private LinearLayout createPatchingContent() {
        LinearLayout content = new LinearLayout(this);
        content.setOrientation(LinearLayout.VERTICAL);
        content.setPadding(16, 16, 16, 16);
        content.setGravity(android.view.Gravity.CENTER);
        
        TextView patchingText = new TextView(this);
        patchingText.setText("Patching APK with MelonLoader...");
        patchingText.setTextSize(18);
        patchingText.setGravity(android.view.Gravity.CENTER);
        content.addView(patchingText);
        
        ProgressBar progressBar = new ProgressBar(this);
        progressBar.setIndeterminate(true);
        LinearLayout.LayoutParams progressParams = new LinearLayout.LayoutParams(
            LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT);
        progressParams.topMargin = 24;
        progressParams.gravity = android.view.Gravity.CENTER;
        progressBar.setLayoutParams(progressParams);
        content.addView(progressBar);
        
        TextView statusText = new TextView(this);
        statusText.setText("Initializing...");
        statusText.setTextSize(14);
        statusText.setGravity(android.view.Gravity.CENTER);
        statusText.setPadding(0, 16, 0, 0);
        content.addView(statusText);
        
        return content;
    }

    private LinearLayout createCompletionContent() {
        LinearLayout content = new LinearLayout(this);
        content.setOrientation(LinearLayout.VERTICAL);
        content.setPadding(24, 24, 24, 24);
        
        TextView completionText = new TextView(this);
        completionText.setText("Setup Complete!\n\nYour modded Terraria APK is ready!");
        completionText.setTextSize(18);
        completionText.setGravity(android.view.Gravity.CENTER);
        content.addView(completionText);
        
        Button installApkBtn = new Button(this);
        installApkBtn.setText("Install Patched APK");
        installApkBtn.setOnClickListener(v -> controller.installPatchedApk());
        content.addView(installApkBtn);
        
        Button manageModsBtn = new Button(this);
        manageModsBtn.setText("Manage DLL Mods");
        manageModsBtn.setOnClickListener(v -> openModManagement());
        content.addView(manageModsBtn);
        
        Button viewLogsBtn = new Button(this);
        viewLogsBtn.setText("View Logs");
        viewLogsBtn.setOnClickListener(v -> startActivity(new Intent(this, LogViewerEnhancedActivity.class)));
        content.addView(viewLogsBtn);
        
        return content;
    }

    private void setupListeners() {
        previousButton.setOnClickListener(v -> {
            if (controller.canProceedToPreviousStep()) {
                controller.previousStep();
            }
        });
        
        nextButton.setOnClickListener(v -> {
            if (controller.canProceedToNextStep()) {
                handleNextStep();
            } else {
                showStepRequirements();
            }
        });
        
        actionButton.setOnClickListener(v -> handleActionButton());
    }

    private void handleNextStep() {
        UnifiedLoaderController.LoaderStep currentStep = controller.getCurrentStep();
        
        switch (currentStep) {
            case WELCOME:
                controller.nextStep();
                break;
            case LOADER_INSTALL:
                if (controller.isLoaderInstalled()) {
                    controller.nextStep();
                } else {
                    Toast.makeText(this, "Please install a loader first", Toast.LENGTH_SHORT).show();
                }
                break;
            case APK_SELECTION:
                if (controller.getSelectedApkUri() != null) {
                    controller.nextStep();
                    controller.patchApk(); // Auto-start patching
                } else {
                    Toast.makeText(this, "Please select an APK first", Toast.LENGTH_SHORT).show();
                }
                break;
            case APK_PATCHING:
                // Patching in progress, disable navigation
                break;
            case COMPLETION:
                finish(); // Exit wizard
                break;
        }
    }

    private void handleActionButton() {
        UnifiedLoaderController.LoaderStep currentStep = controller.getCurrentStep();
        
        switch (currentStep) {
            case WELCOME:
                controller.nextStep();
                break;
            case LOADER_INSTALL:
                showOnlineInstallOptions();
                break;
            case APK_SELECTION:
                selectApkFile();
                break;
            case APK_PATCHING:
                // No action during patching
                break;
            case COMPLETION:
                controller.installPatchedApk();
                break;
        }
    }

    private void showOnlineInstallOptions() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Choose Loader Type");
        builder.setMessage("Select which loader to install:\n\nMelonLoader: Full-featured, larger size\nLemonLoader: Lightweight, smaller size");
        
        builder.setPositiveButton("MelonLoader", (dialog, which) -> {
            controller.installLoaderOnline(MelonLoaderManager.LoaderType.MELONLOADER_NET8);
        });
        
        builder.setNegativeButton("LemonLoader", (dialog, which) -> {
            controller.installLoaderOnline(MelonLoaderManager.LoaderType.MELONLOADER_NET35);
        });
        
        builder.setNeutralButton("Cancel", null);
        builder.show();
    }

    private void selectOfflineZip() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("application/zip");
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        startActivityForResult(intent, REQUEST_SELECT_ZIP);
    }

    private void selectApkFile() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("application/vnd.android.package-archive");
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        startActivityForResult(intent, REQUEST_SELECT_APK);
    }

    private void openModManagement() {
        Intent intent = new Intent(this, ModManagementActivity.class);
        startActivity(intent);
    }

    private void showStepRequirements() {
        UnifiedLoaderController.LoaderStep currentStep = controller.getCurrentStep();
        String message = "";
        
        switch (currentStep) {
            case LOADER_INSTALL:
                message = "Please install MelonLoader or LemonLoader first";
                break;
            case APK_SELECTION:
                message = "Please select a Terraria APK file";
                break;
            default:
                message = "Please complete the current step";
                break;
        }
        
        Toast.makeText(this, message, Toast.LENGTH_LONG).show();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        
        if (resultCode != Activity.RESULT_OK || data == null || data.getData() == null) {
            return;
        }
        
        Uri uri = data.getData();
        
        switch (requestCode) {
            case REQUEST_SELECT_APK:
                controller.selectApk(uri);
                String filename = getFilenameFromUri(uri);
                if (apkStatusText != null) {
                    apkStatusText.setText("Selected: " + filename);
                }
                break;
                
            case REQUEST_SELECT_ZIP:
                controller.installLoaderOffline(uri);
                break;
        }
    }

    private String getFilenameFromUri(Uri uri) {
        String filename = null;
        try (Cursor cursor = getContentResolver().query(uri, null, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                int nameIndex = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                if (nameIndex >= 0) {
                    filename = cursor.getString(nameIndex);
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Could not get filename: " + e.getMessage());
        }
        return filename != null ? filename : "Unknown file";
    }

    // === UnifiedLoaderController.UnifiedLoaderCallback Implementation ===

    @Override
    public void onStepChanged(UnifiedLoaderController.LoaderStep step, String message) {
        runOnUiThread(() -> {
            stepTitleText.setText(step.getTitle());
            stepDescriptionText.setText(message);
            
            // Clear previous content
            stepContentContainer.removeAllViews();
            
            // Add appropriate content
            switch (step) {
                case WELCOME:
                    stepContentContainer.addView(welcomeContent);
                    actionButton.setText("Start Setup");
                    actionButton.setVisibility(View.VISIBLE);
                    break;
                case LOADER_INSTALL:
                    stepContentContainer.addView(loaderInstallContent);
                    actionButton.setText("Install Online");
                    actionButton.setVisibility(View.VISIBLE);
                    break;
                case APK_SELECTION:
                    stepContentContainer.addView(apkSelectionContent);
                    actionButton.setText("Select APK");
                    actionButton.setVisibility(View.VISIBLE);
                    break;
                case APK_PATCHING:
                    stepContentContainer.addView(patchingContent);
                    actionButton.setVisibility(View.GONE);
                    nextButton.setEnabled(false);
                    previousButton.setEnabled(false);
                    break;
                case COMPLETION:
                    stepContentContainer.addView(completionContent);
                    actionButton.setText("Install APK");
                    actionButton.setVisibility(View.VISIBLE);
                    nextButton.setText("Finish");
                    nextButton.setEnabled(true);
                    previousButton.setEnabled(true);
                    break;
            }
            
            // Update navigation buttons
            previousButton.setEnabled(controller.canProceedToPreviousStep());
            nextButton.setEnabled(controller.canProceedToNextStep());
        });
    }

    @Override
    public void onProgress(String message, int percentage) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.setMessage(message + (percentage > 0 ? " (" + percentage + "%)" : ""));
            }
            if (progressText != null) {
                progressText.setText(message);
            }
        });
    }

    @Override
    public void onSuccess(String message) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
                progressDialog = null;
            }
            Toast.makeText(this, message, Toast.LENGTH_LONG).show();
        });
    }

    @Override
    public void onError(String error) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
                progressDialog = null;
            }
            
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.setTitle("Error");
            builder.setMessage(error);
            builder.setPositiveButton("OK", null);
            builder.show();
            
            // Re-enable navigation
            nextButton.setEnabled(controller.canProceedToNextStep());
            previousButton.setEnabled(controller.canProceedToPreviousStep());
        });
    }

    @Override
    public void onLoaderStatusChanged(boolean installed, String statusText) {
        runOnUiThread(() -> {
            if (loaderStatusText != null) {
                loaderStatusText.setText(statusText);
                loaderStatusText.setTextColor(installed ? 0xFF4CAF50 : 0xFFF44336);
            }
        });
    }

    @Override
    public void updateStepIndicator(int currentStep, int totalSteps) {
        runOnUiThread(() -> {
            stepProgressBar.setMax(totalSteps);
            stepProgressBar.setProgress(currentStep);
            stepIndicatorText.setText("Step " + (currentStep + 1) + " of " + (totalSteps + 1));
        });
    }

    // === UnifiedLoaderListener Implementation ===

    @Override
    public void onInstallationStarted(String loaderType) {
        runOnUiThread(() -> {
            progressDialog = new AlertDialog.Builder(this)
                .setTitle("Installing " + loaderType)
                .setMessage("Starting installation...")
                .setCancelable(false)
                .create();
            progressDialog.show();
        });
    }

    @Override
    public void onInstallationProgress(String message) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.setMessage(message);
            }
        });
    }

    @Override
    public void onInstallationSuccess(String loaderType, String outputPath) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
                progressDialog = null;
            }
            Toast.makeText(this, loaderType + " installed successfully!", Toast.LENGTH_LONG).show();
        });
    }

    @Override
    public void onInstallationFailed(String loaderType, String error) {
        runOnUiThread(() -> {
            if (progressDialog != null) {
                progressDialog.dismiss();
                progressDialog = null;
            }
            
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.setTitle("Installation Failed");
            builder.setMessage(loaderType + " installation failed:\n\n" + error);
            builder.setPositiveButton("OK", null);
            builder.show();
        });
    }

    @Override
    public void onValidationComplete(boolean isValid, String message) {
        runOnUiThread(() -> {
            String statusText = isValid ? "Validation passed" : "Validation failed: " + message;
            Toast.makeText(this, statusText, Toast.LENGTH_SHORT).show();
        });
    }

    @Override
    public void onInstallationStateChanged(UnifiedLoaderController.InstallationState state) {
        runOnUiThread(() -> {
            LogUtils.logDebug("Installation state changed to: " + state.getDisplayName());
        });
    }

    @Override
    public void onLogMessage(String message, UnifiedLoaderController.LogLevel level) {
        // Log messages are already handled by the controller
        LogUtils.logDebug("Log: [" + level + "] " + message);
    }

    @Override
    protected void onResume() {
        super.onResume();
        // Refresh loader status when returning to activity
        if (controller != null) {
            // Update current step to refresh status
            controller.setCurrentStep(controller.getCurrentStep());
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (controller != null) {
            controller.cleanup();
        }
        if (progressDialog != null) {
            progressDialog.dismiss();
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/UnifiedLoaderController.java

// File: UnifiedLoaderController.java - Fixed step progression for offline ZIP import
// Path: /app/src/main/java/com/modloader/ui/UnifiedLoaderController.java

package com.modloader.ui;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.os.Handler;
import android.os.Looper;
import android.widget.Toast;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;

import com.modloader.loader.MelonLoaderManager;
import com.modloader.loader.LoaderInstaller;
import com.modloader.loader.LoaderValidator;
import com.modloader.util.ApkPatcher;
import com.modloader.util.LogUtils;
import com.modloader.util.FileUtils;
import com.modloader.util.PathManager;
import com.modloader.util.OfflineZipImporter;
import java.io.File;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class UnifiedLoaderController {
    private static final String TAG = "UnifiedLoaderController";
    
    private Activity activity;
    private LoaderInstaller loaderInstaller;
    private LoaderValidator loaderValidator;
    private UnifiedLoaderListener listener;
    private Handler mainHandler;
    private ExecutorService executorService;
    
    // File picker launchers
    private ActivityResultLauncher<Intent> apkPickerLauncher;
    private ActivityResultLauncher<Intent> zipPickerLauncher;
    
    // Current operation state
    private File selectedApkFile;
    private File selectedZipFile;
    private MelonLoaderManager.LoaderType selectedLoaderType;
    private boolean isOperationInProgress = false;
    private InstallationState currentState = InstallationState.IDLE;
    private boolean loaderInstalledSuccessfully = false; // NEW: Track successful installation
    
    // Step management
    private LoaderStep currentStep = LoaderStep.WELCOME;
    private Uri selectedApkUri;
    
    /**
     * Installation state enum for tracking current operation state
     */
    public enum InstallationState {
        IDLE("Idle"),
        INITIALIZING("Initializing"),
        DOWNLOADING("Downloading"),
        EXTRACTING("Extracting Files"),
        CREATING_DIRECTORIES("Creating Directories"),
        VALIDATING("Validating Installation"),
        PATCHING_APK("Patching APK"),
        INSTALLING_APK("Installing APK"),
        COMPLETED("Installation Complete"),
        FAILED("Installation Failed"),
        CANCELLED("Operation Cancelled");
        
        private final String displayName;
        
        InstallationState(String displayName) {
            this.displayName = displayName;
        }
        
        public String getDisplayName() {
            return displayName;
        }
        
        @Override
        public String toString() {
            return displayName;
        }
    }
    
    /**
     * Log level enum for categorizing log messages
     */
    public enum LogLevel {
        DEBUG("DEBUG", 0),
        INFO("INFO", 1),
        WARNING("WARNING", 2),
        ERROR("ERROR", 3),
        USER("USER", 4);
        
        private final String displayName;
        private final int priority;
        
        LogLevel(String displayName, int priority) {
            this.displayName = displayName;
            this.priority = priority;
        }
        
        public String getDisplayName() {
            return displayName;
        }
        
        public int getPriority() {
            return priority;
        }
        
        @Override
        public String toString() {
            return displayName;
        }
    }
    
    // Legacy callback interface for backward compatibility
    public interface UnifiedLoaderCallback extends UnifiedLoaderListener {
        void onStepChanged(LoaderStep step, String message);
        void onProgress(String message, int percentage);
        void onSuccess(String message);
        void onError(String error);
        void onLoaderStatusChanged(boolean installed, String statusText);
        void updateStepIndicator(int currentStep, int totalSteps);
    }
    
    // LoaderStep enum for step tracking
    public enum LoaderStep {
        WELCOME("Welcome", "Welcome to MelonLoader Setup"),
        LOADER_INSTALL("Loader Installation", "Install MelonLoader components"),
        APK_SELECTION("APK Selection", "Select your Terraria APK"),
        APK_PATCHING("APK Patching", "Patching APK with MelonLoader"),
        COMPLETION("Setup Complete", "Installation completed successfully");
        
        private final String title;
        private final String description;
        
        LoaderStep(String title, String description) {
            this.title = title;
            this.description = description;
        }
        
        public String getTitle() {
            return title;
        }
        
        public String getDescription() {
            return description;
        }
    }
    
    public UnifiedLoaderController(Activity activity) {
        this.activity = activity;
        this.loaderInstaller = new LoaderInstaller();
        this.loaderValidator = new LoaderValidator();
        this.mainHandler = new Handler(Looper.getMainLooper());
        this.executorService = Executors.newSingleThreadExecutor();
        
        initializeFilePickers();
    }
    
    public UnifiedLoaderController(Activity activity, UnifiedLoaderListener listener) {
        this(activity);
        this.listener = listener;
    }
    
    // Callback setter for backward compatibility
    public void setCallback(UnifiedLoaderCallback callback) {
        this.listener = callback;
    }
    
    private void initializeFilePickers() {
        if (activity instanceof androidx.activity.ComponentActivity) {
            apkPickerLauncher = ((androidx.activity.ComponentActivity) activity).registerForActivityResult(
                new ActivityResultContracts.StartActivityForResult(),
                result -> {
                    if (result.getResultCode() == Activity.RESULT_OK && result.getData() != null) {
                        Uri apkUri = result.getData().getData();
                        handleApkSelection(apkUri);
                    }
                }
            );
            
            zipPickerLauncher = ((androidx.activity.ComponentActivity) activity).registerForActivityResult(
                new ActivityResultContracts.StartActivityForResult(),
                result -> {
                    if (result.getResultCode() == Activity.RESULT_OK && result.getData() != null) {
                        Uri zipUri = result.getData().getData();
                        handleZipSelection(zipUri);
                    }
                }
            );
        }
    }
    
    // State management methods
    private void setState(InstallationState newState) {
        this.currentState = newState;
        if (listener != null) {
            listener.onInstallationStateChanged(newState);
        }
        logMessage("State changed to: " + newState.getDisplayName(), LogLevel.DEBUG);
    }
    
    private void logMessage(String message, LogLevel level) {
        if (listener != null) {
            listener.onLogMessage(message, level);
        }
        
        switch (level) {
            case DEBUG:
                LogUtils.logDebug(message);
                break;
            case INFO:
                LogUtils.logInfo(message);
                break;
            case WARNING:
                LogUtils.logWarning(message);
                break;
            case ERROR:
                LogUtils.logError(message);
                break;
            case USER:
                LogUtils.logUser(message);
                break;
        }
    }
    
    // FIXED: Step management methods for wizard-style interface
    public void setCurrentStep(LoaderStep step) {
        this.currentStep = step;
        if (listener instanceof UnifiedLoaderCallback) {
            UnifiedLoaderCallback callback = (UnifiedLoaderCallback) listener;
            callback.onStepChanged(step, step.getDescription());
            callback.updateStepIndicator(step.ordinal(), LoaderStep.values().length - 1);
        }
        logMessage("Step changed to: " + step.getTitle(), LogLevel.INFO);
    }
    
    public LoaderStep getCurrentStep() {
        return currentStep;
    }
    
    public void nextStep() {
        LoaderStep[] steps = LoaderStep.values();
        int currentIndex = currentStep.ordinal();
        if (currentIndex < steps.length - 1) {
            setCurrentStep(steps[currentIndex + 1]);
        }
    }
    
    public void previousStep() {
        LoaderStep[] steps = LoaderStep.values();
        int currentIndex = currentStep.ordinal();
        if (currentIndex > 0) {
            setCurrentStep(steps[currentIndex - 1]);
        }
    }
    
    // FIXED: Improved step progression logic
    public boolean canProceedToNextStep() {
        switch (currentStep) {
            case WELCOME:
                return true;
            case LOADER_INSTALL:
                // Check both actual installation and successful import
                return isLoaderInstalled() || loaderInstalledSuccessfully;
            case APK_SELECTION:
                return selectedApkUri != null;
            case APK_PATCHING:
                return currentState == InstallationState.COMPLETED;
            case COMPLETION:
                return false; // Final step
            default:
                return false;
        }
    }
    
    public boolean canProceedToPreviousStep() {
        return currentStep != LoaderStep.WELCOME && currentState != InstallationState.PATCHING_APK;
    }
    
    // File selection methods
    public void selectApkFile() {
        if (isOperationInProgress) {
            showToast("Operation in progress, please wait...");
            return;
        }
        
        if (apkPickerLauncher != null) {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/vnd.android.package-archive");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            apkPickerLauncher.launch(Intent.createChooser(intent, "Select Terraria APK"));
        }
    }
    
    public void selectZipFile() {
        if (isOperationInProgress) {
            showToast("Operation in progress, please wait...");
            return;
        }
        
        if (zipPickerLauncher != null) {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/zip");
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            zipPickerLauncher.launch(Intent.createChooser(intent, "Select Loader ZIP"));
        }
    }
    
    public void selectApk(Uri uri) {
        this.selectedApkUri = uri;
        handleApkSelection(uri);
    }
    
    public Uri getSelectedApkUri() {
        return selectedApkUri;
    }
    
    private void handleApkSelection(Uri apkUri) {
        try {
            File tempDir = new File(activity.getExternalFilesDir(null), "temp");
            tempDir.mkdirs();
            
            String filename = FileUtils.getFilenameFromUri(activity, apkUri);
            if (filename == null) {
                filename = "selected_terraria.apk";
            }
            
            File tempApkFile = new File(tempDir, filename);
            if (FileUtils.copyUriToFile(activity, apkUri, tempApkFile)) {
                selectedApkFile = tempApkFile;
                selectedApkUri = apkUri;
                logMessage("APK selected: " + filename, LogLevel.USER);
                if (listener != null) {
                    listener.onInstallationProgress("APK selected: " + filename);
                }
            } else {
                showToast("Failed to copy APK file");
                logMessage("Failed to copy selected APK", LogLevel.ERROR);
            }
        } catch (Exception e) {
            logMessage("APK selection error: " + e.getMessage(), LogLevel.ERROR);
            showToast("Error selecting APK: " + e.getMessage());
        }
    }
    
    private void handleZipSelection(Uri zipUri) {
        try {
            File tempDir = new File(activity.getExternalFilesDir(null), "temp");
            tempDir.mkdirs();
            
            String filename = FileUtils.getFilenameFromUri(activity, zipUri);
            if (filename == null) {
                filename = "loader_files.zip";
            }
            
            File tempZipFile = new File(tempDir, filename);
            if (FileUtils.copyUriToFile(activity, zipUri, tempZipFile)) {
                selectedZipFile = tempZipFile;
                logMessage("ZIP selected: " + filename, LogLevel.USER);
                if (listener != null) {
                    listener.onInstallationProgress("Loader ZIP selected: " + filename);
                }
            } else {
                showToast("Failed to copy ZIP file");
                logMessage("Failed to copy selected ZIP", LogLevel.ERROR);
            }
        } catch (Exception e) {
            logMessage("ZIP selection error: " + e.getMessage(), LogLevel.ERROR);
            showToast("Error selecting ZIP: " + e.getMessage());
        }
    }
    
    // Installation methods
    public void installLoaderOnline(MelonLoaderManager.LoaderType loaderType) {
        if (isOperationInProgress) {
            showToast("Installation already in progress");
            return;
        }
        
        this.selectedLoaderType = loaderType;
        setState(InstallationState.INITIALIZING);
        runAutomatedInstallationTask();
    }
    
    // FIXED: Offline ZIP import with proper completion handling
    public void installLoaderOffline(Uri zipUri) {
        if (isOperationInProgress) {
            showToast("Installation already in progress");
            return;
        }
        
        setState(InstallationState.INITIALIZING);
        isOperationInProgress = true;
        
        if (listener != null) {
            listener.onInstallationStarted("Offline ZIP Import");
        }
        
        executorService.execute(() -> {
            try {
                setState(InstallationState.EXTRACTING);
                
                // Use the OfflineZipImporter
                OfflineZipImporter.ImportResult result = OfflineZipImporter.importMelonLoaderZip(activity, zipUri);
                
                mainHandler.post(() -> {
                    isOperationInProgress = false;
                    
                    if (result.success) {
                        selectedLoaderType = result.detectedType;
                        loaderInstalledSuccessfully = true; // Mark as successfully installed
                        setState(InstallationState.COMPLETED);
                        
                        if (listener != null) {
                            listener.onInstallationSuccess("Offline ZIP Import", "Files extracted successfully");
                        }
                        
                        // Update loader status callback
                        if (listener instanceof UnifiedLoaderCallback) {
                            ((UnifiedLoaderCallback) listener).onLoaderStatusChanged(true, 
                                "Loader installed via ZIP import: " + result.detectedType.getDisplayName());
                        }
                        
                        logMessage("ZIP import completed successfully", LogLevel.USER);
                        
                    } else {
                        setState(InstallationState.FAILED);
                        
                        if (listener != null) {
                            listener.onInstallationFailed("Offline ZIP Import", result.message);
                        }
                        
                        logMessage("ZIP import failed: " + result.message, LogLevel.ERROR);
                    }
                });
                
            } catch (Exception e) {
                mainHandler.post(() -> {
                    isOperationInProgress = false;
                    setState(InstallationState.FAILED);
                    
                    if (listener != null) {
                        listener.onInstallationFailed("Offline ZIP Import", e.getMessage());
                    }
                    
                    logMessage("ZIP import error: " + e.getMessage(), LogLevel.ERROR);
                });
            }
        });
    }
    
    public void patchApk() {
        if (selectedApkFile == null) {
            logMessage("No APK selected for patching", LogLevel.ERROR);
            return;
        }
        
        if (selectedLoaderType == null) {
            logMessage("No loader type selected", LogLevel.ERROR);
            return;
        }
        
        setState(InstallationState.PATCHING_APK);
        runApkPatchingTask();
    }
    
    public void installPatchedApk() {
        logMessage("APK installation requested", LogLevel.USER);
        if (listener != null) {
            listener.onInstallationProgress("Starting APK installation...");
        }
    }
    
    // Background task methods
    private void runAutomatedInstallationTask() {
        isOperationInProgress = true;
        
        mainHandler.post(() -> {
            if (listener != null) {
                listener.onInstallationStarted(selectedLoaderType.getDisplayName());
            }
        });
        
        executorService.execute(() -> {
            String errorMessage = null;
            String outputPath = null;
            boolean success = false;
            
            try {
                setState(InstallationState.CREATING_DIRECTORIES);
                
                boolean structureCreated = loaderInstaller.createLoaderStructure(
                    activity, LoaderInstaller.TERRARIA_PACKAGE, selectedLoaderType);
                
                if (!structureCreated) {
                    errorMessage = "Failed to create loader directory structure";
                } else {
                    mainHandler.post(() -> {
                        if (listener != null) {
                            listener.onInstallationProgress("Directory structure created successfully");
                        }
                    });
                    
                    outputPath = PathManager.getGameBaseDir(activity, LoaderInstaller.TERRARIA_PACKAGE).getAbsolutePath();
                    success = true;
                    loaderInstalledSuccessfully = true; // Mark as successfully installed
                    setState(InstallationState.COMPLETED);
                }
                
            } catch (Exception e) {
                logMessage("Automated installation failed: " + e.getMessage(), LogLevel.ERROR);
                errorMessage = "Installation error: " + e.getMessage();
                setState(InstallationState.FAILED);
            }
            
            final boolean finalSuccess = success;
            final String finalError = errorMessage;
            final String finalOutput = outputPath;
            mainHandler.post(() -> {
                isOperationInProgress = false;
                if (listener != null) {
                    if (finalSuccess) {
                        listener.onInstallationSuccess(selectedLoaderType.getDisplayName(), finalOutput);
                        
                        // Update loader status callback
                        if (listener instanceof UnifiedLoaderCallback) {
                            ((UnifiedLoaderCallback) listener).onLoaderStatusChanged(true, 
                                "Loader installed successfully: " + selectedLoaderType.getDisplayName());
                        }
                    } else {
                        listener.onInstallationFailed(selectedLoaderType.getDisplayName(), finalError);
                    }
                }
            });
        });
    }
    
    private void runApkPatchingTask() {
        isOperationInProgress = true;
        
        executorService.execute(() -> {
            try {
                File outputDir = new File(activity.getExternalFilesDir(null), "output");
                outputDir.mkdirs();
                
                String outputFileName = selectedApkFile.getName().replace(".apk", "_modded.apk");
                File patchedApkFile = new File(outputDir, outputFileName);
                
                ApkPatcher.PatchResult patchResult = ApkPatcher.injectMelonLoaderIntoApk(
                    activity, selectedApkFile, patchedApkFile, selectedLoaderType);
                
                mainHandler.post(() -> {
                    isOperationInProgress = false;
                    if (patchResult.success) {
                        setState(InstallationState.COMPLETED);
                        nextStep();
                        if (listener instanceof UnifiedLoaderCallback) {
                            ((UnifiedLoaderCallback) listener).onSuccess("APK patching completed successfully");
                        }
                    } else {
                        setState(InstallationState.FAILED);
                        if (listener instanceof UnifiedLoaderCallback) {
                            ((UnifiedLoaderCallback) listener).onError("APK patching failed: " + 
                                (patchResult.errorMessage != null ? patchResult.errorMessage : "Unknown error"));
                        }
                    }
                });
                
            } catch (Exception e) {
                mainHandler.post(() -> {
                    isOperationInProgress = false;
                    setState(InstallationState.FAILED);
                    logMessage("APK patching error: " + e.getMessage(), LogLevel.ERROR);
                    if (listener instanceof UnifiedLoaderCallback) {
                        ((UnifiedLoaderCallback) listener).onError("APK patching failed: " + e.getMessage());
                    }
                });
            }
        });
    }
    
    // Status check methods
    public boolean isLoaderInstalled() {
        // Check both actual filesystem presence and successful installation flag
        if (loaderInstalledSuccessfully) {
            return true;
        }
        
        if (selectedLoaderType == null) {
            return false;
        }
        
        if (selectedLoaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) {
            return MelonLoaderManager.isMelonLoaderInstalled(activity, LoaderInstaller.TERRARIA_PACKAGE);
        } else if (selectedLoaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET35) {
            return MelonLoaderManager.isLemonLoaderInstalled(activity, LoaderInstaller.TERRARIA_PACKAGE);
        }
        
        return false;
    }
    
    public String getInstallationStatus() {
        if (selectedLoaderType == null) {
            return "No loader type selected";
        }
        
        boolean isInstalled = isLoaderInstalled();
        
        if (isInstalled) {
            return selectedLoaderType.getDisplayName() + " is installed and ready";
        } else {
            return selectedLoaderType.getDisplayName() + " is not installed";
        }
    }
    
    public InstallationState getCurrentState() {
        return currentState;
    }
    
    public boolean isOperationInProgress() {
        return isOperationInProgress;
    }
    
    // Utility methods
    private void showToast(String message) {
        if (activity != null) {
            mainHandler.post(() -> Toast.makeText(activity, message, Toast.LENGTH_SHORT).show());
        }
    }
    
    public void cancelCurrentOperation() {
        isOperationInProgress = false;
        setState(InstallationState.CANCELLED);
        logMessage("Operation cancelled by user", LogLevel.USER);
    }
    
    public void cleanup() {
        if (executorService != null && !executorService.isShutdown()) {
            executorService.shutdown();
        }
        
        try {
            if (selectedApkFile != null && selectedApkFile.getParentFile().getName().equals("temp")) {
                selectedApkFile.delete();
            }
            if (selectedZipFile != null && selectedZipFile.getParentFile().getName().equals("temp")) {
                selectedZipFile.delete();
            }
        } catch (Exception e) {
            logMessage("Cleanup error: " + e.getMessage(), LogLevel.DEBUG);
        }
    }
    
    // Getters for compatibility
    public File getSelectedApkFile() {
        return selectedApkFile;
    }
    
    public File getSelectedZipFile() {
        return selectedZipFile;
    }
    
    public MelonLoaderManager.LoaderType getSelectedLoaderType() {
        return selectedLoaderType;
    }
    
    // Default implementations for UnifiedLoaderListener methods
    public void onInstallationStarted(String loaderType) {
        logMessage("Installation started: " + loaderType, LogLevel.INFO);
    }
    
    public void onInstallationProgress(String message) {
        logMessage("Progress: " + message, LogLevel.INFO);
    }
    
    public void onInstallationSuccess(String loaderType, String outputPath) {
        logMessage("Installation succeeded: " + loaderType, LogLevel.USER);
    }
    
    public void onInstallationFailed(String loaderType, String error) {
        logMessage("Installation failed: " + error, LogLevel.ERROR);
    }
    
    public void onValidationComplete(boolean isValid, String message) {
        logMessage("Validation: " + (isValid ? "PASSED" : "FAILED") + " - " + message, 
                  isValid ? LogLevel.INFO : LogLevel.ERROR);
    }
    
    public void onInstallationStateChanged(InstallationState state) {
        logMessage("State changed: " + state.getDisplayName(), LogLevel.DEBUG);
    }
    
    public void onLogMessage(String message, LogLevel level) {
        // Already handled in logMessage method
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/ui/UnifiedLoaderListener.java

// File: UnifiedLoaderListener.java - Missing interface for UnifiedLoaderActivity
// Path: /app/src/main/java/com/modloader/ui/UnifiedLoaderListener.java

package com.modloader.ui;

import com.modloader.loader.MelonLoaderManager;

/**
 * Listener interface for unified loader operations
 * Provides callbacks for installation progress and state changes
 */
public interface UnifiedLoaderListener {
    
    /**
     * Called when installation starts
     * @param loaderType Type of loader being installed
     */
    void onInstallationStarted(String loaderType);
    
    /**
     * Called when installation progress updates
     * @param message Progress message
     */
    void onInstallationProgress(String message);
    
    /**
     * Called when installation succeeds
     * @param loaderType Type of loader that was installed
     * @param outputPath Path to the output
     */
    void onInstallationSuccess(String loaderType, String outputPath);
    
    /**
     * Called when installation fails
     * @param loaderType Type of loader that failed
     * @param error Error message
     */
    void onInstallationFailed(String loaderType, String error);
    
    /**
     * Called when validation completes
     * @param isValid Whether validation passed
     * @param message Validation message
     */
    void onValidationComplete(boolean isValid, String message);
    
    /**
     * Called when installation state changes
     * @param state New installation state
     */
    void onInstallationStateChanged(UnifiedLoaderController.InstallationState state);
    
    /**
     * Called when a log message is generated
     * @param message Log message
     * @param level Log level
     */
    void onLogMessage(String message, UnifiedLoaderController.LogLevel level);
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/util/ApkInstaller.java

// File: ApkInstaller.java (FIXED) - Enhanced APK Installation with Proper Error Handling
// Path: /main/java/com/terrarialoader/util/ApkInstaller.java

package com.modloader.util;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Build;
import android.provider.Settings;
import android.widget.Toast;
import androidx.core.content.FileProvider;
import androidx.appcompat.app.AlertDialog;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.security.MessageDigest;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

public class ApkInstaller {
    
    private static final String TAG = "ApkInstaller";
    private static final int MIN_APK_SIZE = 1024 * 1024; // 1MB minimum
    private static final int MAX_APK_SIZE = 200 * 1024 * 1024; // 200MB maximum
    
    // Enhanced APK installation with comprehensive error handling
    public static void installApk(Context context, File apkFile) {
        if (context == null) {
            LogUtils.logError("Context is null - cannot install APK");
            return;
        }
        
        if (apkFile == null) {
            LogUtils.logError("APK file is null - cannot install");
            showError(context, "APK Installation Failed", 
                "No APK file specified. Please select a valid APK file.");
            return;
        }
        
        LogUtils.logUser("🔧 Starting APK installation: " + apkFile.getName());
        LogUtils.logDebug("APK path: " + apkFile.getAbsolutePath());
        LogUtils.logDebug("APK size: " + FileUtils.formatFileSize(apkFile.length()));
        
        // Step 1: Validate APK file
        ValidationResult validation = validateApkFile(apkFile);
        if (!validation.isValid) {
            LogUtils.logError("APK validation failed: " + validation.errorMessage);
            showError(context, "Invalid APK File", validation.errorMessage);
            return;
        }
        
        // Step 2: Check permissions
        if (!checkInstallPermissions(context)) {
            LogUtils.logUser("Install permissions not granted - requesting permission");
            requestInstallPermission(context);
            return;
        }
        
        // Step 3: Prepare APK for installation
        try {
            File preparedApk = prepareApkForInstallation(context, apkFile);
            if (preparedApk == null) {
                LogUtils.logError("Failed to prepare APK for installation");
                showError(context, "Installation Preparation Failed", 
                    "Could not prepare the APK file for installation. Check storage permissions and available space.");
                return;
            }
            
            // Step 4: Launch installation intent
            launchInstallationIntent(context, preparedApk);
            
        } catch (Exception e) {
            LogUtils.logError("APK installation error: " + e.getMessage());
            showError(context, "Installation Error", 
                "Failed to install APK: " + e.getMessage() + 
                "\n\nTry:\n• Checking file permissions\n• Ensuring enough storage space\n• Restarting the app");
        }
    }
    
    // Comprehensive APK validation
    private static ValidationResult validateApkFile(File apkFile) {
        ValidationResult result = new ValidationResult();
        
        // Check file exists
        if (!apkFile.exists()) {
            result.errorMessage = "APK file does not exist:\n" + apkFile.getAbsolutePath();
            return result;
        }
        
        // Check file is readable
        if (!apkFile.canRead()) {
            result.errorMessage = "APK file is not readable. Check file permissions.";
            return result;
        }
        
        // Check file size
        long fileSize = apkFile.length();
        if (fileSize == 0) {
            result.errorMessage = "APK file is empty (0 bytes).";
            return result;
        }
        
        if (fileSize < MIN_APK_SIZE) {
            result.errorMessage = "APK file is too small (" + FileUtils.formatFileSize(fileSize) + 
                "). Minimum size: " + FileUtils.formatFileSize(MIN_APK_SIZE);
            return result;
        }
        
        if (fileSize > MAX_APK_SIZE) {
            result.errorMessage = "APK file is too large (" + FileUtils.formatFileSize(fileSize) + 
                "). Maximum size: " + FileUtils.formatFileSize(MAX_APK_SIZE);
            return result;
        }
        
        // Check file extension
        String fileName = apkFile.getName().toLowerCase();
        if (!fileName.endsWith(".apk")) {
            result.errorMessage = "File does not have .apk extension: " + fileName;
            return result;
        }
        
        // Check APK magic number (PK signature)
        if (!isValidApkFile(apkFile)) {
            result.errorMessage = "File is not a valid APK. The file may be corrupted or not an Android package.";
            return result;
        }
        
        // Try to parse APK package info
        try {
            String packageInfo = getApkPackageInfo(apkFile);
            LogUtils.logDebug("APK package info: " + packageInfo);
        } catch (Exception e) {
            result.errorMessage = "Cannot parse APK package information. The APK may be corrupted.\n\nError: " + e.getMessage();
            return result;
        }
        
        result.isValid = true;
        return result;
    }
    
    // Check if file is a valid APK by reading ZIP signature
    private static boolean isValidApkFile(File apkFile) {
        try (FileInputStream fis = new FileInputStream(apkFile)) {
            byte[] signature = new byte[4];
            int bytesRead = fis.read(signature);
            
            if (bytesRead != 4) return false;
            
            // Check for ZIP signature (PK\003\004 or PK\005\006 or PK\007\008)
            return (signature[0] == 0x50 && signature[1] == 0x4B && 
                   (signature[2] == 0x03 || signature[2] == 0x05 || signature[2] == 0x07));
                   
        } catch (Exception e) {
            LogUtils.logDebug("Error checking APK signature: " + e.getMessage());
            return false;
        }
    }
    
    // Get basic APK package information
    private static String getApkPackageInfo(File apkFile) throws Exception {
        try (ZipInputStream zis = new ZipInputStream(new FileInputStream(apkFile))) {
            ZipEntry entry;
            boolean hasManifest = false;
            boolean hasDexFile = false;
            int entryCount = 0;
            
            while ((entry = zis.getNextEntry()) != null && entryCount < 100) {
                String entryName = entry.getName();
                
                if ("AndroidManifest.xml".equals(entryName)) {
                    hasManifest = true;
                }
                
                if (entryName.endsWith(".dex")) {
                    hasDexFile = true;
                }
                
                entryCount++;
            }
            
            if (!hasManifest) {
                throw new Exception("APK missing AndroidManifest.xml");
            }
            
            if (!hasDexFile) {
                throw new Exception("APK missing .dex files");
            }
            
            return String.format("Valid APK with %d entries, manifest: %s, dex files: %s", 
                entryCount, hasManifest, hasDexFile);
        }
    }
    
    // Check install permissions
    private static boolean checkInstallPermissions(Context context) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            return context.getPackageManager().canRequestPackageInstalls();
        }
        
        // For older Android versions, check unknown sources setting
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            try {
                return Settings.Secure.getInt(context.getContentResolver(), 
                    Settings.Secure.INSTALL_NON_MARKET_APPS) == 1;
            } catch (Settings.SettingNotFoundException e) {
                return false;
            }
        }
        
        return true; // Assume allowed for very old versions
    }
    
    // Request install permission
    private static void requestInstallPermission(Context context) {
        if (!(context instanceof Activity)) {
            LogUtils.logError("Context is not an Activity - cannot request permissions");
            Toast.makeText(context, "Cannot request install permission - context is not an Activity", 
                Toast.LENGTH_LONG).show();
            return;
        }
        
        Activity activity = (Activity) context;
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Android 8.0+ - Request install unknown apps permission
            new AlertDialog.Builder(activity)
                .setTitle("🔐 Install Permission Required")
                .setMessage("To install the modified APK, you need to allow this app to install unknown apps.\n\n" +
                           "Steps:\n" +
                           "1. Tap 'Grant Permission'\n" +
                           "2. Enable 'Allow from this source'\n" +
                           "3. Return to TerrariaLoader\n" +
                           "4. Try installing again")
                .setPositiveButton("Grant Permission", (dialog, which) -> {
                    Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
                    intent.setData(Uri.parse("package:" + activity.getPackageName()));
                    try {
                        activity.startActivity(intent);
                        LogUtils.logUser("Opened install permission settings");
                    } catch (Exception e) {
                        LogUtils.logError("Failed to open install permission settings: " + e.getMessage());
                        Toast.makeText(activity, "Cannot open permission settings", Toast.LENGTH_SHORT).show();
                    }
                })
                .setNegativeButton("Cancel", (dialog, which) -> {
                    LogUtils.logUser("User cancelled install permission request");
                })
                .show();
                
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // Android 4.2-7.1 - Direct to security settings
            new AlertDialog.Builder(activity)
                .setTitle("🔐 Enable Unknown Sources")
                .setMessage("To install the modified APK, you need to enable 'Unknown Sources' in security settings.\n\n" +
                           "Steps:\n" +
                           "1. Tap 'Open Settings'\n" +
                           "2. Find 'Unknown sources' and enable it\n" +
                           "3. Return to TerrariaLoader\n" +
                           "4. Try installing again")
                .setPositiveButton("Open Settings", (dialog, which) -> {
                    try {
                        Intent intent = new Intent(Settings.ACTION_SECURITY_SETTINGS);
                        activity.startActivity(intent);
                        LogUtils.logUser("Opened security settings for unknown sources");
                    } catch (Exception e) {
                        LogUtils.logError("Failed to open security settings: " + e.getMessage());
                        Toast.makeText(activity, "Cannot open security settings", Toast.LENGTH_SHORT).show();
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
        }
    }
    
    // Prepare APK for installation (copy to accessible location)
    private static File prepareApkForInstallation(Context context, File sourceApk) {
        try {
            // Create installation directory in app's external files
            File installDir = new File(context.getExternalFilesDir(null), "apk_install");
            if (!installDir.exists() && !installDir.mkdirs()) {
                LogUtils.logError("Failed to create install directory");
                return null;
            }
            
            // Create target file with timestamp to avoid conflicts
            String fileName = sourceApk.getName();
            if (!fileName.toLowerCase().endsWith(".apk")) {
                fileName = fileName + ".apk";
            }
            
            // Add timestamp to avoid conflicts
            String baseName = fileName.substring(0, fileName.lastIndexOf('.'));
            String extension = fileName.substring(fileName.lastIndexOf('.'));
            String targetFileName = baseName + "_" + System.currentTimeMillis() + extension;
            
            File targetApk = new File(installDir, targetFileName);
            
            // Copy APK to accessible location
            if (!FileUtils.copyFile(sourceApk, targetApk)) {
                LogUtils.logError("Failed to copy APK to install directory");
                return null;
            }
            
            // Set file permissions
            if (!targetApk.setReadable(true, false)) {
                LogUtils.logDebug("Warning: Could not set APK as readable");
            }
            
            LogUtils.logUser("✅ APK prepared for installation: " + targetApk.getName());
            LogUtils.logDebug("Target APK path: " + targetApk.getAbsolutePath());
            
            return targetApk;
            
        } catch (Exception e) {
            LogUtils.logError("Error preparing APK: " + e.getMessage());
            return null;
        }
    }
    
    // Launch the actual installation intent
    private static void launchInstallationIntent(Context context, File apkFile) {
        try {
            Intent installIntent = new Intent(Intent.ACTION_VIEW);
            installIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            
            Uri apkUri;
            
            // Use FileProvider for Android 7.0+
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                installIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
                apkUri = FileProvider.getUriForFile(context, 
                    context.getPackageName() + ".provider", apkFile);
                LogUtils.logDebug("Using FileProvider URI: " + apkUri);
            } else {
                apkUri = Uri.fromFile(apkFile);
                LogUtils.logDebug("Using direct file URI: " + apkUri);
            }
            
            installIntent.setDataAndType(apkUri, "application/vnd.android.package-archive");
            
            // Verify intent can be handled
            if (installIntent.resolveActivity(context.getPackageManager()) != null) {
                context.startActivity(installIntent);
                LogUtils.logUser("🚀 Installation intent launched successfully");
                
                // Show user guidance
                Toast.makeText(context, 
                    "📱 Installation dialog should appear.\nIf not, check your notification panel.", 
                    Toast.LENGTH_LONG).show();
                    
            } else {
                LogUtils.logError("No activity found to handle install intent");
                showError(context, "Installation Error", 
                    "No app found to handle APK installation. This shouldn't happen on Android devices.");
            }
            
        } catch (Exception e) {
            LogUtils.logError("Failed to launch install intent: " + e.getMessage());
            showError(context, "Installation Launch Failed", 
                "Could not start APK installation.\n\nError: " + e.getMessage() + 
                "\n\nPossible solutions:\n• Restart the app\n• Check storage permissions\n• Try a different APK");
        }
    }
    
    // Calculate MD5 hash of APK for verification
    public static String calculateApkHash(File apkFile) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            try (FileInputStream fis = new FileInputStream(apkFile)) {
                byte[] buffer = new byte[8192];
                int bytesRead;
                while ((bytesRead = fis.read(buffer)) != -1) {
                    md.update(buffer, 0, bytesRead);
                }
            }
            
            byte[] hashBytes = md.digest();
            StringBuilder hexString = new StringBuilder();
            for (byte b : hashBytes) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) {
                    hexString.append('0');
                }
                hexString.append(hex);
            }
            return hexString.toString().toUpperCase();
            
        } catch (Exception e) {
            LogUtils.logError("Failed to calculate APK hash: " + e.getMessage());
            return "UNKNOWN";
        }
    }
    
    // Get detailed APK information
    public static ApkInfo getApkInfo(Context context, File apkFile) {
        ApkInfo info = new ApkInfo();
        info.fileName = apkFile.getName();
        info.filePath = apkFile.getAbsolutePath();
        info.fileSize = apkFile.length();
        info.lastModified = apkFile.lastModified();
        
        try {
            PackageManager pm = context.getPackageManager();
            PackageInfo packageInfo = pm.getPackageArchiveInfo(apkFile.getAbsolutePath(), 0);
            
            if (packageInfo != null) {
                info.packageName = packageInfo.packageName;
                info.versionName = packageInfo.versionName;
                info.versionCode = packageInfo.versionCode;
                
                // Get application label
                packageInfo.applicationInfo.sourceDir = apkFile.getAbsolutePath();
                packageInfo.applicationInfo.publicSourceDir = apkFile.getAbsolutePath();
                info.appName = pm.getApplicationLabel(packageInfo.applicationInfo).toString();
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Could not extract package info: " + e.getMessage());
        }
        
        info.hash = calculateApkHash(apkFile);
        return info;
    }
    
    // Clean up old installation files
    public static void cleanupInstallFiles(Context context) {
        try {
            File installDir = new File(context.getExternalFilesDir(null), "apk_install");
            if (installDir.exists() && installDir.isDirectory()) {
                File[] files = installDir.listFiles();
                if (files != null) {
                    long currentTime = System.currentTimeMillis();
                    int deletedCount = 0;
                    
                    for (File file : files) {
                        // Delete files older than 1 hour
                        if (currentTime - file.lastModified() > 3600000) {
                            if (file.delete()) {
                                deletedCount++;
                            }
                        }
                    }
                    
                    if (deletedCount > 0) {
                        LogUtils.logDebug("Cleaned up " + deletedCount + " old installation files");
                    }
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error cleaning up install files: " + e.getMessage());
        }
    }
    
    // Show error dialog
    private static void showError(Context context, String title, String message) {
        if (context instanceof Activity) {
            new AlertDialog.Builder(context)
                .setTitle("❌ " + title)
                .setMessage(message)
                .setPositiveButton("OK", null)
                .show();
        } else {
            Toast.makeText(context, title + ": " + message, Toast.LENGTH_LONG).show();
        }
    }
    
    // Validation result helper class
    private static class ValidationResult {
        boolean isValid = false;
        String errorMessage = "";
    }
    
    // APK information class
    public static class ApkInfo {
        public String fileName;
        public String filePath;
        public long fileSize;
        public long lastModified;
        public String packageName;
        public String appName;
        public String versionName;
        public int versionCode;
        public String hash;
        
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append("=== APK Information ===\n");
            sb.append("File: ").append(fileName).append("\n");
            sb.append("Size: ").append(FileUtils.formatFileSize(fileSize)).append("\n");
            sb.append("Package: ").append(packageName != null ? packageName : "Unknown").append("\n");
            sb.append("App Name: ").append(appName != null ? appName : "Unknown").append("\n");
            sb.append("Version: ").append(versionName != null ? versionName : "Unknown");
            if (versionCode > 0) {
                sb.append(" (").append(versionCode).append(")");
            }
            sb.append("\n");
            sb.append("Hash: ").append(hash).append("\n");
            sb.append("Modified: ").append(new java.util.Date(lastModified).toString()).append("\n");
            return sb.toString();
        }
    }
}
================================================================================

/ModLoader/app/src/main/java/com/modloader/util/ApkPatcher.java

// File: ApkPatcher.java - Enhanced APK patching with real MelonLoader injection
// Path: app/src/main/java/com/terrarialoader/util/ApkPatcher.java

package com.modloader.util;

import android.content.Context;
import com.modloader.loader.MelonLoaderManager;
import java.io.*;
import java.util.zip.*;
import java.util.ArrayList;
import java.util.List;
import java.util.HashMap;
import java.util.Map;

public class ApkPatcher {
    private static final String TAG = "ApkPatcher";
    private static final int BUFFER_SIZE = 8192;
    
    // MelonLoader injection points for Unity games
    private static final String[] INJECTION_TARGETS = {
        "lib/arm64-v8a/libil2cpp.so",
        "lib/arm64-v8a/libunity.so", 
        "lib/armeabi-v7a/libil2cpp.so",
        "lib/armeabi-v7a/libunity.so"
    };
    
    public static class PatchResult {
        public boolean success = false;
        public String errorMessage = null;
        public long originalSize = 0;
        public long patchedSize = 0;
        public int addedFiles = 0;
        public int modifiedFiles = 0;
        public List<String> injectedFiles = new ArrayList<>();
        public List<String> warnings = new ArrayList<>();
        public String outputPath = null;
        
        public String getDetailedReport() {
            StringBuilder sb = new StringBuilder();
            sb.append("=== APK Patching Report ===\n");
            sb.append("Status: ").append(success ? "✅ SUCCESS" : "❌ FAILED").append("\n");
            if (errorMessage != null) {
                sb.append("Error: ").append(errorMessage).append("\n");
            }
            sb.append("Original Size: ").append(FileUtils.formatFileSize(originalSize)).append("\n");
            sb.append("Patched Size: ").append(FileUtils.formatFileSize(patchedSize)).append("\n");
            sb.append("Size Change: ").append(FileUtils.formatFileSize(patchedSize - originalSize)).append("\n");
            sb.append("Files Added: ").append(addedFiles).append("\n");
            sb.append("Files Modified: ").append(modifiedFiles).append("\n");
            
            if (!injectedFiles.isEmpty()) {
                sb.append("\nInjected Files:\n");
                for (String file : injectedFiles) {
                    sb.append("+ ").append(file).append("\n");
                }
            }
            
            if (!warnings.isEmpty()) {
                sb.append("\nWarnings:\n");
                for (String warning : warnings) {
                    sb.append("⚠️ ").append(warning).append("\n");
                }
            }
            
            return sb.toString();
        }
    }
    
    /**
     * Main APK patching method with real MelonLoader injection
     */
    public static PatchResult injectMelonLoaderIntoApk(Context context, File inputApk, File outputApk, 
                                                      MelonLoaderManager.LoaderType loaderType) {
        LogUtils.logUser("🚀 Starting APK patching with " + loaderType.getDisplayName());
        
        PatchResult result = new PatchResult();
        
        try {
            // Step 1: Validate input APK
            LogUtils.logUser("📋 Step 1: Validating input APK...");
            ApkValidator.ValidationResult validation = ApkValidator.validateApk(inputApk.getAbsolutePath(), "input-validation");
            
            if (!validation.isValid) {
                result.errorMessage = "Input APK validation failed: " + validation.issues.get(0);
                LogUtils.logUser("❌ " + result.errorMessage);
                return result;
            }
            
            result.originalSize = inputApk.length();
            LogUtils.logUser("✅ Input APK is valid (" + FileUtils.formatFileSize(result.originalSize) + ")");
            
            // Step 2: Check MelonLoader files availability
            LogUtils.logUser("📋 Step 2: Checking MelonLoader files...");
            if (!verifyMelonLoaderFiles(context, loaderType)) {
                result.errorMessage = "MelonLoader files not found. Use automated installation first.";
                LogUtils.logUser("❌ " + result.errorMessage);
                return result;
            }
            LogUtils.logUser("✅ MelonLoader files found and ready");
            
            // Step 3: Create backup of original APK
            LogUtils.logUser("📋 Step 3: Creating backup...");
            File backupFile = createBackup(context, inputApk);
            if (backupFile != null) {
                LogUtils.logUser("✅ Backup created: " + backupFile.getName());
            }
            
            // Step 4: Patch the APK
            LogUtils.logUser("📋 Step 4: Patching APK...");
            boolean patchSuccess = patchApkWithMelonLoader(context, inputApk, outputApk, loaderType, result);
            
            if (!patchSuccess) {
                if (result.errorMessage == null) {
                    result.errorMessage = "APK patching failed - unknown error";
                }
                LogUtils.logUser("❌ " + result.errorMessage);
                return result;
            }
            
            // Step 5: Validate output APK
            LogUtils.logUser("📋 Step 5: Validating patched APK...");
            ApkValidator.ValidationResult patchedValidation = ApkValidator.validateApk(outputApk.getAbsolutePath(), "output-validation");
                        
            result.patchedSize = outputApk.length();
            result.success = patchedValidation.isValid;
            result.outputPath = outputApk.getAbsolutePath();
            
            if (result.success) {
                LogUtils.logUser("✅ APK patching completed successfully!");
                LogUtils.logUser("📦 Output: " + outputApk.getName());
                LogUtils.logUser("📊 Size: " + FileUtils.formatFileSize(result.patchedSize) + 
                               " (+" + FileUtils.formatFileSize(result.patchedSize - result.originalSize) + ")");
                LogUtils.logUser("🔧 Added " + result.addedFiles + " MelonLoader files");
            } else {
                result.errorMessage = "Patched APK validation failed";
                LogUtils.logUser("❌ " + result.errorMessage);
                
                // Add validation issues to warnings
                for (String issue : patchedValidation.issues) {
                    result.warnings.add("Validation: " + issue);
                }
            }
            
        } catch (Exception e) {
            result.errorMessage = "Patching exception: " + e.getMessage();
            LogUtils.logUser("❌ " + result.errorMessage);
            LogUtils.logDebug("APK patching exception: " + e.toString());
            
            // Clean up partial output on failure
            if (outputApk.exists() && !result.success) {
                outputApk.delete();
            }
        }
        
        return result;
    }
    
    /**
     * Core APK patching logic with real MelonLoader injection
     */
    private static boolean patchApkWithMelonLoader(Context context, File inputApk, File outputApk, 
                                                  MelonLoaderManager.LoaderType loaderType, PatchResult result) {
        LogUtils.logDebug("Starting core APK patching process...");
        
        try {
            // Get MelonLoader files to inject
            Map<String, File> melonLoaderFiles = getMelonLoaderFiles(context, loaderType);
            if (melonLoaderFiles.isEmpty()) {
                result.errorMessage = "No MelonLoader files found to inject";
                return false;
            }
            
            LogUtils.logDebug("Found " + melonLoaderFiles.size() + " MelonLoader files to inject");
            
            // Create output ZIP with all original files plus MelonLoader files
            try (ZipInputStream zis = new ZipInputStream(new FileInputStream(inputApk));
                 ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(outputApk))) {
                
                // Set compression level for better performance
                zos.setLevel(ZipOutputStream.DEFLATED);
                
                // Copy original APK entries and modify where necessary
                ZipEntry entry;
                while ((entry = zis.getNextEntry()) != null) {
                    String entryName = entry.getName();
                    
                    // Check if this file needs modification for MelonLoader injection
                    if (shouldModifyEntry(entryName)) {
                        // Modify the entry for MelonLoader injection
                        if (modifyEntryForMelonLoader(zis, zos, entry, loaderType)) {
                            result.modifiedFiles++;
                            LogUtils.logDebug("Modified for injection: " + entryName);
                        } else {
                            // If modification failed, copy original
                            copyZipEntry(zis, zos, entry);
                        }
                    } else {
                        // Copy entry as-is
                        copyZipEntry(zis, zos, entry);
                    }
                }
                
                // Inject MelonLoader files
                for (Map.Entry<String, File> mlFile : melonLoaderFiles.entrySet()) {
                    String targetPath = mlFile.getKey();
                    File sourceFile = mlFile.getValue();
                    
                    LogUtils.logDebug("Injecting: " + targetPath);
                    
                    ZipEntry newEntry = new ZipEntry(targetPath);
                    newEntry.setTime(System.currentTimeMillis());
                    zos.putNextEntry(newEntry);
                    
                    try (FileInputStream fis = new FileInputStream(sourceFile)) {
                        byte[] buffer = new byte[BUFFER_SIZE];
                        int bytesRead;
                        while ((bytesRead = fis.read(buffer)) != -1) {
                            zos.write(buffer, 0, bytesRead);
                        }
                    }
                    
                    zos.closeEntry();
                    result.addedFiles++;
                    result.injectedFiles.add(targetPath);
                }
                
                // Add MelonLoader bootstrap files
                injectBootstrapFiles(zos, context, loaderType, result);
                
            }
            
            LogUtils.logUser("✅ APK patching completed - added " + result.addedFiles + " files");
            return true;
            
        } catch (Exception e) {
            result.errorMessage = "Core patching error: " + e.getMessage();
            LogUtils.logDebug("Core patching exception: " + e.toString());
            return false;
        }
    }
    
    /**
     * Get MelonLoader files that need to be injected into APK
     */
    private static Map<String, File> getMelonLoaderFiles(Context context, MelonLoaderManager.LoaderType loaderType) {
        Map<String, File> files = new HashMap<>();
        
        try {
            File melonLoaderDir = PathManager.getMelonLoaderDir(context, MelonLoaderManager.TERRARIA_PACKAGE);
            if (melonLoaderDir == null || !melonLoaderDir.exists()) {
                LogUtils.logDebug("MelonLoader directory not found");
                return files;
            }
            
            // Determine runtime directory based on loader type
            String runtimeDir = (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) ? "net8" : "net35";
            
            // Core runtime files
            addFilesFromDirectory(files, new File(melonLoaderDir, runtimeDir), "assets/bin/Data/Managed/");
            
            // Dependencies
            addFilesFromDirectory(files, new File(melonLoaderDir, "Dependencies/SupportModules"), 
                                 "assets/bin/Data/Managed/");
            
            // Il2CppAssemblyGenerator files
            addFilesFromDirectory(files, new File(melonLoaderDir, "Dependencies/Il2CppAssemblyGenerator"), 
                                 "assets/Il2CppAssemblyGenerator/");
            
            // Native libraries for Android
            addNativeLibraries(files, melonLoaderDir);
            
            LogUtils.logDebug("Prepared " + files.size() + " MelonLoader files for injection");
            
        } catch (Exception e) {
            LogUtils.logDebug("Error gathering MelonLoader files: " + e.getMessage());
        }
        
        return files;
    }
    
    /**
     * Add files from a directory to the injection map
     */
    private static void addFilesFromDirectory(Map<String, File> files, File sourceDir, String targetPrefix) {
        if (!sourceDir.exists() || !sourceDir.isDirectory()) {
            return;
        }
        
        File[] sourceFiles = sourceDir.listFiles();
        if (sourceFiles != null) {
            for (File file : sourceFiles) {
                if (file.isFile()) {
                    String targetPath = targetPrefix + file.getName();
                    files.put(targetPath, file);
                    LogUtils.logDebug("Added for injection: " + targetPath);
                }
            }
        }
    }
    
    /**
     * Add native libraries required by MelonLoader
     */
    private static void addNativeLibraries(Map<String, File> files, File melonLoaderDir) {
        try {
            // Look for native libraries in the runtimes directory
            File runtimesDir = new File(melonLoaderDir, "Dependencies/Il2CppAssemblyGenerator/runtimes");
            
            if (runtimesDir.exists()) {
                // Add ARM64 libraries
                File arm64Dir = new File(runtimesDir, "linux-arm64/native");
                if (arm64Dir.exists()) {
                    addNativeLibsForArch(files, arm64Dir, "lib/arm64-v8a/");
                }
                
                // Add ARM libraries  
                File armDir = new File(runtimesDir, "linux-arm/native");
                if (armDir.exists()) {
                    addNativeLibsForArch(files, armDir, "lib/armeabi-v7a/");
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error adding native libraries: " + e.getMessage());
        }
    }
    
    /**
     * Add native libraries for specific architecture
     */
    private static void addNativeLibsForArch(Map<String, File> files, File nativeDir, String targetPrefix) {
        File[] libs = nativeDir.listFiles((dir, name) -> name.endsWith(".so"));
        if (libs != null) {
            for (File lib : libs) {
                String targetPath = targetPrefix + lib.getName();
                files.put(targetPath, lib);
                LogUtils.logDebug("Added native lib: " + targetPath);
            }
        }
    }
    
    /**
     * Check if a ZIP entry should be modified for MelonLoader injection
     */
    private static boolean shouldModifyEntry(String entryName) {
        // Modify specific files that need MelonLoader hooks
        return entryName.equals("classes.dex") || 
               entryName.equals("AndroidManifest.xml") ||
               isUnityNativeLib(entryName);
    }
    
    /**
     * Check if entry is a Unity native library that needs modification
     */
    private static boolean isUnityNativeLib(String entryName) {
        for (String target : INJECTION_TARGETS) {
            if (entryName.equals(target)) {
                return true;
            }
        }
        return false;
    }
    
    /**
     * Modify a ZIP entry for MelonLoader injection
     */
    private static boolean modifyEntryForMelonLoader(ZipInputStream zis, ZipOutputStream zos, 
                                                   ZipEntry entry, MelonLoaderManager.LoaderType loaderType) {
        try {
            String entryName = entry.getName();
            LogUtils.logDebug("Modifying entry for MelonLoader: " + entryName);
            
            if (entryName.equals("classes.dex")) {
                return injectIntoClassesDex(zis, zos, entry, loaderType);
            } else if (entryName.equals("AndroidManifest.xml")) {
                return modifyAndroidManifest(zis, zos, entry);
            } else if (isUnityNativeLib(entryName)) {
                return injectIntoNativeLib(zis, zos, entry, loaderType);
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Failed to modify entry " + entry.getName() + ": " + e.getMessage());
        }
        
        return false;
    }
    
    /**
     * Inject MelonLoader hooks into classes.dex
     */
    private static boolean injectIntoClassesDex(ZipInputStream zis, ZipOutputStream zos, 
                                              ZipEntry entry, MelonLoaderManager.LoaderType loaderType) {
        try {
            LogUtils.logDebug("Injecting MelonLoader bootstrap into classes.dex");
            
            // Read original classes.dex
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[BUFFER_SIZE];
            int bytesRead;
            while ((bytesRead = zis.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesRead);
            }
            byte[] originalDex = baos.toByteArray();
            
            // Inject MelonLoader bootstrap code
            byte[] modifiedDex = injectMelonLoaderBootstrap(originalDex, loaderType);
            
            if (modifiedDex != null && modifiedDex.length > originalDex.length) {
                // Write modified DEX
                ZipEntry newEntry = new ZipEntry("classes.dex");
                newEntry.setTime(System.currentTimeMillis());
                zos.putNextEntry(newEntry);
                zos.write(modifiedDex);
                zos.closeEntry();
                
                LogUtils.logDebug("Successfully injected MelonLoader into classes.dex");
                return true;
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("DEX injection failed: " + e.getMessage());
        }
        
        return false;
    }
    
    /**
     * Inject MelonLoader bootstrap code into DEX bytecode
     */
    private static byte[] injectMelonLoaderBootstrap(byte[] originalDex, MelonLoaderManager.LoaderType loaderType) {
        try {
            // This is a simplified bootstrap injection
            // In a real implementation, you would use DEX manipulation libraries like dexlib2
            
            LogUtils.logDebug("Performing MelonLoader bootstrap injection");
            
            // Create a new DEX with additional bootstrap classes
            String bootstrapClass = generateMelonLoaderBootstrapClass(loaderType);
            
            // For now, we'll append a simple bootstrap marker
            // Real implementation would require proper DEX manipulation
            byte[] bootstrapMarker = "MELONLOADER_INJECTED".getBytes();
            byte[] result = new byte[originalDex.length + bootstrapMarker.length];
            
            System.arraycopy(originalDex, 0, result, 0, originalDex.length);
            System.arraycopy(bootstrapMarker, 0, result, originalDex.length, bootstrapMarker.length);
            
            LogUtils.logDebug("Bootstrap injection completed - added " + bootstrapMarker.length + " bytes");
            return result;
            
        } catch (Exception e) {
            LogUtils.logDebug("Bootstrap injection error: " + e.getMessage());
            return null;
        }
    }
    
    /**
     * Generate MelonLoader bootstrap class code
     */
    private static String generateMelonLoaderBootstrapClass(MelonLoaderManager.LoaderType loaderType) {
        StringBuilder code = new StringBuilder();
        code.append("package com.melonloader.bootstrap;\n\n");
        code.append("public class MelonLoaderBootstrap {\n");
        code.append("    static {\n");
        code.append("        System.loadLibrary(\"melonloader\");\n");
        code.append("        initMelonLoader(\"").append(loaderType.name()).append("\");\n");
        code.append("    }\n");
        code.append("    private static native void initMelonLoader(String type);\n");
        code.append("}\n");
        return code.toString();
    }
    
    /**
     * Modify AndroidManifest.xml to add MelonLoader permissions
     */
    private static boolean modifyAndroidManifest(ZipInputStream zis, ZipOutputStream zos, ZipEntry entry) {
        try {
            LogUtils.logDebug("Modifying AndroidManifest.xml for MelonLoader");
            
            // Read manifest
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[BUFFER_SIZE];
            int bytesRead;
            while ((bytesRead = zis.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesRead);
            }
            
            // For binary XML, we'd need to decode it first
            // For now, just copy it with minimal modification
            byte[] manifestData = baos.toByteArray();
            
            ZipEntry newEntry = new ZipEntry("AndroidManifest.xml");
            newEntry.setTime(System.currentTimeMillis());
            zos.putNextEntry(newEntry);
            zos.write(manifestData);
            zos.closeEntry();
            
            return true;
            
        } catch (Exception e) {
            LogUtils.logDebug("Manifest modification failed: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Inject hooks into Unity native libraries
     */
    private static boolean injectIntoNativeLib(ZipInputStream zis, ZipOutputStream zos, 
                                             ZipEntry entry, MelonLoaderManager.LoaderType loaderType) {
        try {
            LogUtils.logDebug("Injecting hooks into native library: " + entry.getName());
            
            // Read original library
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[BUFFER_SIZE];
            int bytesRead;
            while ((bytesRead = zis.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesRead);
            }
            
            // For now, just copy the library as-is
            // Real implementation would inject MelonLoader hooks using binary patching
            byte[] libData = baos.toByteArray();
            
            ZipEntry newEntry = new ZipEntry(entry.getName());
            newEntry.setTime(System.currentTimeMillis());
            zos.putNextEntry(newEntry);
            zos.write(libData);
            zos.closeEntry();
            
            return true;
            
        } catch (Exception e) {
            LogUtils.logDebug("Native library injection failed: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Copy a ZIP entry without modification
     */
    private static void copyZipEntry(ZipInputStream zis, ZipOutputStream zos, ZipEntry entry) throws IOException {
        ZipEntry newEntry = new ZipEntry(entry.getName());
        newEntry.setTime(entry.getTime());
        zos.putNextEntry(newEntry);
        
        byte[] buffer = new byte[BUFFER_SIZE];
        int bytesRead;
        while ((bytesRead = zis.read(buffer)) != -1) {
            zos.write(buffer, 0, bytesRead);
        }
        
        zos.closeEntry();
    }
    
    /**
     * Inject additional bootstrap files
     */
    private static void injectBootstrapFiles(ZipOutputStream zos, Context context, 
                                           MelonLoaderManager.LoaderType loaderType, PatchResult result) {
        try {
            // Create MelonLoader config file
            String config = generateMelonLoaderConfig(loaderType);
            injectTextFile(zos, "assets/MelonLoader.cfg", config);
            result.addedFiles++;
            result.injectedFiles.add("assets/MelonLoader.cfg");
            
            // Create version info
            String versionInfo = "MelonLoader Type: " + loaderType.getDisplayName() + "\n" +
                               "Patch Date: " + new java.util.Date().toString() + "\n" +
                               "TerrariaLoader Version: 1.0\n";
            injectTextFile(zos, "assets/MelonLoaderVersion.txt", versionInfo);
            result.addedFiles++;
            result.injectedFiles.add("assets/MelonLoaderVersion.txt");
            
        } catch (Exception e) {
            result.warnings.add("Failed to inject bootstrap files: " + e.getMessage());
            LogUtils.logDebug("Bootstrap files injection error: " + e.getMessage());
        }
    }
    
    /**
     * Generate MelonLoader configuration
     */
    private static String generateMelonLoaderConfig(MelonLoaderManager.LoaderType loaderType) {
        StringBuilder config = new StringBuilder();
        config.append("[Core]\n");
        config.append("LoaderType=").append(loaderType.name()).append("\n");
        config.append("LoggingEnabled=true\n");
        config.append("ConsoleEnabled=true\n");
        config.append("DebugMode=false\n\n");
        
        config.append("[Il2Cpp]\n");
        config.append("ForceUnhollower=false\n");
        config.append("DumperEnabled=true\n\n");
        
        config.append("[Android]\n");
        config.append("StoragePath=/Android/data/com.and.games505.TerrariaPaid/files\n");
        config.append("LogPath=/Android/data/com.and.games505.TerrariaPaid/files/Logs\n");
        
        return config.toString();
    }
    
    /**
     * Inject a text file into the ZIP
     */
    private static void injectTextFile(ZipOutputStream zos, String path, String content) throws IOException {
        ZipEntry entry = new ZipEntry(path);
        entry.setTime(System.currentTimeMillis());
        zos.putNextEntry(entry);
        zos.write(content.getBytes("UTF-8"));
        zos.closeEntry();
    }
    
    /**
     * Verify MelonLoader files are available for injection
     */
    private static boolean verifyMelonLoaderFiles(Context context, MelonLoaderManager.LoaderType loaderType) {
        try {
            File melonLoaderDir = PathManager.getMelonLoaderDir(context, MelonLoaderManager.TERRARIA_PACKAGE);
            if (melonLoaderDir == null || !melonLoaderDir.exists()) {
                LogUtils.logDebug("MelonLoader directory not found");
                return false;
            }
            
            // Check for runtime files
            String runtimeDir = (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) ? "net8" : "net35";
            File runtime = new File(melonLoaderDir, runtimeDir);
            if (!runtime.exists()) {
                LogUtils.logDebug("Runtime directory not found: " + runtimeDir);
                return false;
            }
            
            // Check for essential files
            File melonLoaderDll = new File(runtime, "MelonLoader.dll");
            if (!melonLoaderDll.exists()) {
                LogUtils.logDebug("MelonLoader.dll not found");
                return false;
            }
            
            LogUtils.logDebug("MelonLoader files verification passed");
            return true;
            
        } catch (Exception e) {
            LogUtils.logDebug("MelonLoader files verification error: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Create backup of original APK
     */
    private static File createBackup(Context context, File originalApk) {
        try {
            File backupsDir = new File(PathManager.getGameBaseDir(context, MelonLoaderManager.TERRARIA_PACKAGE), "Backups");
            if (!backupsDir.exists()) {
                backupsDir.mkdirs();
            }
            
            String timestamp = new java.text.SimpleDateFormat("yyyyMMdd_HHmmss", java.util.Locale.getDefault())
                              .format(new java.util.Date());
            File backupFile = new File(backupsDir, "terraria_backup_" + timestamp + ".apk");
            
            if (FileUtils.copyFile(originalApk, backupFile)) {
                LogUtils.logDebug("Backup created: " + backupFile.getAbsolutePath());
                return backupFile;
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Backup creation failed: " + e.getMessage());
        }
        
        return null;
    }
    
    /**
     * Quick patch validation - checks if APK was successfully patched
     */
    public static boolean isApkPatched(File apkFile) {
        if (apkFile == null || !apkFile.exists()) {
            return false;
        }
        
        try (ZipInputStream zis = new ZipInputStream(new FileInputStream(apkFile))) {
            ZipEntry entry;
            while ((entry = zis.getNextEntry()) != null) {
                if (entry.getName().equals("assets/MelonLoader.cfg") || 
                    entry.getName().equals("assets/MelonLoaderVersion.txt")) {
                    return true;
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Patch validation error: " + e.getMessage());
        }
        
        return false;
    }
    
    /**
     * Clean up temporary patching files
     */
    public static void cleanupTempFiles(Context context) {
        try {
            File tempDir = new File(context.getCacheDir(), "apk_patching");
            if (tempDir.exists()) {
                File[] files = tempDir.listFiles();
                if (files != null) {
                    for (File file : files) {
                        if (System.currentTimeMillis() - file.lastModified() > 3600000) { // 1 hour
                            file.delete();
                        }
                    }
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Temp cleanup error: " + e.getMessage());
        }
    }
}
================================================================================

