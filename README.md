# Flutter Build APK

Workflow GitHub Actions untuk membangun project Flutter menjadi APK secara otomatis.

Project ini dirancang agar dapat menerima project Flutter dalam bentuk ZIP melalui GitHub Actions dan menangani berbagai struktur project, termasuk:

- ZIP berisi `pubspec.yaml` langsung
- ZIP memiliki folder root tambahan
- Folder `lib/` berada di dalam subfolder
- Entry point tidak selalu bernama `lib/main.dart`
- Project belum memiliki folder `android/`
- Project sudah memiliki folder `android/`
- Project memiliki Gradle Wrapper sendiri
- Project tidak memiliki Gradle Wrapper
- Build APK release
- Upload APK sebagai GitHub Actions Artifact
- Membuat APK hasil build di folder final

---

## Fitur

### Flutter

- Flutter Stable
- Flutter 3.47.0
- `flutter pub get`
- `flutter analyze`
- `flutter build apk --release`

### Android

Workflow dapat menangani project Flutter yang:

```text
project/
├── pubspec.yaml
├── lib/
│   └── main.dart
└── android/
```

maupun:

```text
project/
└── MyFlutterProject/
    ├── pubspec.yaml
    ├── lib/
    └── android/
```

Jika folder `android/` tidak tersedia, workflow dapat membuat platform Android menggunakan:

```bash
flutter create --platforms=android .
```

---

# Struktur Repository Build Bot

Contoh repository:

```text
build-apk/
├── .github/
│   └── workflows/
│       └── build.yml
├── README.md
└── package.json
```

Workflow utama:

```text
.github/workflows/build.yml
```

---

# Workflow

Nama workflow:

```yaml
name: Build Flutter APK
```

Workflow dijalankan menggunakan:

```yaml
on:
  workflow_dispatch:
```

Workflow dapat menerima input:

```text
zip_url
tag
build_type
```

Contoh:

```yaml
inputs:
  zip_url:
    description: "URL ZIP Flutter Project"
    required: true
    type: string

  tag:
    description: "Build Tag"
    required: true
    type: string

  build_type:
    description: "Build Type"
    required: false
    default: "release"
    type: string
```

---

# Cara Kerja

Alur build:

```text
Bot
 │
 │ trigger workflow
 ▼
GitHub Actions
 │
 ├── Checkout
 │
 ├── Setup Java
 │
 ├── Setup Flutter
 │
 ├── Download ZIP
 │
 ├── Extract ZIP
 │
 ├── Detect Flutter Project
 │
 ├── Detect pubspec.yaml
 │
 ├── Detect lib/
 │
 ├── Detect Entry Point
 │
 ├── Prepare Flutter
 │
 ├── Prepare Android
 │
 ├── Prepare Gradle
 │
 ├── Flutter Analyze
 │
 ├── Build APK
 │
 ├── Verify APK
 │
 ├── Prepare Release APK
 │
 └── Upload Artifact
```

---

# Struktur ZIP yang Didukung

## Struktur 1

Project langsung berada di root ZIP:

```text
project.zip
├── pubspec.yaml
├── lib/
│   └── main.dart
└── android/
```

---

## Struktur 2

Project berada di dalam folder:

```text
project.zip
└── Moodnote-main/
    ├── pubspec.yaml
    ├── lib/
    │   └── main.dart
    └── android/
```

Workflow akan mencari `pubspec.yaml` secara otomatis.

---

# Entry Point

Workflow tidak mengharuskan entry point bernama:

```text
lib/main.dart
```

Workflow dapat mencari file Dart di dalam folder `lib/`.

Contoh:

```text
lib/
├── app.dart
├── home.dart
└── main.dart
```

Jika tersedia:

```text
lib/main.dart
```

maka file tersebut akan digunakan.

Jika tidak tersedia, workflow dapat mendeteksi file Dart yang tersedia dan menggunakannya sebagai entry point sesuai konfigurasi build.

---

# Android Folder

Project Flutter tidak selalu memiliki folder:

```text
android/
```

Contoh project:

```text
pubspec.yaml
lib/
├── app.dart
└── main.dart
```

Project seperti ini belum memiliki platform Android.

Workflow akan menjalankan:

```bash
flutter create --platforms=android .
```

sehingga menjadi:

```text
pubspec.yaml
lib/
android/
├── app/
├── gradle/
├── gradlew
├── gradlew.bat
└── settings.gradle
```

---

# Jika Android Sudah Ada

Jika project sudah memiliki:

```text
android/
```

workflow tidak akan membuat ulang Android project.

Contoh:

```text
pubspec.yaml
lib/
android/
├── app/
├── gradle/
├── gradlew
└── settings.gradle
```

Workflow akan menggunakan konfigurasi Android yang sudah ada.

---

# Java

Build menggunakan:

```text
Java 17
```

Distribusi:

```text
Temurin
```

Konfigurasi:

```yaml
- name: Setup Java
  uses: actions/setup-java@v5
  with:
    distribution: temurin
    java-version: '17'
    cache: gradle
```

---

# Flutter

Flutter menggunakan channel:

```text
stable
```

Versi:

```text
3.47.0
```

Action:

```yaml
uses: subosito/flutter-action@v2
```

Konfigurasi:

```yaml
- name: Setup Flutter
  uses: subosito/flutter-action@v2
  with:
    flutter-version: '3.47.0'
    channel: stable
    cache: true
```

---

# Gradle

Project dapat menggunakan Gradle Wrapper bawaan.

Jika tersedia:

```text
android/gradlew
android/gradle/wrapper/gradle-wrapper.properties
```

workflow akan menggunakannya.

Jika tidak tersedia, workflow dapat menyiapkan Gradle yang diperlukan.

Gradle yang digunakan oleh workflow:

```text
8.14
```

---

# GitHub Actions

Action utama:

```text
actions/checkout
actions/setup-java
subosito/flutter-action
gradle/actions/setup-gradle
actions/upload-artifact
```

Contoh:

```yaml
uses: actions/checkout@v4
```

```yaml
uses: actions/setup-java@v5
```

```yaml
uses: subosito/flutter-action@v2
```

```yaml
uses: gradle/actions/setup-gradle@v4
```

```yaml
uses: actions/upload-artifact@v4
```

---

# GitHub Token

Bot membutuhkan token GitHub.

Secret:

```text
GH_PAT
```

Token digunakan untuk mengambil ZIP project jika ZIP berada pada repository atau endpoint GitHub yang membutuhkan autentikasi.

Contoh:

```yaml
env:
  GH_TOKEN: ${{ secrets.GH_PAT }}
```

---

# Download Project

Workflow menerima:

```text
zip_url
```

Contoh:

```text
https://api.github.com/repos/OWNER/REPO/zipball/main
```

Download dilakukan menggunakan:

```bash
curl
```

Contoh:

```bash
curl -L \
  --fail-with-body \
  -H "Authorization: Bearer $GH_TOKEN" \
  -H "Accept: application/octet-stream" \
  "$ZIP_URL" \
  -o project/project.zip
```

Setelah download:

```bash
file project/project.zip
```

dan:

```bash
unzip -t project/project.zip
```

digunakan untuk memvalidasi ZIP.

---

# Detect Flutter Project

Workflow mencari:

```text
pubspec.yaml
```

Contoh:

```bash
find project/source \
  -name pubspec.yaml \
  -type f
```

Jika ditemukan:

```text
project/source/Moodnote-main/pubspec.yaml
```

maka:

```text
PROJECT_DIR=project/source/Moodnote-main
```

Jika ditemukan:

```text
project/source/pubspec.yaml
```

maka:

```text
PROJECT_DIR=project/source
```

---

# Prepare Flutter

Setelah project ditemukan:

```bash
flutter --version
flutter pub get
```

Flutter dependencies akan di-download berdasarkan:

```text
pubspec.yaml
```

---

# Flutter Analyze

Workflow dapat menjalankan:

```bash
flutter analyze
```

untuk mendeteksi error pada source code.

Contoh error:

```text
lib/main.dart:10:5
Error
```

Jika project mempunyai error Dart yang menyebabkan build gagal, hasil analyze dapat digunakan untuk mengetahui masalahnya.

---

# Build APK

Build utama:

```bash
flutter build apk --release
```

Hasil default Flutter biasanya berada di:

```text
build/app/outputs/flutter-apk/
```

Contoh:

```text
app-release.apk
```

---

# Final APK

Workflow menyiapkan folder:

```text
build/final/
```

Contoh:

```text
build/final/
└── app-release.apk
```

atau APK yang sudah diberi nama berdasarkan tag:

```text
build/final/
└── build-123456.apk
```

---

# Upload Artifact

APK di-upload menggunakan:

```yaml
- name: Upload APK Artifact
  uses: actions/upload-artifact@v4
  with:
    name: flutter-apk-${{ inputs.tag }}
    path: ${{ steps.detect.outputs.project_dir }}/build/final/*.apk
    if-no-files-found: error
    retention-days: 7
```

Artifact dapat ditemukan di halaman:

```text
GitHub
→ Actions
→ Workflow Run
→ Artifacts
```

---

# Artifact Bukan Release

Penting:

```text
actions/upload-artifact@v4
```

menghasilkan:

```text
GitHub Actions Artifact
```

bukan:

```text
GitHub Release
```

Jadi:

```yaml
uses: actions/upload-artifact@v4
```

tidak otomatis membuat:

```text
Releases
```

atau:

```text
release.apk
```

di halaman GitHub Releases.

Artifact hanya tersedia pada workflow run.

---

# GitHub Release

Jika ingin APK masuk ke:

```text
GitHub
→ Releases
```

maka workflow harus mempunyai step tambahan untuk membuat GitHub Release.

Contohnya dapat menggunakan:

```text
softprops/action-gh-release
```

atau GitHub CLI:

```bash
gh release create
```

Release membutuhkan:

```text
tag
```

Contoh:

```text
build-123456
```

dan file:

```text
build/final/app-release.apk
```

---

# Integrasi Dengan Bot

Bot dapat memanggil workflow menggunakan GitHub REST API.

Endpoint:

```text
POST /repos/{owner}/{repo}/actions/workflows/build.yml/dispatches
```

Bot mengirim:

```json
{
  "ref": "main",
  "inputs": {
    "zip_url": "ZIP_URL",
    "tag": "build-123456",
    "build_type": "release"
  }
}
```

---

# Trigger Workflow

Contoh fungsi bot:

```javascript
async function triggerWorkflow(assetUrl, tagName, buildType = "release") {
  const beforeTrigger = new Date().toISOString();

  const res = await githubRequest(
    "POST",
    `/repos/${GITHUB_OWNER}/${GITHUB_REPO}/actions/workflows/build.yml/dispatches`,
    {
      ref: "main",
      inputs: {
        zip_url: assetUrl,
        tag: tagName,
        build_type: buildType
      },
    }
  );

  if (res.status !== 204) {
    throw new Error(`Gagal trigger workflow: ${JSON.stringify(res.body)}`);
  }

  return await waitForNewRunId(beforeTrigger, 30000);
}
```

---

# Workflow File

Nama file yang digunakan bot:

```text
build.yml
```

Lokasi:

```text
.github/workflows/build.yml
```

Bukan:

```text
build.yaml
```

kecuali fungsi bot juga diubah untuk memanggil:

```text
build.yaml
```

Karena bot saat ini menggunakan:

```javascript
/actions/workflows/build.yml/dispatches
```

maka file workflow harus bernama:

```text
build.yml
```

---

# Release Tag

Bot menghasilkan tag seperti:

```text
build-2027479396-1786790299371
```

Tag tersebut dapat digunakan sebagai identifier build.

Contoh:

```text
Build Type: release
Tag: build-2027479396-1786790299371
```

---

# Artifact Download

Bot dapat mengambil daftar artifact menggunakan:

```text
GET /repos/{owner}/{repo}/actions/runs/{run_id}/artifacts
```

Fungsi:

```javascript
async function getArtifacts(runId) {
  const res = await githubRequest(
    "GET",
    `/repos/${GITHUB_OWNER}/${GITHUB_REPO}/actions/runs/${runId}/artifacts`
  );

  return res.body.artifacts || [];
}
```

---

# Download Artifact

Bot dapat mendownload artifact menggunakan:

```text
GET /repos/{owner}/{repo}/actions/artifacts/{artifact_id}/zip
```

Fungsi:

```javascript
downloadArtifactZip(artifactId, destPath)
```

Artifact GitHub berbentuk ZIP.

Contoh:

```text
flutter-apk-build-123456.zip
```

Setelah diekstrak, APK tersedia di dalamnya.

---

# Monitoring Build

Bot dapat mengecek status workflow menggunakan:

```text
GET /repos/{owner}/{repo}/actions/runs/{run_id}
```

Status dapat berupa:

```text
queued
```

```text
in_progress
```

```text
completed
```

Conclusion:

```text
success
```

```text
failure
```

```text
cancelled
```

---

# Failed Step

Bot dapat mengambil job:

```text
GET /repos/{owner}/{repo}/actions/runs/{run_id}/jobs
```

Kemudian mencari:

```text
conclusion === "failure"
```

Bot dapat menampilkan step yang gagal, misalnya:

```text
Setup Java
```

```text
Setup Flutter
```

```text
Download Project
```

```text
Detect Flutter Project
```

```text
Prepare Flutter
```

```text
Prepare Android Platform
```

```text
Prepare Gradle Wrapper
```

```text
Flutter Analyze
```

```text
Build APK
```

```text
Verify APK
```

---

# Contoh Error

Jika muncul:

```text
Target kernel_snapshot_program failed
```

itu biasanya bukan berarti Gradle Wrapper rusak.

Error tersebut dapat berasal dari:

```text
Dart code
Flutter code
dependency
plugin
build configuration
```

Periksa log sebelum:

```text
Target kernel_snapshot_program failed
```

karena biasanya terdapat error utama yang menjelaskan penyebabnya.

---

# Contoh Error Android

Jika muncul:

```text
Folder android tidak ditemukan.
```

project belum mempunyai platform Android.

Workflow dapat membuatnya menggunakan:

```bash
flutter create --platforms=android .
```

---

# Contoh Error Gradle

Jika muncul:

```text
Gradle threw an error while downloading artifacts from the network.
```

kemungkinan masalah berasal dari:

```text
network
Gradle distribution
Gradle cache
GitHub runner
```

bukan dari source code Flutter secara langsung.

---

# Contoh Error Setup Java

Jika muncul:

```text
No file in /home/runner/work/build-apk/build-apk
matched to [**/*.gradle, **/gradle-wrapper.properties,...]
```

itu biasanya berasal dari konfigurasi:

```yaml
cache: gradle
```

pada:

```yaml
actions/setup-java
```

karena action mencoba mencari file Gradle pada repository utama sebelum project Flutter hasil ZIP tersedia.

Solusi yang lebih aman untuk workflow yang project-nya didownload setelah checkout adalah tidak menggunakan:

```yaml
cache: gradle
```

pada `actions/setup-java`.

Gunakan:

```yaml
- name: Setup Java
  uses: actions/setup-java@v5
  with:
    distribution: temurin
    java-version: '17'
```

Kemudian cache Gradle dapat ditangani setelah project Flutter sudah tersedia.

---

# Struktur Project Flutter Minimal

Project Flutter minimal:

```text
my-project/
├── pubspec.yaml
└── lib/
    └── main.dart
```

Contoh:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Hello Flutter'),
        ),
      ),
    ),
  );
}
```

Project tersebut belum harus mempunyai:

```text
android/
ios/
web/
windows/
linux/
macos/
```

Platform dapat dibuat menggunakan Flutter CLI.

---

# pubspec.yaml

Contoh:

```yaml
name: example_app

description: Example Flutter Application

publish_to: "none"

version: 1.0.0+1

environment:
  sdk: ">=3.0.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter

flutter:
  uses-material-design: true
```

---

# Repository Build Bot

Repository utama:

```text
build-apk
```

Workflow:

```text
.github/workflows/build.yml
```

Bot menggunakan:

```text
GitHub REST API
```

untuk:

```text
Upload project
Create release
Upload ZIP
Trigger workflow
Check workflow status
Get artifacts
Download artifacts
Get failed job logs
```

---

# Alur Lengkap Bot

```text
User
 │
 │ Upload Flutter ZIP
 ▼
Bot
 │
 ├── Upload ZIP
 │
 ├── Create Release
 │
 ├── Upload ZIP Asset
 │
 └── Trigger build.yml
 │
 ▼
GitHub Actions
 │
 ├── Checkout
 ├── Setup Java
 ├── Setup Flutter
 ├── Download ZIP
 ├── Extract ZIP
 ├── Detect Project
 ├── Detect pubspec.yaml
 ├── Prepare Flutter
 ├── Detect Android
 ├── Generate Android jika diperlukan
 ├── Prepare Gradle
 ├── Analyze
 ├── Build APK
 ├── Verify APK
 └── Upload Artifact
 │
 ▼
GitHub Artifact
 │
 ▼
Bot
 │
 ├── Get Artifact
 ├── Download Artifact
 └── Kirim APK ke User
```

---

# Nama File

Gunakan:

```text
build.yml
```

Lokasi:

```text
.github/workflows/build.yml
```

Pastikan fungsi bot juga menggunakan:

```javascript
/actions/workflows/build.yml/dispatches
```

---

# Catatan

Workflow tidak dapat memperbaiki semua error source code Flutter.

Jika project mempunyai error seperti:

```text
Undefined name
```

```text
The method 'xxx' isn't defined
```

```text
Target kernel_snapshot_program failed
```

```text
Gradle task assembleRelease failed
```

maka log build harus diperiksa untuk menemukan error pertama yang muncul.

`Target kernel_snapshot_program failed` biasanya merupakan efek akhir dari error sebelumnya.

---

# Status

```text
Flutter Build System
Status: Active
Platform: GitHub Actions
Output: APK
Build Mode: Release
Java: 17
Flutter: 3.47.0
Gradle: 8.14
```

---

# Lisensi

Project ini digunakan untuk kebutuhan build dan automation Flutter APK.

Gunakan GitHub Actions, GitHub API, Flutter, Gradle, dan dependency pihak ketiga sesuai dengan lisensi masing-masing.
