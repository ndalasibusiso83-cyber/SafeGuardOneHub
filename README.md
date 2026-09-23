name: Build APK
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '17', distribution: 'temurin' }
      - uses: android-actions/setup-android@v3
      - name: Build SafeGuard NDALA APK
        run: |
          rm -rf app build.gradle settings.gradle gradle gradlew
          mkdir -p app/src/main/java/com/safeguard/onehub
          echo "include ':app'" > settings.gradle
          cat > app/build.gradle << 'E1'
          plugins { id 'com.android.application' }
          android { namespace 'com.safeguard.onehub'; compileSdk 34; defaultConfig { applicationId "com.safeguard.onehub"; minSdk 24; targetSdk 34; versionCode 1; versionName "1.0" } buildTypes { debug { minifyEnabled false } } }
          dependencies { implementation 'androidx.appcompat:appcompat:1.6.1' }
          E1
          cat > app/src/main/AndroidManifest.xml << 'E2'
          <?xml version="1.0" encoding="utf-8"?><manifest xmlns:android="http://schemas.android.com/apk/res/android"><uses-permission android:name="android.permission.INTERNET"/><application android:label="SafeGuard One Hub"><activity android:name=".MainActivity" android:exported="true"><intent-filter><action android:name="android.intent.action.MAIN"/><category android:name="android.intent.category.LAUNCHER"/></intent-filter></activity></application></manifest>
          E2
          cat > app/src/main/java/com/safeguard/onehub/MainActivity.java << 'E3'
          package com.safeguard.onehub; import android.os.Bundle; import androidx.appcompat.app.AppCompatActivity; import android.webkit.*; public class MainActivity extends AppCompatActivity{WebView w;protected void onCreate(Bundle b){super.onCreate(b);w=new WebView(this);setContentView(w);w.getSettings().setJavaScriptEnabled(true);w.setWebViewClient(new WebViewClient(){public boolean shouldOverrideUrlLoading(WebView v, WebResourceRequest r){String u=r.getUrl().toString().toLowerCase();if(u.contains("pornhub")||u.contains("xvideos")||u.contains("xnxx")){v.loadData("<h1>Blocked by SafeGuard NDALA</h1>","text/html","UTF-8");return true;}return false;}});w.loadUrl("https://www.google.com");}}
          E3
          wget https://services.gradle.org/distributions/gradle-8.7-bin.zip
          unzip -q gradle-8.7-bin.zip
         ./gradle-8.7/bin/gradle wrapper
        ./gradlew app:assembleDebug --stacktrace
      - uses: actions/upload-artifact@v4
        with:
          name: SafeGuard-NDALA-APK-FINAL
          path: app/build/outputs/apk/debug/*.apk
