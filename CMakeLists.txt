cmake_minimum_required(VERSION 3.1.3 FATAL_ERROR)

project(gl3w)

set(CMAKE_VERBOSE_MAKEFILE false)
set(CMAKE_POSITION_INDEPENDENT_CODE ON) # -fPIC

set(PYTHON_EXECUTABLE "python" CACHE STRING "")
option(GL3W_INSTALL "Generate installation target" ON)

set(SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/src)
set(HEADER_DIR ${CMAKE_CURRENT_BINARY_DIR}/include)

set(HEADER_FILES
	"${HEADER_DIR}/GL/gl3w.h"
	"${HEADER_DIR}/GL/glcorearb.h"
)

set(SOURCE_FILES
	"${SOURCE_DIR}/gl3w.c"
)

# add command to create the header and source files
add_custom_command(
	OUTPUT
		"${SOURCE_DIR}/gl3w.c"
		"${HEADER_DIR}/GL/gl3w.h"
		"${HEADER_DIR}/GL/glcorearb.h"
	COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/gl3w_gen.py
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/gl3w_gen.py
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)

# add pseudo target that depends on the generated files
add_custom_target(
	gl3w_gen ALL
	DEPENDS
		"${SOURCE_DIR}/gl3w.c"
		"${HEADER_DIR}/GL/gl3w.h"
		"${HEADER_DIR}/GL/glcorearb.h"
)

# create gl3w target
add_library(${PROJECT_NAME} ${SOURCE_FILES})
add_dependencies(${PROJECT_NAME} gl3w_gen)
include_directories(${HEADER_DIR})
target_link_libraries(${PROJECT_NAME})

if (GL3W_INSTALL)
	install(DIRECTORY ${gl3w_BINARY_DIR}/include/GL DESTINATION include
		FILES_MATCHING PATTERN gl3w.h PATTERN glcorearb.h)
	install(FILES ${gl3w_BINARY_DIR}/libgl3w.a DESTINATION lib)
endif ()