# @brief Build configuration
# @author Wolfram Rösler <wolfram@roesler-ac.de>
# @date 2017-02-12

cmake_minimum_required(VERSION 2.8)

# The name of the object file with the version number function
set(VERSION ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/version.o)

# Vanilla project configuration
project(version)
add_executable(version main.cpp)
target_link_libraries(version ${VERSION})

# The actual magic happens here
execute_process(COMMAND touch ${VERSION})
add_custom_command(TARGET version PRE_LINK
    COMMAND bash ${CMAKE_SOURCE_DIR}/makeversion.sh -o ${VERSION}
)
