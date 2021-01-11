################################################################################
cmake_minimum_required(VERSION 3.8)
project(FloatTetwild)
################################################################################

# FloatTetwild options
option(FLOAT_TETWILD_ENABLE_TBB         "Enable TBB"                    ON)
option(FLOAT_TETWILD_USE_FLOAT          "Use floats instead of double"  OFF)
option(FLOAT_TETWILD_WITH_SANITIZERS    "Use sanitizers"                OFF)
option(FLOAT_TETWILD_WITH_EXACT_ENVELOPE    "Use exact envelope"        OFF)

# Options for libigl modules
option(LIBIGL_USE_STATIC_LIBRARY     "Use libigl as static library" OFF)
option(LIBIGL_WITH_COMISO            "Use CoMiso"                   OFF)
option(LIBIGL_WITH_EMBREE            "Use Embree"                   OFF)
option(LIBIGL_WITH_OPENGL            "Use OpenGL"                   OFF)
option(LIBIGL_WITH_OPENGL_GLFW       "Use GLFW"                     OFF)
option(LIBIGL_WITH_OPENGL_GLFW_IMGUI "Use ImGui"                    OFF)
option(LIBIGL_WITH_PNG               "Use PNG"                      OFF)
option(LIBIGL_WITH_TETGEN            "Use Tetgen"                   OFF)
option(LIBIGL_WITH_TRIANGLE          "Use Triangle"                 OFF)
option(LIBIGL_WITH_PREDICATES        "Use exact predicates"         OFF)
option(LIBIGL_WITH_XML               "Use XML"                      OFF)

# Find dependencies
find_package(Eigen3 CONFIG REQUIRED)
find_package(Geogram CONFIG REQUIRED)
find_package(fmt CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(libigl CONFIG REQUIRED)
find_package(Threads REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)
find_path(GMP_INCLUDE_DIR "gmp.h" )
find_library(GMP_LIBRARIES NAMES gmp mpir)
if(NOT GMP_INCLUDE_DIR OR NOT GMP_LIBRARIES)
	MESSAGE(FATAL_ERROR "Cannot find GMP or MPIR")
endif()
find_package(CLI11 CONFIG REQUIRED)

#include(FetchContent)
#FetchContent_Declare(windingnumber
#	PREFIX         ${FLOAT_TETWILD_EXTERNAL}
#    GIT_REPOSITORY https://github.com/alecjacobson/WindingNumber.git
#    GIT_TAG        1e6081e52905575d8e98fb8b7c0921274a18752f
#)
#FetchContent_MakeAvailable(windingnumber)
#FetchContent_GetProperties(windingnumber)
#if(NOT windingnumber_POPULATED)
#  FetchContent_Populate(windingnumber)
#endif()
#set(windingnumber_SOURCES
#	${windingnumber_SOURCE_DIR}/SYS_Math.h
#	${windingnumber_SOURCE_DIR}/SYS_Types.h
#	${windingnumber_SOURCE_DIR}/UT_Array.cpp
#	${windingnumber_SOURCE_DIR}/UT_Array.h
#	${windingnumber_SOURCE_DIR}/UT_ArrayImpl.h
#	${windingnumber_SOURCE_DIR}/UT_BVH.h
#	${windingnumber_SOURCE_DIR}/UT_BVHImpl.h
#	${windingnumber_SOURCE_DIR}/UT_FixedVector.h
#	${windingnumber_SOURCE_DIR}/UT_ParallelUtil.h
#	${windingnumber_SOURCE_DIR}/UT_SmallArray.h
#	${windingnumber_SOURCE_DIR}/UT_SolidAngle.cpp
#	${windingnumber_SOURCE_DIR}/UT_SolidAngle.h
#	${windingnumber_SOURCE_DIR}/VM_SIMD.h
#	${windingnumber_SOURCE_DIR}/VM_SSEFunc.h
#)
#add_library(fast_winding_number STATIC ${windingnumber_SOURCES})
#target_link_libraries(fast_winding_number PRIVATE TBB::tbb)
#target_compile_features(fast_winding_number PRIVATE ${CXX17_FEATURES})
#target_include_directories(fast_winding_number PUBLIC "${FLOAT_TETWILD_EXTERNAL}/")

if(FLOAT_TETWILD_ENABLE_TBB)
	find_package(TBB CONFIG REQUIRED)
endif()
if(FLOAT_TETWILD_WITH_EXACT_ENVELOPE)
	#TODO cleanup CMakeLists https://github.com/wangbolun300/fast-envelope
endif()

################################################################################
# FloatTetwild library
################################################################################
add_library(${PROJECT_NAME})
set(SOURCES
	src/AABBWrapper.h
	src/AABBWrapper.cpp
	src/auto_table.hpp
	src/auto_table.cpp
	src/CSGTreeParser.hpp
	src/CSGTreeParser.cpp
	src/LocalOperations.h
	src/LocalOperations.cpp
	src/EdgeSplitting.h
	src/EdgeSplitting.cpp
	src/EdgeCollapsing.h
	src/EdgeCollapsing.cpp
	src/EdgeSwapping.h
	src/EdgeSwapping.cpp
	src/VertexSmoothing.h
	src/VertexSmoothing.cpp
	src/MeshImprovement.h
	src/MeshImprovement.cpp
	src/FloatTetwild.h
	src/FloatTetwild.cpp
	src/FloatTetDelaunay.h
	src/FloatTetDelaunay.cpp
	src/TriangleInsertion.cpp
	src/TriangleInsertion.h
	src/CutMesh.cpp
	src/CutMesh.h
	src/intersections.h
	src/intersections.cpp
	src/Simplification.h
	src/Simplification.cpp
	src/Mesh.cpp
	src/Mesh.hpp
	src/MeshIO.hpp
	src/MeshIO.cpp
	src/Logger.hpp
	src/Logger.cpp
	src/Parameters.h
	src/Types.hpp
	src/Statistics.h
	src/external/Predicates.cpp
	src/external/Predicates.hpp
	src/external/mesh_AABB.h
	src/external/mesh_AABB.cpp
	src/external/MshLoader.h
	src/external/MshLoader.cpp
	src/external/MshSaver.h
	src/external/MshSaver.cpp
	src/external/Exception.h
	src/external/triangle_triangle_intersection.cpp
	src/external/getRSS.c
	src/external/get_mem.h
	src/external/get_mem.cpp
	src/external/bfs_orient.h
	src/external/bfs_orient.cpp
	src/external/FastWindingNumber.hpp
	src/external/FastWindingNumber.cpp
	src/external/WindingNumber.h
	src/external/Rational.h
)
if(NOT WIN32)
    list(APPEND SOURCES src/external/predicates.c) # linker error if present on WIN32
endif()
foreach(filepath IN ITEMS ${SOURCES})
	get_filename_component(filename "${filepath}" NAME)
	if(${filename} MATCHES ".*\.(hpp|h|ipp)$")
		configure_file(${filepath} ${PROJECT_BINARY_DIR}/include/floattetwild/${filename})
	endif()
endforeach()
target_sources(${PROJECT_NAME} PRIVATE ${SOURCES})

# Public include directory for FloatTetwild
target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_BINARY_DIR}/include)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_11)

################################################################################
# Required libraries
################################################################################

if(FLOAT_TETWILD_USE_FLOAT)
	target_compile_definitions(${PROJECT_NAME} PUBLIC -DFLOAT_TETWILD_USE_FLOAT)
endif()

target_include_directories(${PROJECT_NAME} PUBLIC ${GMP_INCLUDE_DIRS})
target_link_libraries(
	${PROJECT_NAME}
	PUBLIC
		Eigen3::Eigen
		Geogram::geogram
		fmt::fmt-header-only
		spdlog::spdlog_header_only
		nlohmann_json::nlohmann_json
		igl::core
		Threads::Threads
		${GMP_LIBRARIES}
)
if(FLOAT_TETWILD_ENABLE_TBB)
	target_link_libraries(${PROJECT_NAME} PUBLIC TBB::tbb)
	target_compile_definitions(${PROJECT_NAME} PUBLIC FLOAT_TETWILD_USE_TBB)
endif()
if(FLOAT_TETWILD_WITH_EXACT_ENVELOPE)
	target_link_libraries(${PROJECT_NAME} PUBLIC FastEnvelope)
	target_compile_definitions(${PROJECT_NAME} PUBLIC NEW_ENVELOPE)
endif()


################################################################################
# FloatTetwild executable
################################################################################
add_executable(${PROJECT_NAME}_bin src/main.cpp)
target_compile_features(${PROJECT_NAME}_bin PUBLIC ${CXX14_FEATURES})
target_link_libraries(${PROJECT_NAME}_bin PRIVATE ${PROJECT_NAME} CLI11::CLI11 Eigen3::Eigen)

if(TARGET igl::tetgen)
	target_link_libraries(${PROJECT_NAME} PUBLIC igl::tetgen)
	target_compile_definitions(${PROJECT_NAME} PUBLIC -DLIBIGL_WITH_TETGEN)
endif()



################################################################################
# Build static executable on Windows
################################################################################
foreach(flag_var
    CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
    CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
  if(${flag_var} MATCHES "/MD")
    string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
  endif()
endforeach()

