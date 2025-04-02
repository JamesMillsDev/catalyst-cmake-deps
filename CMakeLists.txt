# external/imgui/CMakeLists.txt

# Set minimum CMake version
cmake_minimum_required(VERSION 3.10)

# Project name (Optional)
project(ImGui)

set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/imgui")

# Remove the backends directory after removing the unwanted files
file(REMOVE_RECURSE "${IMGUI_DIR}")

execute_process(
    COMMAND git submodule update --init --recursive
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    RESULT_VARIABLE GIT_SUBMODULE_RESULT
    ERROR_VARIABLE GIT_SUBMODULE_ERROR
)

# Specify source files
set(IMGUI_SOURCES
    "${IMGUI_DIR}/imgui.cpp"
    "${IMGUI_DIR}/imgui_draw.cpp"
    "${IMGUI_DIR}/imgui_tables.cpp"
    "${IMGUI_DIR}/imgui_widgets.cpp"
)

# List of all possible backend files (both .cpp and .h)
set(ALL_BACKENDS 
	"dx9"
	"dx11"
	"dx12"
	"vulkan"
	"sdl"
	"win32"
	"metal"
	"glfw"
	"opengl3"
)

# List of backends you want to include (both .cpp and .h files)
set(INCLUDE_BACKENDS 
	"glfw" 
	"opengl3"
)

# Move the included backends to the main directory (both .cpp and .h files)
foreach(backend ${INCLUDE_BACKENDS})
	if(EXISTS "${IMGUI_DIR}/backends/imgui_impl_${backend}.h")
		file(RENAME "${IMGUI_DIR}/backends/imgui_impl_${backend}.h" "${IMGUI_DIR}/imgui_impl_${backend}.h")
	endif()
	if(EXISTS "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp")
		file(RENAME "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp" "${IMGUI_DIR}/imgui_impl_${backend}.cpp")
	endif()
endforeach()

# Iterate through all backend files, and remove those not in the INCLUDE_BACKENDS list
foreach(backend ${ALL_BACKENDS})
    if(NOT "${backend}" IN_LIST INCLUDE_BACKENDS)
        file(REMOVE "${IMGUI_DIR}/backends/imgui_impl_${backend}.h")
		if(EXISTS "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp")
			file(REMOVE "${IMGUI_DIR}/backends/imgui_impl_${backend}.cpp")
		elseif(EXISTS "${IMGUI_DIR}/backends/imgui_impl_${backend}.mm")
			file(REMOVE "${IMGUI_DIR}/backends/imgui_impl_${backend}.mm")
		endif()
    endif()
endforeach()

# Remove the backends directory after removing the unwanted files
file(REMOVE_RECURSE "${IMGUI_DIR}/backends")

list(GET INCLUDE_BACKENDS 0 FIRST_BACKEND)

set(BACKENDS "${IMGUI_DIR}/imgui_impl_${FIRST_BACKEND}.cpp")
foreach(backend ${INCLUDE_BACKENDS})
	list(FIND BACKENDS "${IMGUI_DIR}/imgui_impl_${backend}.cpp" ITEM_INDEX)
	if(ITEM_INDEX EQUAL -1)
		list(APPEND BACKENDS "${IMGUI_DIR}/imgui_impl_${backend}.cpp")
	endif()
endforeach()

# Create a shared library for ImGui
add_library(imgui SHARED ${IMGUI_SOURCES} ${BACKENDS})

# Add include directories
target_include_directories(imgui PUBLIC ${IMGUI_DIR})