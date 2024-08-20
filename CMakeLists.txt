cmake_minimum_required(VERSION 3.10)

# Project name
project(BitarrayWasm VERSION 1.0 LANGUAGES C)

# Find Python and the Python interpreter
find_package(Python3 COMPONENTS Interpreter Development REQUIRED)

# Add the source files for the C extension
set(SOURCE_FILES bitarray/_bitarray.c)

# Define the Python extension module
add_library(_bitarray MODULE ${SOURCE_FILES})

# Set the output directory for the compiled extension
set_target_properties(_bitarray PROPERTIES
    PREFIX ""
    SUFFIX ".so"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bitarray"
)

# Link against Python libraries
target_link_libraries(_bitarray PRIVATE Python3::Python)

# Optionally, add a custom command to run tests (you might have a different way of running tests)
add_custom_target(test
    COMMAND ${Python3_EXECUTABLE} setup.py test
    DEPENDS _bitarray
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

# Add a custom target to install the package
add_custom_target(install
    COMMAND ${Python3_EXECUTABLE} setup.py install
    DEPENDS _bitarray
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

# Custom command for building documentation
add_custom_target(doc
    COMMAND ${Python3_EXECUTABLE} update_doc.py
    COMMAND ${Python3_EXECUTABLE} setup.py sdist
    COMMAND twine check dist/*
    DEPENDS _bitarray
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

# Custom target for running MyPy
add_custom_target(mypy
    COMMAND mypy bitarray/*.pyi bitarray/test_*.py examples/*.py examples/huffman/*.py examples/sparse/*.py
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

# Clean up build artifacts
add_custom_target(clean
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/build
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_SOURCE_DIR}/dist
    COMMAND ${CMAKE_COMMAND} -E remove "${CMAKE_SOURCE_DIR}/bitarray/*.so"
    COMMAND ${CMAKE_COMMAND} -E remove "${CMAKE_SOURCE_DIR}/bitarray/*.o"
    COMMAND ${CMAKE_COMMAND} -E remove "${CMAKE_SOURCE_DIR}/bitarray/*.pyc"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/bitarray/__pycache__"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/examples/__pycache__"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/examples/*/__pycache__"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/.mypy_cache"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/bitarray/.mypy_cache"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/examples/.mypy_cache"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/examples/*/.mypy_cache"
)
