cmake_minimum_required(VERSION 3.0.0)

project(fft2d VERSION 1.0.0)

# Tensorflow sets PIC, but we can make sure this is set in the toolchain:
#
# set(CMAKE_POSITION_INDEPENDENT_CODE ON)

#     fft4g.c    : FFT Package in C       - Fast Version   I   (radix 4,2)
#     fft4g.f    : FFT Package in Fortran - Fast Version   I   (radix 4,2)
#     fft4g_h.c  : FFT Package in C       - Simple Version I   (radix 4,2)
#     fft8g.c    : FFT Package in C       - Fast Version   II  (radix 8,4,2)
#     fft8g.f    : FFT Package in Fortran - Fast Version   II  (radix 8,4,2)
#     fft8g_h.c  : FFT Package in C       - Simple Version II  (radix 8,4,2)
#     fftsg.c    : FFT Package in C       - Fast Version   III (Split-Radix)
#     fftsg.f    : FFT Package in Fortran - Fast Version   III (Split-Radix)
#     fftsg_h.c  : FFT Package in C       - Simple Version III (Split-Radix)

# Here we use only the fftsg.c version for "fast split-radix" processing.
# This is the only file that is used by tensorflow, for example.
#
# TODO:
#
# As a future extension, this could be modified to provide each
# implementation as a separate module in the package:
#
# target_link_libraries(app fft2d::fftsg)
# target_link_libraries(app fft2d::fft4g)
# target_link_libraries(app fft2d::fft4g_h)
# target_link_libraries(app fft2d::fft8g)
# target_link_libraries(app fft2d::fft8g_h)

set(FFT2D_SRCS fftsg.c) 

add_library(fft2d ${FFT2D_SRCS})

option(FFT2D_BUILD_EXAMPLES "Build examples" OFF)
if(FFT2D_BUILD_EXAMPLES)
  add_subdirectory(sample1)
  add_subdirectory(sample2)
endif()

### Install ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
  )

# Note: use 'targets_export_name'
configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
  )

install(
  TARGETS ${PROJECT_NAME}
  EXPORT "${targets_export_name}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "${include_install_dir}"
  )

# The original package has no exported header files

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )


