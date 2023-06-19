## Laboratory work V

## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.

Предворительная работа перед написанием Make-файлов:

```sh
% git clone https://github.com/arixixix/Tlab05.git
% cd Tlab05
% cd banking 
% touch CMakeLists.txt
% ls
Account.cpp	CMakeList.txt	Transaction.cpp
Account.h	CMakeLists.txt	Transaction.h
% rm CMakeList.txt 
% vim CMakeLists.txt 
% cd ..
% touch CMakeLists.txt
% vim CMakeLists.txt
```


#### banking/CMakeLists.txt

```sh
cmake_minimum_required(VERSION 3.16.3)

set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")

project(banking)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(banking STATIC Account.cpp Account.h Transaction.cpp Transaction.h)
```

#### CMakeLists.txt

```sh
cmake_minimum_required(VERSION 3.4)

SET(COVERAGE OFF CACHE BOOL "Coverage")
SET(CMAKE_CXX_COMPILER "/usr/bin/g++")

project(TestRunning)

add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/googletest" "gtest")

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/banking)

add_executable(RunTest ${CMAKE_CURRENT_SOURCE_DIR}/test.cpp)

if (COVERAGE)
    target_compile_options(RunTest PRIVATE --coverage)
    target_link_libraries(RunTest PRIVATE --coverage)
endif()

target_include_directories(RunTest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/banking)

target_link_libraries(RunTest PRIVATE gtest gtest_main gmock_main banking)
```


2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
  
Предварительная работа перед написанием тестов:

```sh
% git submodule add https://github.com/google/googletest.git
COMMIT & PUSH
% touch test.cpp
% vim test.cpp 
COMMIT & PUSH
```

#### test.cpp

```sh
#include <iostream>
#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>
#include <gmock/gmock.h>

class MockAccount : public Account {
public:
    MockAccount(int id, int balance):Account(id, balance){};
    MOCK_METHOD(void, Unlock, ());
    MOCK_METHOD(void, Lock, ());
    MOCK_METHOD(int, id, (), (const));
    MOCK_METHOD(void, ChangeBalance, (int diff), ());
    MOCK_METHOD(int, GetBalance, (), ());
};

class MockTransaction: public Transaction {
public:
    MOCK_METHOD(bool, Make, (Account& from, Account& to, int sum), ());
    MOCK_METHOD(void, set_fee, (int fee), ());
    MOCK_METHOD(int, fee, (), ());
};

TEST(Account, Balance_ID_Change) {
    MockAccount acc(1, 100);
    EXPECT_CALL(acc, GetBalance()).Times(3);
    EXPECT_CALL(acc, Lock()).Times(1);
    EXPECT_CALL(acc, Unlock()).Times(1);
    EXPECT_CALL(acc, ChangeBalance(testing::_)).Times(2);
    EXPECT_CALL(acc, id()).Times(1);
    acc.GetBalance();
    acc.id();
    acc.Unlock();
    acc.ChangeBalance(1000);
    acc.GetBalance();
    acc.ChangeBalance(2);
    acc.GetBalance();
    acc.Lock();
}

TEST(Account, Balance_ID_Change_2) {
    Account acc(0, 100);
    EXPECT_THROW(acc.ChangeBalance(50), std::runtime_error);
    acc.Lock();
    acc.ChangeBalance(50);
    EXPECT_EQ(acc.GetBalance(), 150);
    EXPECT_THROW(acc.Lock(), std::runtime_error);
    acc.Unlock();
}

TEST(Transaction, TransTest) {
    MockTransaction trans;
    MockAccount first(1, 100);
    MockAccount second(2, 250);
    MockAccount flat_org(3, 10000);
    MockAccount org(4, 5000);
    EXPECT_CALL(trans, set_fee(testing::_)).Times(1);
    EXPECT_CALL(trans, fee()).Times(1);
    EXPECT_CALL(trans, Make(testing::_, testing::_, testing::_)).Times(2);
    EXPECT_CALL(first, GetBalance()).Times(1);
    EXPECT_CALL(second, GetBalance()).Times(1);
    trans.set_fee(300);
    trans.Make(first, second, 2000);
    trans.fee();
    first.GetBalance();
    second.GetBalance();
    trans.Make(org, first, 1000);
}
```

3. Настройте сборочную процедуру на GitHub Actions.

Предварительная работа:

```sh
% mkdir .github          
% mkdir .github/workflows
% touch .github/workflows/main.yml
% vim .github/workflows/main.yml
COOMIT & PUSH
```

#### .github/workflows/main.yml

```sh
name: bank

on:
  push:
    branches: main

  workflow_dispatch:
      
jobs:

  BuildProject:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2

        - name: build banking
          shell: bash
          run: |
            ls
            cd banking
            cmake -H. -B_build
            cmake --build _build
          
  Testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: update
        run:  |
          git submodule update --init
          sudo apt install lcov
          sudo apt install g++-9
      
      - name: test
        shell: bash
        run: |
          mkdir _build && cd _build
          CXX=/usr/bin/g++-9 cmake -DCOVERAGE=1 ..
          cmake --build .
          ./RunTest
          lcov -t "banking" -o lcov.info -c -d .
          
      - name: CovBeg
        uses: coverallsapp/github-action@master
        with:
          github-token: ${{ secrets.github_token }}
          #parallel: true
          path-to-lcov: ./_build/lcov.info
          coveralls-endpoint: https://coveralls.io

      - name: CovFin
        uses: coverallsapp/github-action@master
        with:
          github-token: ${{ secrets.github_token }}
```


4. Настройте [Coveralls.io](https://coveralls.io/).

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```
