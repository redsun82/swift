set(SWIFT_INSTALL_PACKAGE_DIR lib${LLVM_LIBDIR_SUFFIX}/cmake/swift)
set(swift_cmake_builddir "${SWIFT_BINARY_DIR}/${SWIFT_INSTALL_PACKAGE_DIR}")

# Generate build-tree exports list only
set(SWIFT_EXPORTS_FILE ${swift_cmake_builddir}/SwiftExports.cmake)
get_property(SWIFT_EXPORTS GLOBAL PROPERTY SWIFT_EXPORTS)
get_property(SWIFT_BUILDTREE_EXPORTS GLOBAL PROPERTY SWIFT_BUILDTREE_EXPORTS)

set(SWIFT_CONFIG_EXPORTS ${SWIFT_EXPORTS} ${SWIFT_BUILDTREE_EXPORTS})
export(TARGETS ${SWIFT_CONFIG_EXPORTS} FILE ${SWIFT_EXPORTS_FILE})


set(SWIFT_INCLUDE_DIRS ${SWIFT_INCLUDE_DIR} ${SWIFT_MAIN_INCLUDE_DIR})
set(SWIFT_LIBRARY_DIRS ${SWIFT_LIBRARY_OUTPUT_INTDIR})

configure_file(
  SwiftConfig.cmake.in
  ${swift_cmake_builddir}/SwiftConfig.cmake
  @ONLY)


# Generate install-tree CMake files
set(SWIFT_CONFIG_CODE "
# Compute the installation prefix from this LLVMConfig.cmake file location.
get_filename_component(SWIFT_INSTALL_PREFIX \"\${CMAKE_CURRENT_LIST_FILE}\" PATH)")
# Construct the proper number of get_filename_component(... PATH)
# calls to compute the installation prefix.
string(REGEX REPLACE "/" ";" _count "${SWIFT_INSTALL_PACKAGE_DIR}")
foreach(p ${_count})
  set(SWIFT_CONFIG_CODE "${SWIFT_CONFIG_CODE}
get_filename_component(SWIFT_INSTALL_PREFIX \"\${SWIFT_INSTALL_PREFIX}\" PATH)")
endforeach(p)
set(SWIFT_INCLUDE_DIRS "\${SWIFT_INSTALL_PREFIX}/include")
set(SWIFT_LIBRARY_DIRS "\${SWIFT_INSTALL_PREFIX}/lib")
set(SWIFT_CMAKE_DIR "\${SWIFT_INSTALL_PREFIX}/${SWIFT_INSTALL_PACKAGE_DIR}")
set(SWIFT_BINARY_DIR "\${SWIFT_INSTALL_PREFIX}")

set(SWIFT_EXPORTS_FILE "\${SWIFT_CMAKE_DIR}/SwiftExports.cmake")
set(SWIFT_CONFIG_EXPORTS ${SWIFT_EXPORTS})

install(TARGETS ${SWIFT_CONFIG_EXPORTS}
  DESTINATION .
  EXPORT SwiftExports
  COMPONENT dev)

include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/SwiftConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/SwiftConfig.cmake"
  INSTALL_DESTINATION ${SWIFT_INSTALL_PACKAGE_DIR}/cmake/swift)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/SwiftConfig.cmake
  DESTINATION ${SWIFT_INSTALL_PACKAGE_DIR})

install(EXPORT SwiftExports DESTINATION ${SWIFT_INSTALL_PACKAGE_DIR})

# installing each header individually to preserve the directory structure
macro(install_headers directory ext)
  file(GLOB_RECURSE headers RELATIVE ${directory} ${directory}/*.${ext})
  foreach(header ${headers})
    get_filename_component(dir ${header} DIRECTORY)
    install(FILES ${directory}/${header} DESTINATION ${SWIFT_INCLUDE_DIRS}/${dir})
  endforeach()
endmacro()
install_headers(${SWIFT_INCLUDE_DIR} h)
install_headers(${SWIFT_MAIN_INCLUDE_DIR} h)
install_headers(${SWIFT_INCLUDE_DIR} def)
install_headers(${SWIFT_MAIN_INCLUDE_DIR} def)

install(DIRECTORY ${CMAKE_SOURCE_DIR}/stdlib/public/SwiftShims DESTINATION stdlib/public)
