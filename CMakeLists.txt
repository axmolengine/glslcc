cmake_minimum_required(VERSION 3.0)
project(axslcc)

 if (WIN32)
    set (CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
    set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")
    set (CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
endif()

function(remove_compile_options DEST_VAR COMPILER_FLAGS FLAGS)
        separate_arguments(FLAGS)
        foreach(FLAG ${FLAGS})
            string(REPLACE "${FLAG}" "" COMPILER_FLAGS "${COMPILER_FLAGS}")
        endforeach()
        set(${DEST_VAR} ${COMPILER_FLAGS} PARENT_SCOPE)
endfunction()

if(WIN32)
    option(STATIC_LINK "Statically link with system standard libraries. Affects release build only" ON)
else()
    option(STATIC_LINK "Statically link with system standard libraries. Affects release build only" OFF)
endif()
option(ENABLE_SPVREMAPPER "Enables building of SPVRemapper" ON)
option(ENABLE_AMD_EXTENSIONS "Enables support of AMD-specific extensions" ON)
option(ENABLE_GLSLANG_BINARIES "Builds glslangValidator and spirv-remap" ON)
option(ENABLE_NV_EXTENSIONS "Enables support of Nvidia-specific extensions" ON)
option(ENABLE_HLSL "Enables HLSL input support" OFF)
option(ENABLE_OPT "Enables spirv-opt capability if present" ON)

option(USE_CCACHE "Use ccache" OFF)
if (WIN32)
    option(ENABLE_D3D11_COMPILER "Use Direct3D11 compiler (d3dcompiler.lib + d3dcompiler_47.dll)" ON)
endif()

set(SX_BUILD_TESTS OFF CACHE BOOL "" FORCE)
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

if(ENABLE_AMD_EXTENSIONS)
    add_definitions(-DAMD_EXTENSIONS)
endif(ENABLE_AMD_EXTENSIONS)

if(ENABLE_NV_EXTENSIONS)
    add_definitions(-DNV_EXTENSIONS)
endif(ENABLE_NV_EXTENSIONS)

if(ENABLE_HLSL)
    add_definitions(-DENABLE_HLSL)
endif(ENABLE_HLSL)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

if (MSVC)
	remove_compile_options(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}" "/GR")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} /wd4530")
	
	if (STATIC_LINK)
		remove_compile_options(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}" "/MD")
		remove_compile_options(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}" "/MD")
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
		set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
	endif()
endif()

set(ENABLE_CTEST FALSE CACHE BOOL "" FORCE)
set(SPIRV_CROSS_ENABLE_TESTS OFF CACHE BOOL "" FORCE)

# halx99: optimize require https://github.com/KhronosGroup/SPIRV-Tools?
set(ALLOW_EXTERNAL_SPIRV_TOOLS OFF CACHE BOOL "" FORCE)

if (NOT (IS_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/3rdparty/glslang"))
    message(STATUS "Downloading glslang ...")
    find_program(GIT_COMMAND NAMES git REQUIRED)
    execute_process(COMMAND ${GIT_COMMAND} clone "https://github.com/KhronosGroup/glslang.git" "${CMAKE_CURRENT_LIST_DIR}/3rdparty/glslang")
    find_program(PYTHON_COMMAND NAMES python3 python REQUIRED)
    execute_process(COMMAND ${PYTHON_COMMAND} "${CMAKE_CURRENT_LIST_DIR}/3rdparty/glslang/update_glslang_sources.py" 
        WORKING_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/3rdparty/glslang")
endif()
add_subdirectory(3rdparty/glslang)
add_subdirectory(3rdparty/spirv-cross)
add_subdirectory(3rdparty/sx)
add_subdirectory(src)

