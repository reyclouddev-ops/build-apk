
Flutter Build APK

GitHub Actions workflow untuk membangun project Flutter menjadi APK secara otomatis menggunakan GitHub Actions.

Project Flutter dikirim dalam bentuk ZIP melalui "workflow_dispatch", kemudian workflow akan:

- Download project
- Extract ZIP
- Mendeteksi "pubspec.yaml"
- Mendeteksi lokasi project Flutter
- Mendeteksi entry point Dart
- Menyiapkan Android jika belum tersedia
- Menjalankan Flutter
- Menjalankan "flutter analyze"
- Build APK
- Memeriksa hasil APK
- Upload APK sebagai Artifact
- Membuat GitHub Release
- Upload APK ke Release

Repository

"reyclouddev-ops/build-apk" (https://reference-url-citation.invalid/1)

Teknologi

- Flutter Stable 3.47.0
- Dart 3.13.0
- Java 17
- Gradle
- GitHub Actions
- GitHub CLI/API
- "subosito/flutter-action@v2"

Struktur Repository

build-apk/
├── .github/
│   └── workflows/
│       └── build.yml
├── README.md
└── package.json

Workflow utama berada di:

.github/workflows/build.yml

Cara Kerja

Flutter ZIP
    │
    ▼
GitHub Actions
    │
    ├── Checkout
    ├── Setup Java
    ├── Setup Flutter
    ├── Download Project
    ├── Extract ZIP
    ├── Detect Flutter Project
    ├── Detect Entry Point
    ├── Prepare Flutter
    ├── Prepare Android
    ├── Verify Gradle
    ├── Flutter Analyze
    ├── Build APK
    ├── Verify APK
    ├── Upload Build Logs
    ├── Upload APK Artifact
    ├── Create GitHub Release
    ├── Upload APK Release
    └── Finalize Release

Workflow Input

Workflow menggunakan:

on:
  workflow_dispatch:

Input yang tersedia:

"zip_url"

URL ZIP project Flutter.

Contoh:

https://example.com/project.zip

ZIP juga dapat berasal dari GitHub Release Asset.

"tag"

Tag untuk hasil build.

Contoh:

build-123456

Tag digunakan sebagai identifier build dan GitHub Release.

"build_type"

Jenis APK yang dibuat.

Pilihan:

debug
profile
release

Default:

release

Contoh Input

zip_url:
https://example.com/flutter-project.zip

tag:
build-123456

build_type:
release

Struktur ZIP

Workflow dapat menemukan project Flutter walaupun "pubspec.yaml" tidak berada langsung di root ZIP.

Root langsung

project.zip
├── pubspec.yaml
├── lib/
│   └── main.dart
├── android/
└── assets/

Menggunakan folder project

project.zip
└── MyProject/
    ├── pubspec.yaml
    ├── lib/
    │   └── main.dart
    ├── android/
    └── assets/

Workflow mencari:

pubspec.yaml

kemudian menggunakan folder tempat file tersebut berada sebagai project Flutter.

Entry Point

Workflow terlebih dahulu mencari:

lib/main.dart

Jika tidak tersedia, workflow mencari file Dart lain di dalam "lib/" yang memiliki:

void main(

Contoh:

lib/
├── app.dart
├── dashboard.dart
└── start.dart

Jika "start.dart" memiliki "void main()", file tersebut dapat digunakan sebagai entry point.

Android

Jika project sudah mempunyai folder:

android/

workflow akan menggunakan Android project tersebut.

Contoh:

project/
├── pubspec.yaml
├── lib/
└── android/
    ├── app/
    ├── gradle/
    ├── gradlew
    └── settings.gradle

Jika folder Android belum tersedia, workflow akan menjalankan:

flutter create --platforms=android --project-name flutter_build_app .

Setelah itu workflow memastikan file Android yang diperlukan tersedia.

Android Project Tidak Diubah Untuk Semua User

Workflow bekerja di environment GitHub Actions yang dibuat khusus untuk setiap workflow run.

Project yang di-download berada di workspace runner, misalnya:

project/source/

Perubahan seperti:

flutter clean
flutter pub get
flutter create

hanya berlaku pada workspace build tersebut.

File project asli user tidak diubah oleh workflow.

Java

Workflow menggunakan:

Java 17

Distribusi:

Temurin

Konfigurasi:

- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: "17"
    cache: gradle

Flutter

Versi Flutter:

3.47.0

Channel:

stable

Action:

uses: subosito/flutter-action@v2

Dependencies

Workflow menjalankan:

flutter pub get

sebelum proses build.

Build juga menjalankan "flutter pub get" kembali setelah:

flutter clean

Analyze

Workflow menjalankan:

flutter analyze

Hasil analyze disimpan sebagai:

analyze.log

Analyze tidak langsung menghentikan workflow karena step tersebut menggunakan:

continue-on-error: true

Error build Dart yang sebenarnya tetap akan menyebabkan proses "flutter build apk" gagal.

Build APK

Untuk debug:

flutter build apk \
  --debug \
  --target="$ENTRY_POINT"

Untuk profile:

flutter build apk \
  --profile \
  --target="$ENTRY_POINT"

Untuk release:

flutter build apk \
  --release \
  --target="$ENTRY_POINT"

Workflow juga menggunakan:

--android-skip-build-dependency-validation

Output APK

Setelah build berhasil, workflow mencari APK pada:

build/app/outputs/

APK kemudian disalin menjadi:

build/final/release.apk

GitHub Actions Artifact

APK di-upload menggunakan:

actions/upload-artifact@v4

Nama artifact:

flutter-apk-{tag}

Contoh:

flutter-apk-build-123456

Artifact disimpan selama:

7 hari

Build Logs

Workflow juga menyimpan log build.

File yang dapat di-upload:

build_output.log
build_error.log
analyze.log

Nama artifact:

build-logs-{tag}

Log ini berguna untuk mengetahui penyebab build gagal.

GitHub Release

Jika build berhasil, workflow membuat GitHub Release berdasarkan:

tag

Contoh:

build-123456

Nama release:

Build build-123456

Release dibuat sebagai:

Pre-release

APK Pada Release

APK yang di-upload ke Release menggunakan nama:

app-{tag}.apk

Contoh:

app-build-123456.apk

Sedangkan file sementara pada workspace:

build/final/release.apk

GitHub Token

Workflow membutuhkan secret:

GH_PAT

Secret digunakan untuk:

- Download GitHub Release Asset
- Mengakses GitHub API
- Membuat Release
- Upload APK ke Release
- Mengelola Release berdasarkan tag

Jangan memasukkan token GitHub langsung ke source code.

Permission

Workflow menggunakan:

permissions:
  contents: write
  actions: write

"contents: write" digunakan untuk kebutuhan GitHub Release.

"actions: write" digunakan untuk kebutuhan workflow/API terkait Actions.

Integrasi Bot

Workflow dapat dipanggil oleh bot menggunakan GitHub Actions API.

Endpoint:

POST /repos/{owner}/{repo}/actions/workflows/build.yml/dispatches

Input:

{
  "ref": "main",
  "inputs": {
    "zip_url": "ZIP_URL",
    "tag": "build-123456",
    "build_type": "release"
  }
}

Bot dapat menggunakan:

build.yml

sebagai workflow yang dipanggil.

Monitoring Build

Setelah workflow dijalankan, bot dapat memonitor:

GET /repos/{owner}/{repo}/actions/runs/{run_id}

Status:

queued
in_progress
completed

Conclusion:

success
failure
cancelled

Artifact

Artifact dapat diambil melalui GitHub Actions API:

GET /repos/{owner}/{repo}/actions/runs/{run_id}/artifacts

Kemudian artifact dapat didownload menggunakan:

GET /repos/{owner}/{repo}/actions/artifacts/{artifact_id}/zip

Artifact berisi APK:

release.apk

Error Build

Jika muncul:

Target kernel_snapshot_program failed

jangan langsung menganggap Gradle yang bermasalah.

Periksa error sebelum baris tersebut.

Biasanya penyebabnya dapat berasal dari:

Dart code
Flutter code
dependency
plugin
Android configuration
Gradle configuration

Contoh:

lib/page_login.dart:153:13
Error: No named parameter with the name 'email'.

Error seperti ini berasal dari source code Dart, bukan dari proses download APK.

Android Configuration Error

Jika muncul:

Android Gradle configuration tidak ditemukan

periksa project ZIP dan pastikan terdapat:

android/
├── app/
├── gradle/
├── gradlew
└── settings.gradle

atau project menggunakan:

settings.gradle.kts

dan:

android/app/build.gradle.kts

Gradle Wrapper

Workflow memeriksa:

android/gradlew

Kemudian menjalankan:

chmod +x android/gradlew

Workflow juga melakukan verifikasi:

./gradlew --version

Troubleshooting

APK gagal dibuat

Periksa:

Build APK

Kemudian lihat:

build_output.log

"pubspec.yaml" tidak ditemukan

Pastikan ZIP benar-benar berisi project Flutter.

Minimal:

pubspec.yaml
lib/

Android tidak ditemukan

Workflow akan mencoba membuat Android platform secara otomatis jika folder "android/" belum tersedia.

Entry point tidak ditemukan

Pastikan terdapat:

void main() {

di salah satu file Dart dalam:

lib/

Dependency gagal

Coba build project secara lokal:

flutter pub get
flutter analyze
flutter build apk --release

Rekomendasi Struktur Project Flutter

project/
├── android/
├── assets/
│   ├── images/
│   ├── videos/
│   └── icon/
├── lib/
│   ├── main.dart
│   ├── api_service.dart
│   └── pages/
├── pubspec.yaml
└── pubspec.lock

Launcher Icon

Jika project menggunakan "flutter_launcher_icons", konfigurasi dapat ditempatkan di "pubspec.yaml".

Contoh:

dev_dependencies:
  flutter_launcher_icons: ^0.14.4

flutter_launcher_icons:
  android: true
  ios: false
  image_path: assets/icon/icon.png
  min_sdk_android: 21

Pastikan path asset benar-benar sama dengan struktur project.

Contoh:

assets/
└── icon/
    └── icon.png

Jika project menggunakan nama folder berbeda seperti:

assest/

maka path pada "pubspec.yaml" juga harus menggunakan nama tersebut secara konsisten.

Catatan Penting

Workflow build bersifat per-run.

Project yang dikirim melalui "zip_url" digunakan sebagai source untuk proses build pada runner GitHub Actions.

Perubahan yang dilakukan selama workflow berjalan tidak otomatis ditulis kembali ke repository project asal.

APK hasil build tersedia melalui:

GitHub Actions Artifact

dan:

GitHub Release

Lisensi

Project ini digunakan untuk kebutuhan build APK Flutter melalui GitHub Actions.

Silakan sesuaikan lisensi dan aturan penggunaan sesuai kebutuhan repository.
