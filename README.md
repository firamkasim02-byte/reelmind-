name: Build ReelMind APK

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Generate Flutter App & Build APK
        run: |
          mkdir build_app
          cd build_app
          flutter create --platforms=android --org com.reelmind --project-name reelmind .
          cp ../app_code.dart lib/main.dart
          flutter build apk --release

      - name: Upload Ready-to-Install APK
        uses: actions/upload-artifact@v4
        with:
          name: ReelMind-Android-APK
          path: build_app/build/app/outputs/flutter-apk/app-release.apk
