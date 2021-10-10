# Build the Math library used by this project. Depending on
# the USE_MYMATH option, this is build to be either an internal
# implementation or the system math library

add_library(MathFunctions MathFunctions.cpp)

# State that clients of this libraries will need to include
# the current source directory to find the header
target_include_directories(
	MathFunctions
	INTERFACE 
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
	)

option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)

	# Make the USE_MYMATH define visible in the source code, but
	# not to users of this library (i.e. the PRIVATE keyword)
	target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

	# Add the executable that will generate the table for SQRT 
	# calculation, which needs to be first built as part of the build
	# process
	add_executable(MakeTable MakeTable.cpp)

	# Add the command that will generate the sqrt table
	# using the executable
	add_custom_command(
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
		COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
		DEPENDS MakeTable
		)

	# Add the library that does the local implementation of sqrt,
	# whcih is always linked statically
	add_library(
		SqrtLibrary STATIC
		mysqrt.cpp
		${CMAKE_CURRENT_BINARY_DIR}/Table.h # depends on the
											# generated table
		)

	# State that we, and only we, need to include our binary dir
	# to find Table.h
	target_include_directories(SqrtLibrary PRIVATE
		${CMAKE_CURRENT_BINARY_DIR}
		)

  # state that SqrtLibrary need PIC when the default is shared
  # libraries
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )

	target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(
	TARGETS ${installable_libs} 
	DESTINATION lib
	EXPORT MathFunctionsTargets)
install(FILES MathFunctions.h DESTINATION include)
