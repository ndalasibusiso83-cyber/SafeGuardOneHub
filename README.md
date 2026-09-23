name: Build APK
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - uses: android-actions/setup-android@v3

      - name: Build SafeGuard APK
        run: |
          mkdir -p app/src/main/java/com/safeguard/onehub
          mkdir -p app/src/main/res/layout

          cat > settings.gradle << 'EOF'
          include ':app'
          EOF

          cat > app/build.gradle << 'EOF'
          plugins { id 'com.android.application' }
          android {
            namespace 'com.safeguard.onehub'
            compileSdk 34
            defaultConfig {
              applicationId "com.safeguard.onehub"
              minSdk 24
              targetSdk 34
              versionCode 1
              versionName "1.0"
            }
            buildTypes { debug { minifyEnabled false } }
          }
          dependencies {
            implementation 'androidx.appcompat:appcompat:1.6.1'
            implementation 'com.google.android.material:material:1.11.0'
          }
          EOF

          cat > app/src/main/AndroidManifest.xml << 'EOF'
          <?xml version="1.0" encoding="utf-8"?>
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">
            <uses-permission android:name="android.permission.INTERNET"/>
            <application android:allowBackup="true" android:label="SafeGuard One Hub" android:theme="@android:style/Theme.Light">
              <activity android:name=".MainActivity" android:exported="true">
                <intent-filter><action android:name="android.intent.action.MAIN"/><category android:name="android.intent.category.LAUNCHER"/></intent-filter>
              </activity>
            </application>
          </manifest>
          EOF

          cat > app/src/main/java/com/safeguard/onehub/MainActivity.java << 'EOF'
          package com.safeguard.onehub;
          import android.os.Bundle;
          import androidx.appcompat.app.AppCompatActivity;
          import android.webkit.*;
          public class MainActivity extends AppCompatActivity {
            WebView wv;
            protected void onCreate(Bundle b){
              super.onCreate(b);
              wv=new WebView(this); setContentView(wv);
              wv.getSettings().setJavaScriptEnabled(true);
              wv.setWebViewClient(new WebViewClient(){
                public boolean shouldOverrideUrlLoading(WebView v, WebResourceRequest r){
                  String u=r.getUrl().toString().toLowerCase();
                  if(u.contains("pornhub")||u.contains("xvideos")||u.contains("xnxx")||u.contains("xhamster")||u.contains("onlyfans")){
                    v.loadData("<html><body style='text-align:center;padding-top:50px'><h1>🛡️ Blocked by SafeGuard NDALA</h1><p>Protected by Family ID: NDALA</p></body></html>","text/html","UTF-8");
                    return true;
                  }
                  return false;
                }
              });
              wv.loadUrl("https://www.google.com");
            }
          }
          EOF

          gradle init --type basic --dsl groovy --overwrite 2>/dev/null || true
          echo "rootProject.name='SafeGuardOneHub'" > settings.gradle
          echo "include ':app'" >> settings.gradle
          gradle :app:assembleDebug

      - uses: actions/upload-artifact@v4
        with:
          name: SafeGuard-APK-NDALA-FIXED
          path: app/build/outputs/apk/debug/*.apk
