# external/imgui/CMakeLists.txt

# Set minimum CMake version
cmake_minimum_required(VERSION 3.10)

include(FetchContent)

FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
)

execute_process(
    COMMAND git submodule update --init --recursive
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    RESULT_VARIABLE GIT_SUBMODULE_RESULT
    ERROR_VARIABLE GIT_SUBMODULE_ERROR
)

# Ensure the correct branch is checked out for ImGui submodule
execute_process(
    COMMAND git -C ${CMAKE_SOURCE_DIR}/imgui fetch origin docking
    COMMAND git -C ${CMAKE_SOURCE_DIR}/imgui checkout docking
    COMMAND git -C ${CMAKE_SOURCE_DIR}/imgui pull origin docking
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    RESULT_VARIABLE GIT_RESULT
)

if (NOT GIT_RESULT EQUAL "0")
    message(FATAL_ERROR "Failed to fetch and checkout ImGui docking branch")
endif()

FetchContent_MakeAvailable(glfw)

# Project name (Optional)
project(ImGui)

set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/imgui")

# List of all possible backend files (both .cpp and .h)
set(ALL_BACKENDS 
	"glfw"
	"opengl3"
)

# Specify source files
file(GLOB IMGUI_SOURCES "${IMGUI_DIR}/*.cpp" "${IMGUI_DIR}/*.h")

list(GET ALL_BACKENDS 0 FIRST_BACKEND)

set(BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${FIRST_BACKEND}.h")
list(APPEND BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${FIRST_BACKEND}.cpp")
foreach(backend ${ALL_BACKENDS})
	list(FIND BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${backend}.h" ITEM_INDEX)
	if(ITEM_INDEX EQUAL -1)
		list(APPEND BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${backend}.h")
	endif()
	list(FIND BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp" ITEM_INDEX)
	if(ITEM_INDEX EQUAL -1)
		list(APPEND BACKENDS "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp")
	endif()
endforeach()

# Create a shared library for ImGui
add_library(imgui SHARED ${IMGUI_SOURCES} ${BACKENDS})

target_link_libraries(imgui PRIVATE
        glfw
)

# Add include directories
target_include_directories(imgui PUBLIC ${IMGUI_DIR} ${glfw_SOURCE_DIR}/include)
