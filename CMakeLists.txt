cmake_minimum_required(VERSION 3.10)

# Project name and version
project(Tutorial VERSION 1.0)


# Create an interface target for compiler flags
add_library(tutorial_compiler_flags INTERFACE)

# Specify C++11 standard
target_compile_features(
	tutorial_compiler_flags
	INTERFACE 
	cxx_std_11)

# Add compiler-dependent flags
set(gcc_like_compiler
	"$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_compiler "$<COMPILE_LANG_AND_ID:CXX,MSVC>")

target_compile_options(
	tutorial_compiler_flags
	INTERFACE
	 "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
	 "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
	)

# Control where the static and shared libraries are built
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# This will generate TutorialConfig.h in the build directory
configure_file(TutorialConfig.h.in TutorialConfig.h)

# Add the MathFunctions library
add_subdirectory(MathFunctions)

# Add the executable (the main target)
add_executable(Tutorial tutorial.cpp)
target_link_libraries(Tutorial PUBLIC MathFunctions)

# Setting the include directory as the project binary dir
# means that we can include TutorialConfig.h with angled brackets

target_include_directories(Tutorial PUBLIC 
						   "${PROJECT_BINARY_DIR}"
						   )

# Installation steps. Use --prefix to specify the installation
# folder and avoid file access restriction problems, e.g.
# cmake --install . --config Debug --prefix "./"

install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
	DESTINATION include
	)

install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

# install the configuration file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )

export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
# Testing start here

enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
	PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number*"
	)

# define a function to simplify adding the next tests

function(do_test target arg result)
	add_test(NAME Comp${arg} COMMAND ${target} ${arg})
	set_tests_properties(Comp${arg}
		PROPERTIES PASS_REGULAR_EXPRESSION ${result}
		)
endfunction(do_test)

# do a bunch of result based tests

do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")

# Include CPack for installer generation. Fun fact:
# The name 'cpack' conflicts with Chocolatey, use
# full path to invoke

include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
