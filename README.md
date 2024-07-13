## Homework

### Задание
После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1).

**1. Создаем файл CMakeLists.txt в корне репозитория lab06, который мы сначала привязываем к гитхабу**

```bash
cmake_minimum_required(VERSION 3.4)
project(solver)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include_directories(solver_application)

include(CPackConfig.cmake)
```

* Подключение `include(CPackCongig.cmake)` в CMakeLists.txt позволяет использовать функционал CPack для создания пакетов и установочных файлов.


**2. Создаем файл CPackConig.cmake, где указываем необходимые параметры нашего проекта**

```bash
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT a_m_andreev2005.ru)
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for solver")
set(CPACK_PACKAGE_VENDOR "amandreev17")
set(CPACK_PACKAGE_PACK_NAME "solver-${PRINT_VERSION}")

set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "solves equations")

set(CPACK_GENERATOR "DEB;RPM")
set(CPACK_RPM_PACKAGE_SUMMARY "solves equations")

include(CPack)
```

**3. Создаем папку .github/workflows, в которой создаем два файла сборки: c-ccp.yml и tag.yml**

* Назначение 1-го файла:
- Этот файл создает рабочий процесс (workflow) для билдинга проекта в GitHub Actions при каждом push или pull_request в ветку main, он нужен для:
- Выполняет проверку кода репозитория.
- Настраивает сборку проекта с помощью CMake.


```bash
name: Build
on:
  push:
jobs:
  build-project:
    name: Build Project
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4.1.2

      - name: Configure and solver
        run: |
          cmake -B build
          cmake --build build
```

* Назначение 2-го файла:
- Файл создает рабочий процесс (workflow) для билдинга проекта в GitHub Actions при каждом push нового тэга версии (с тэгом в формате 'v*.*.*.*'). Он выполняет следующие действия:
- Запускает сборку проекта.
- Создает упакованные файлы.
- Создает и публикует релиз в GitHub с созданными артефактами (пакеты DEB, RPM, TAR.GZ, ZIP).
     

```bash
name: CMake

on:
 push:
   tags:
     - 'v*.*.*.*'

jobs: 

  build:

    runs-on: ubuntu-latest
    
    permissions:
      contents: write

    steps:
    - uses: actions/checkout@v3
    - name: Configure Solver
      run: cmake -B build
    - name: Build Solver
      run: cmake --build build
    - name: Build package
      run: cmake --build build --target package
    - name: Build source package
      run: cmake --build build --target package_source

    - name: Make a release
      uses: ncipollo/release-action@v1.14.0
      with:
        artifacts: "build/*.deb,build/*.rpm,build/*.tar.gz,build/*.zip"
        token: ${{ secrets.GITHUB_TOKEN }}
```

* Для создания тэга пишем в терминал:

```bash
$ git tag v1.0.0.0
```

* После пушим тэг:
```bash
$ git push origin --tags
```
