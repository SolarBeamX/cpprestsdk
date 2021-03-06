project(casablanca-ios NONE)
cmake_minimum_required(VERSION 3.1)

enable_testing()

if (CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET)
  set (ENV{CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET} ${CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET})
endif()

set(TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/ios-cmake/ios.toolchain.cmake")

set(SIM64_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/build.x86_64" CACHE INTERNAL "")
set(SIM64_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../Release" CACHE INTERNAL "")

set(ARM64_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/build.arm64" CACHE INTERNAL "")
set(ARM64_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../Release" CACHE INTERNAL "")
add_test(NAME ios_runner
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../Release/tests/common/testrunner/ios
    COMMAND xcodebuild test -project ios_runner.xcodeproj -configuration=${CMAKE_BUILD_TYPE} -scheme ios_runner -destination "platform=iOS Simulator,name=iPhone 6" LIBRARY_SEARCH_PATH=${SIM_BINARY_DIR}
  )

file(MAKE_DIRECTORY ${SIM64_BINARY_DIR})
execute_process(WORKING_DIRECTORY ${SIM64_BINARY_DIR}
  COMMAND ${CMAKE_COMMAND}
    -GXcode
    -DCMAKE_TOOLCHAIN_FILE=${TOOLCHAIN_FILE}
    -DIOS_PLATFORM=SIMULATOR64
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    "${SIM64_SOURCE_DIR}"
)

file(MAKE_DIRECTORY ${ARM64_BINARY_DIR})
execute_process(WORKING_DIRECTORY ${ARM64_BINARY_DIR}
  COMMAND ${CMAKE_COMMAND}
    -GXcode
    -DCMAKE_TOOLCHAIN_FILE=${TOOLCHAIN_FILE}
    -DIOS_PLATFORM=OS64
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    "${ARM64_SOURCE_DIR}"
    )


## Simulator x86_64 version
add_custom_target(sim64
  COMMAND ${CMAKE_COMMAND}
    --build ${SIM64_BINARY_DIR}
    --config ${CMAKE_BUILD_TYPE}
    COMMENT "Building for x86_64 (simulator)"
VERBATIM
)

## ARM64 version
add_custom_target(arm64
  COMMAND ${CMAKE_COMMAND}
    --build ${ARM64_BINARY_DIR}
    --config ${CMAKE_BUILD_TYPE}
  COMMENT "Building for arm64"
  VERBATIM
)

set(LIB_CPPREST libcpprest.a)
add_custom_command(
  OUTPUT ${LIB_CPPREST}
  COMMAND lipo -create
    -output "${CMAKE_CURRENT_BINARY_DIR}/${LIB_CPPREST}"
    ${SIM64_BINARY_DIR}/Binaries/${CMAKE_BUILD_TYPE}/${LIB_CPPREST}
    ${ARM64_BINARY_DIR}/Binaries/${CMAKE_BUILD_TYPE}/${LIB_CPPREST}
  DEPENDS
    sim64
    arm64
    "${SIM64_BINARY_DIR}/Binaries/${CMAKE_BUILD_TYPE}/${LIB_CPPREST}"
    "${ARM64_BINARY_DIR}/Binaries/${CMAKE_BUILD_TYPE}/${LIB_CPPREST}"
  VERBATIM
)

add_custom_target(cpprest ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${LIB_CPPREST})
