name: Build APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Setup Java 21
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'

      - name: Set Android SDK Environment
        run: |
          echo "ANDROID_HOME=/usr/local/lib/android/sdk" >> $GITHUB_ENV
          echo "ANDROID_SDK_ROOT=/usr/local/lib/android/sdk" >> $GITHUB_ENV

      - name: Prepare Web Assets & Install Capacitor
        run: |
          mkdir -p www
          cp -r *.html www/ 2>/dev/null || true
          cp -r *.js www/ 2>/dev/null || true
          cp -r *.css www/ 2>/dev/null || true
          if [ ! -f "www/index.html" ]; then
            first_html=$(ls www/*.html 2>/dev/null | head -n 1)
            if [ -n "$first_html" ]; then
              cp "$first_html" www/index.html
            fi
          fi
          npm init -y
          npm install @capacitor/core@6 @capacitor/cli@6 @capacitor/android@6
          npx cap init "ADNOC Order" "com.adnoc.order" --web-dir www
          npx cap add android
          npx cap sync android

      - name: Build Android APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: ADNOC-Order-APK
          path: android/app/build/outputs/apk/debug/app-debug.apk
