# external/imgui/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

include(FetchContent)

# Fetch GLFW (this part is fine)
FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
)
FetchContent_MakeAvailable(glfw)

# Fetch ImGui manually (since it has no CMakeLists.txt)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG docking  # Use the docking branch
    GIT_SHALLOW TRUE
)
FetchContent_Populate(imgui) # Clones the repo into imgui_SOURCE_DIR

# Project name (optional)
project(ImGui)

set(IMGUI_DIR "${imgui_SOURCE_DIR}")

# List of all possible backend files (both .cpp and .h)
set(ALL_BACKENDS 
	"glfw"
	"opengl3"
)

# Collect ImGui source files manually
file(GLOB IMGUI_SOURCES 
    "${IMGUI_DIR}/*.cpp"
    "${IMGUI_DIR}/*.h"
)

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

# Create an ImGui static library
add_library(imgui STATIC ${IMGUI_SOURCES} ${BACKENDS})

target_link_libraries(imgui PRIVATE glfw)

# Include directories
target_include_directories(imgui PUBLIC 
    ${IMGUI_DIR} 
    ${glfw_SOURCE_DIR}/include
)
