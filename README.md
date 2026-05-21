# Отчет по лабораторной работе №4

**Студент:** Петров Владислав Михайлович ИУ8-22

**Тема:** Настройка системы непрерывной интеграции Travis

---

## 1. Ход выполнения работы

### 2. Подготовка проекта

Был скопирован проект из лабораторной работы №3:

### Конфигурация Travis CI

```yaml
language: cpp
matrix:
  include:
    - os: linux
      compiler: gcc
    - os: linux
      compiler: clang
script:
  - cmake -H. -B_build
  - cmake --build _build
```

### 2.1. Настройка GitHub Actions (Linux)



```yaml
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build:
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        compiler: [gcc, clang]
    env:
      CC: ${{ matrix.compiler }}
      CXX: ${{ matrix.compiler == 'gcc' && 'g++' || 'clang++' }}
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y cmake build-essential
      - name: Configure
        run: cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=install
      - name: Build
        run: cmake --build _build
      - name: Install
        run: cmake --build _build --target install
```

### 2.2 Настройка AppVeyor Windows

Cоздан файл .appveyoer.yml

```yaml
image: Visual Studio 2019
platform: x64
configuration: Release
build_script:
  - cmake -H. -B_build
  - cmake --build _build --config Release

build_script:
  - mkdir build
  - cd build
  - cmake -G "%GENERATOR%" -A %PLATFORM% ..
  - cmake --build . --config %CONFIGURATION%
  - cmake --build . --config %CONFIGURATION% --target install

test: off
```

### 2.3 Исправление CmakeLists.txt

В данной работе у меня вышли некоторые ошибки при работе со старой сборкой
поэтому решил обновить некоторые файлы 

Для корректной работы CI были исправлены все CMakeLists.txt:

formatter_lib/CMakeLists.txt:

```
add_library(formatter STATIC formatter.cpp)
target_include_directories(formatter PUBLIC .)
install(TARGETS formatter ARCHIVE DESTINATION lib)
install(FILES formatter.h DESTINATION include)
```
formatter_ex/CMakeLists.txt:

```
add_library(formatter_ex STATIC formatter_ex.cpp)
target_link_libraries(formatter_ex formatter)
target_include_directories(formatter_ex PUBLIC .)
install(TARGETS formatter_ex ARCHIVE DESTINATION lib)
install(FILES formatter_ex.h DESTINATION include
```
solver_lib/CMakeLists.txt:
```
add_library(solver_lib STATIC solver.cpp)
target_include_directories(solver_lib PUBLIC .)
install(TARGETS solver_lib ARCHIVE DESTINATION lib)
install(FILES solver.h DESTINATION include)
```
hello_world/CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.10)
project(hello_world)

add_executable(hello_world main.cpp)
target_link_libraries(hello_world formatter_ex)
install(TARGETS hello_world RUNTIME DESTINATION bin)
```

