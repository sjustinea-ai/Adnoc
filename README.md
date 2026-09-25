name: Build APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Setup Java 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Prepare Web Directory
        run: |
          mkdir -p www
          cp -r *.html www/ 2>/dev/null || true
          if [ ! -f "www/index.html" ]; then
            first_html=$(ls www/*.html 2>/dev/null | head -n 1)
            if [ -n "$first_html" ]; then
              cp "$first_html" www/index.html
            fi
          fi

      - name: Build APK using Cordova
        run: |
          npm install -g cordova
          cordova create android-app com.adnoc.order "ADNOC Order"
          cd android-app
          cp -r ../www/* www/
          cordova platform add android@12.0.0
          cordova build android --debug

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: ADNOC-Order-APK
          path: android-app/platforms/android/app/build/outputs/apk/debug/app-debug.apk
