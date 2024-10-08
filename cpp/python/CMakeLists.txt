find_package(Python3 COMPONENTS Interpreter Development)

if(PYTHON3_FOUND)
    if(NOT EXISTS ${PROJECT_SOURCE_DIR}/python/pybind11/CMakeLists.txt)
        message(STATUS "pybind11 dependency not available.")
        find_package(Git)
        if(GIT_FOUND)
            message(STATUS "Downloading pybind11 repository...")
            if (NOT EXISTS ${PROJECT_SOURCE_DIR}/.git) # Manages the case when an archive is used
                execute_process(COMMAND git init WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} OUTPUT_QUIET ERROR_QUIET)
                execute_process(COMMAND rm -Rf ${PROJECT_SOURCE_DIR}/python/pybind11 OUTPUT_QUIET ERROR_QUIET)
                execute_process(COMMAND git submodule add -f https://github.com/pybind/pybind11 python/pybind11 WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} OUTPUT_QUIET ERROR_QUIET)
                execute_process(COMMAND git submodule update --init --recursive WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} OUTPUT_QUIET ERROR_QUIET)
            else() # When using a clone
                execute_process(COMMAND git submodule update --init --recursive WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} OUTPUT_QUIET ERROR_QUIET)
            endif()
            if(NOT EXISTS ${PROJECT_SOURCE_DIR}/python/pybind11/CMakeLists.txt)
                message(ERROR "pybind11 download failure.")
            else()
                message(STATUS "Downloaded pybind11 dependency successfully.")
            endif()
        else()
            message(ERROR "Git not found, pybind11 dependency could not be downloaded.")
        endif()
    endif()

    add_subdirectory(pybind11)
    add_subdirectory(src)

    add_library(pysignals MODULE
        $<TARGET_OBJECTS:pysignals-module-obj>
    )
    set_target_properties(pysignals PROPERTIES PREFIX "" OUTPUT_NAME "pysignals" SUFFIX ".so")
    target_link_libraries(pysignals PUBLIC signals PRIVATE pybind11::module)

    set(PYTHON_BINDINGS_INSTALL_DIR "${Python3_SITEARCH}"
        CACHE STRING "Install destination for Python targets")
    install(TARGETS pysignals DESTINATION "${PYTHON_BINDINGS_INSTALL_DIR}")

    add_subdirectory(example)

else()
    message(STATUS "Python3 not found: Python bindings module '[library]' will not be built.")
endif()