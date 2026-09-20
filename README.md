Лабораторная работа №6

репозиторий: https://github.com/Shohiii/lab06

условие лабораторной работы: https://github.com/tp-labs/lab06

Homework
После настройки непрерывной интеграции необходимо организовать создание пакетов для изменений, помеченных тегами.
Пакет должен содержать приложение `solver` из предыдущей лабораторной работы.
Каждый новый релиз должен содержать:
архивы с исходным кодом: `.tar.gz`, `.zip`;
пакеты с бинарным файлом `solver`: `.deb`, `.rpm`, `.msi`, `.dmg`.
Если commit помечен тегом, CI должен собрать пакеты и разместить их в GitHub Releases.
---
1. Подготовка проекта `solver`
В качестве основы использован проект из лабораторной работы №3.
Проверка структуры исходного проекта:
```bash
find /tmp/lab03-source -maxdepth 2 -type f | sort
```
<details>
<summary>Вывод</summary>
```text
/tmp/lab03-source/.git/HEAD
/tmp/lab03-source/.git/config
/tmp/lab03-source/.git/description
/tmp/lab03-source/.git/index
/tmp/lab03-source/.git/packed-refs
/tmp/lab03-source/.gitignore
/tmp/lab03-source/CMakeLists.txt
/tmp/lab03-source/LICENSE
/tmp/lab03-source/README.md
/tmp/lab03-source/formatter_ex_lib/CMakeLists.txt
/tmp/lab03-source/formatter_ex_lib/formatter_ex.cpp
/tmp/lab03-source/formatter_ex_lib/formatter_ex.h
/tmp/lab03-source/formatter_lib/CMakeLists.txt
/tmp/lab03-source/formatter_lib/formatter.cpp
/tmp/lab03-source/formatter_lib/formatter.h
/tmp/lab03-source/hello_world_application/CMakeLists.txt
/tmp/lab03-source/hello_world_application/hello_world.cpp
/tmp/lab03-source/preview.png
/tmp/lab03-source/solver_application/CMakeLists.txt
/tmp/lab03-source/solver_application/equation.cpp
/tmp/lab03-source/solver_lib/CMakeLists.txt
/tmp/lab03-source/solver_lib/solver.cpp
/tmp/lab03-source/solver_lib/solver.h
```
</details>
В проект были перенесены библиотеки `formatter`, `formatter_ex`, `solver_lib` и приложения `hello_world` и `solver`.
Для приложения `solver` добавлена установка бинарного файла:
```cmake
install(TARGETS solver
    RUNTIME DESTINATION bin
)
```
Проверка конфигурации:
```bash
cat solver_application/CMakeLists.txt
```
Вывод:
```text
cmake_minimum_required(VERSION 3.10)

project(solver)

add_executable(solver
    equation.cpp
)

target_link_libraries(solver
    formatter_ex
    solver_lib
)

install(TARGETS solver
    RUNTIME DESTINATION bin
)
```
---
2. Сборка и проверка приложения `solver`
Конфигурация проекта:
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
```
Вывод:
```text
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/nikita/lab06/build
```
Сборка:
```bash
cmake --build build
```
Вывод:
```text
[ 20%] Built target formatter
[ 40%] Built target formatter_ex
[ 60%] Built target solver_lib
[ 80%] Built target hello_world
[100%] Built target solver
```
Проверка наличия исполняемого файла:
```bash
find build -type f \( -name solver -o -name solver.exe \) -print
```
Вывод:
```text
build/solver_application/solver
```
Проверка работы приложения на уравнении `x² - 3x + 2 = 0`:
```bash
printf "1 -3 2\n" | ./build/solver_application/solver
```
Вывод:
```text
-------------------------
x1 = 1.000000
-------------------------
-------------------------
x2 = 2.000000
-------------------------
```
---
3. Проверка установки `solver`
Команда:
```bash
cmake --install build --prefix build/install
```
Вывод:
```text
-- Install configuration: "Release"
-- Up-to-date: /home/nikita/lab06/build/install/bin/solver
```
Проверка установленного файла:
```bash
find build/install -maxdepth 3 -type f -print
```
Вывод:
```text
build/install/bin/solver
```
Проверка установленного приложения:
```bash
printf "1 -3 2\n" | ./build/install/bin/solver
```
Вывод:
```text
-------------------------
x1 = 1.000000
-------------------------
-------------------------
x2 = 2.000000
-------------------------
```
---
4. Настройка CPack
Для пакетирования создан файл `CPackConfig.cmake`.
Команда проверки:
```bash
cat CPackConfig.cmake
```
<details>
<summary>Вывод</summary>
```cmake
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VENDOR "Shohiii")
set(CPACK_PACKAGE_CONTACT "Shohiii")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Quadratic equation solver")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION")

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt")

set(CPACK_PACKAGE_FILE_NAME
    "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}"
)

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Shohiii")
set(CPACK_DEBIAN_PACKAGE_SECTION "utils")

set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Applications/Engineering")
set(CPACK_RPM_CHANGELOG_FILE "${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md")

set(CPACK_WIX_VERSION "4")
set(CPACK_WIX_UPGRADE_GUID "C6DA779D-12CC-43FA-9946-8CFA5CA1A509")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_PACKAGE_FILE_NAME
    "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-source"
)

set(CPACK_SOURCE_IGNORE_FILES
    "/build/"
    "/_build/"
    "/_CPack_Packages/"
    "/artifacts/"
    "/\\.git/"
    "lab06_terminal.txt"
)

include(CPack)
```
</details>
---
5. Создание DEB-пакета
Команда:
```bash
cpack -G DEB
```
Вывод:
```text
CPack: Create package using DEB
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
-- CPACK_DEBIAN_PACKAGE_DEPENDS not set, the package will have no dependencies.
CPack: - package: /home/nikita/lab06/build/solver-1.0.0-Linux-x86_64.deb generated.
```
Проверка созданного пакета:
```bash
ls -lh *.deb
```
Вывод:
```text
-rw-r--r-- 1 nikita nikita 6.3K Sep 20 13:23 solver-1.0.0-Linux-x86_64.deb
```
Проверка содержимого пакета:
```bash
dpkg-deb -c ./*.deb
```
Вывод:
```text
drwxr-xr-x root/root         0 2026-09-20 13:23 ./usr/
drwxr-xr-x root/root         0 2026-09-20 13:23 ./usr/bin/
-rwxr-xr-x root/root     18832 2026-09-20 13:18 ./usr/bin/solver
```
Таким образом, DEB-пакет действительно содержит бинарный файл `solver`.
---
6. Создание RPM-пакета
Для генерации RPM был установлен пакет `rpm`.
Проверка:
```bash
rpmbuild --version
```
Вывод:
```text
RPM version 6.0.1
```
Создание пакета:
```bash
cpack -G RPM
```
Вывод:
```text
CPack: Create package using RPM
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
CPackRPM: Will use GENERATED spec file: /home/nikita/lab06/build/_CPack_Packages/Linux/RPM/SPECS/solver.spec
CPack: - package: /home/nikita/lab06/build/solver-1.0.0-Linux-x86_64.rpm generated.
```
Проверка созданного файла:
```bash
ls -lh *.rpm
```
Вывод:
```text
-rw-r--r-- 1 nikita nikita 13K Sep 20 13:32 solver-1.0.0-Linux-x86_64.rpm
```
Проверка содержимого:
```bash
rpm -qlp ./*.rpm
```
Вывод:
```text
error: can't create transaction lock on /var/lib/rpm/.rpm.lock (No such file or directory)
/usr/bin/solver
```
Несмотря на предупреждение RPM в WSL, список содержимого был получен, и пакет содержит `/usr/bin/solver`.
---
7. Создание архивов с исходным кодом
Создание `.tar.gz`:
```bash
cpack --config CPackSourceConfig.cmake -G TGZ
```
Вывод:
```text
CPack: Create package using TGZ
CPack: Install projects
CPack: - Install directory: /home/nikita/lab06
CPack: Create package
CPack: - package: /home/nikita/lab06/build/solver-1.0.0-source.tar.gz generated.
```
Проверка:
```bash
ls -lh *.tar.gz
```
Вывод:
```text
-rw-r--r-- 1 nikita nikita 6.5K Sep 20 13:32 solver-1.0.0-source.tar.gz
```
Создание `.zip`:
```bash
cpack --config CPackSourceConfig.cmake -G ZIP
```
Вывод:
```text
CPack: Create package using ZIP
CPack: Install projects
CPack: - Install directory: /home/nikita/lab06
CPack: Create package
CPack: - package: /home/nikita/lab06/build/solver-1.0.0-source.zip generated.
```
Проверка:
```bash
ls -lh *.zip
```
Вывод:
```text
-rw-r--r-- 1 nikita nikita 16K Sep 20 13:33 solver-1.0.0-source.zip
```
Все созданные локально пакеты:
```bash
ls -lh *.deb *.rpm *.tar.gz *.zip
```
Вывод:
```text
-rw-r--r-- 1 nikita nikita 6.3K Sep 20 13:23 solver-1.0.0-Linux-x86_64.deb
-rw-r--r-- 1 nikita nikita  13K Sep 20 13:32 solver-1.0.0-Linux-x86_64.rpm
-rw-r--r-- 1 nikita nikita 6.5K Sep 20 13:32 solver-1.0.0-source.tar.gz
-rw-r--r-- 1 nikita nikita  16K Sep 20 13:33 solver-1.0.0-source.zip
```
---
8. Настройка CI для сборки пакетов
Для автоматической сборки используется один файл:
```text
.github/workflows/release.yml
```
Он запускается при отправке тега вида `v*` и содержит четыре задания:
`Linux packages` — сборка `.deb`, `.rpm`, `.tar.gz`, `.zip`;
`macOS DMG` — сборка `.dmg`;
`Windows MSI` — сборка `.msi`;
`Create GitHub Release` — загрузка всех пакетов в GitHub Release.
Проверка файла:
```bash
cat .github/workflows/release.yml
```
<details>
<summary>Вывод</summary>
```yaml
name: Build packages and release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  linux:
    name: Linux packages
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y cmake g++ rpm

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Release

      - name: Build
        run: cmake --build build

      - name: Build DEB
        working-directory: build
        run: cpack -G DEB

      - name: Build RPM
        working-directory: build
        run: cpack -G RPM

      - name: Build source TGZ
        working-directory: build
        run: cpack --config CPackSourceConfig.cmake -G TGZ

      - name: Build source ZIP
        working-directory: build
        run: cpack --config CPackSourceConfig.cmake -G ZIP

      - name: Show packages
        run: |
          find build -maxdepth 1 -type f \( \
            -name "*.deb" -o \
            -name "*.rpm" -o \
            -name "*.tar.gz" -o \
            -name "*.zip" \
          \) -print

      - name: Upload Linux artifacts
        uses: actions/upload-artifact@v4
        with:
          name: linux-packages
          path: |
            build/*.deb
            build/*.rpm
            build/*.tar.gz
            build/*.zip
          if-no-files-found: error

  macos:
    name: macOS DMG
    runs-on: macos-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Release

      - name: Build
        run: cmake --build build

      - name: Build DMG
        working-directory: build
        run: cpack -G DragNDrop

      - name: Show DMG
        run: find build -maxdepth 1 -type f -name "*.dmg" -print

      - name: Upload DMG
        uses: actions/upload-artifact@v4
        with:
          name: macos-package
          path: build/*.dmg
          if-no-files-found: error

  windows:
    name: Windows MSI
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install WiX
        shell: powershell
        run: |
          dotnet tool install --global wix --version 4.0.4
          wix extension add --global WixToolset.UI.wixext/4.0.4

      - name: Configure
        run: cmake -S . -B build

      - name: Build
        run: cmake --build build --config Release

      - name: Build MSI
        working-directory: build
        run: cpack -C Release -G WIX

      - name: Show MSI
        shell: powershell
        run: Get-ChildItem build -Filter *.msi

      - name: Upload MSI
        uses: actions/upload-artifact@v4
        with:
          name: windows-package
          path: build/*.msi
          if-no-files-found: error

  release:
    name: Create GitHub Release
    needs:
      - linux
      - macos
      - windows
    runs-on: ubuntu-latest

    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts
          merge-multiple: true

      - name: Show release files
        run: find artifacts -maxdepth 1 -type f -print

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            artifacts/*.deb
            artifacts/*.rpm
            artifacts/*.msi
            artifacts/*.dmg
            artifacts/*.tar.gz
            artifacts/*.zip
```
</details>
---
9. Создание тега и запуск CI
Создан тег релиза:
```bash
git tag v1.0.0
```
Команда не выводит текст при успешном создании тега.
Тег отправлен в удалённый репозиторий:
```bash
git push origin v1.0.0
```
Вывод:
```text
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/Shohiii/lab06.git
 * [new tag]         v1.0.0 -> v1.0.0
```
После отправки тега автоматически запустился workflow:
```bash
gh run list --workflow release.yml --limit 5
```
Вывод:
```text
STATUS  TITLE                   WORKFLOW                    BRANCH  EVENT  ID           ELAPSED  AGE
*       Remove accidental file  Build packages and release  v1.0.0  push   35513602342  16s      less than a minute ago
```
---
10. Результат выполнения GitHub Actions
Проверка запуска:
```bash
gh run view 35513602342
```
<details>
<summary>Вывод</summary>
```text
✓ v1.0.0 Build packages and release · 35513602342
Triggered via push about 6 minutes ago

JOBS
✓ macOS DMG in 37s (ID 106085682457)
✓ Windows MSI in 1m14s (ID 106085682579)
✓ Linux packages in 22s (ID 106085682651)
✓ Create GitHub Release in 6s (ID 106085844802)

ARTIFACTS
macos-package
linux-packages
windows-package

View this run on GitHub:
https://github.com/Shohiii/lab06/actions/runs/35513602342
```
</details>
Все четыре задания CI завершились успешно.
---
11. Проверка GitHub Release
Список релизов:
```bash
gh release list
```
Вывод:
```text
TITLE     TYPE    TAG NAME  PUBLISHED
v1.0.0    Latest  v1.0.0    about 6 minutes ago
v0.1.0.1          v0.1.0.1  about 2 months ago
v0.1.1.0          v0.1.1.0  about 2 months ago
```
Просмотр релиза:
```bash
gh release view v1.0.0
```
Вывод:
```text
v1.0.0
github-actions[bot] released this about 6 minutes ago

Assets
solver-1.0.0-Darwin-arm64.dmg   33.58 KiB
solver-1.0.0-Linux-x86_64.deb   6.20 KiB
solver-1.0.0-Linux-x86_64.rpm   12.76 KiB
solver-1.0.0-source.tar.gz      7.12 KiB
solver-1.0.0-source.zip         17.42 KiB
solver-1.0.0-Windows-AMD64.msi  856.00 KiB

View on GitHub: https://github.com/Shohiii/lab06/releases/tag/v1.0.0
```
Проверка имён всех прикреплённых файлов:
```bash
gh release view v1.0.0 --json assets --jq '.assets[].name'
```
Вывод:
```text
solver-1.0.0-Darwin-arm64.dmg
solver-1.0.0-Linux-x86_64.deb
solver-1.0.0-Linux-x86_64.rpm
solver-1.0.0-source.tar.gz
solver-1.0.0-source.zip
solver-1.0.0-Windows-AMD64.msi
```
Проверка тега в удалённом репозитории:
```bash
git ls-remote --tags origin | grep "refs/tags/v1.0.0"
```
Вывод:
```text
53a67334de1092b605aa5d56bb1d31541ef3afdf        refs/tags/v1.0.0
```
---
Итог
Для приложения `solver` настроено пакетирование через CPack.
При отправке тега `v1.0.0` GitHub Actions автоматически выполнил сборки на Linux, macOS и Windows и создал GitHub Release.
Релиз содержит все требуемые заданием файлы:
`solver-1.0.0-source.tar.gz`;
`solver-1.0.0-source.zip`;
`solver-1.0.0-Linux-x86_64.deb`;
`solver-1.0.0-Linux-x86_64.rpm`;
`solver-1.0.0-Windows-AMD64.msi`;
`solver-1.0.0-Darwin-arm64.dmg`.
