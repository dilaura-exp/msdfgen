cmake_minimum_required(VERSION 3.16.0 FATAL_ERROR)
set(PROJECT_NAME msdfgen)

add_subdirectory(freetype)

################################################################################
# Target
################################################################################
add_library(${PROJECT_NAME} STATIC "")

use_props(${PROJECT_NAME} "${CMAKE_CONFIGURATION_TYPES}" "${DEFAULT_CXX_PROPS}")
#set(ROOT_NAMESPACE imgui)

set_target_properties(${PROJECT_NAME} PROPERTIES
    VS_GLOBAL_KEYWORD "Win32Proj"
)

################################################################################
# Source groups
################################################################################


function(ADD_SOURCES_RECURSE RELATIVE_DIR FILE_EXTENSION)
    file(GLOB_RECURSE Source_Files LIST_DIRECTORIES false RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${RELATIVE_DIR}/*.${FILE_EXTENSION})
    target_sources(${PROJECT_NAME} PUBLIC ${Source_Files})
endfunction()

ADD_SOURCES_RECURSE(core h)
ADD_SOURCES_RECURSE(core hpp)
ADD_SOURCES_RECURSE(core cpp)
ADD_SOURCES_RECURSE(ext h)
ADD_SOURCES_RECURSE(ext hpp)
ADD_SOURCES_RECURSE(ext cpp)
ADD_SOURCES_RECURSE(lib cpp)
ADD_SOURCES_RECURSE(include h)

################################################################################
# Output directory
################################################################################
set_target_properties(${PROJECT_NAME} PROPERTIES
    OUTPUT_DIRECTORY_DEBUG   "${CMAKE_SOURCE_DIR}/Bin/$<CONFIG>-${CMAKE_VS_PLATFORM_NAME}/${PROJECT_NAME}/"
    OUTPUT_DIRECTORY_DIST    "${CMAKE_SOURCE_DIR}/Bin/$<CONFIG>-${CMAKE_VS_PLATFORM_NAME}/${PROJECT_NAME}/"
    OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/Bin/$<CONFIG>-${CMAKE_VS_PLATFORM_NAME}/${PROJECT_NAME}/"
)
set_target_properties(${PROJECT_NAME} PROPERTIES
    INTERPROCEDURAL_OPTIMIZATION_DIST    "TRUE"
    INTERPROCEDURAL_OPTIMIZATION_RELEASE "TRUE"
    LINKER_LANGUAGE CXX
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
)

################################################################################
# MSVC runtime library
################################################################################
get_property(MSVC_RUNTIME_LIBRARY_DEFAULT TARGET ${PROJECT_NAME} PROPERTY MSVC_RUNTIME_LIBRARY)
string(CONCAT "MSVC_RUNTIME_LIBRARY_STR"
    $<$<CONFIG:Debug>:
        MultiThreadedDebug
    >
    $<$<CONFIG:Dist>:
        MultiThreaded
    >
    $<$<CONFIG:Release>:
        MultiThreaded
    >
    #$<$<NOT:$<OR:$<CONFIG:Dist>,$<CONFIG:Release>>>:${MSVC_RUNTIME_LIBRARY_DEFAULT}>
)
set_target_properties(${PROJECT_NAME} PROPERTIES MSVC_RUNTIME_LIBRARY ${MSVC_RUNTIME_LIBRARY_STR})


################################################################################
# Include Directories
################################################################################

target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/include;"
                                                  "${CMAKE_CURRENT_SOURCE_DIR}/freetype/include;")

################################################################################
# Compile definitions
################################################################################
target_compile_definitions(${PROJECT_NAME} PRIVATE
    "$<$<CONFIG:Debug>:"
        "_DEBUG;"
    ">"
    "$<$<CONFIG:Dist>:"
        "NDEBUG"
    ">"
    "$<$<CONFIG:Release>:"
        "NDEBUG"
    ">"
    "_CONSOLE;"
    "UNICODE;"
    "_UNICODE;"

	"FT2_BUILD_LIBRARY"
	"_CRT_SECURE_NO_WARNINGS"
	"_CRT_NONSTDC_NO_WARNINGS"
)

################################################################################
# Compile and link options
################################################################################
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Dist>:
            /Oi;
            /Gy;
            /arch:AVX2; #vectorextensions
            /arch:BMI; /arch:POPCNT; /arch:LZCNT; /arch:F16C; #isaextensions
        >
        $<$<CONFIG:Release>:
            /Oi;
            /Gy;
            /arch:AVX2; #vectorextensions
            /arch:BMI; /arch:POPCNT; /arch:LZCNT; /arch:F16C; #isaextensions
        >
        /permissive-;
        /sdl;
        /W3;
        ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
        ${DEFAULT_CXX_EXCEPTION_HANDLING}
    )
    target_link_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Dist>:
            /OPT:REF;
            /OPT:ICF
        >
        $<$<CONFIG:Release>:
            /OPT:REF;
            /OPT:ICF
        >
        /DEBUG;
        /SUBSYSTEM:CONSOLE
    )
endif()


target_link_libraries(${PROJECT_NAME} PRIVATE freetype)
