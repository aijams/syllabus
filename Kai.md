Hi, it's me.

```
if (NOT TARGET gtensor::gtensor_@GTENSOR_DEVICE@)
  message(STATUS "include targets ${GTENSOR_BUILD_DEVICES}")
  include("${GTENSOR_CMAKE_DIR}/gtensor-targets.cmake")
  # Promote all targets to global so we can alias them below. See also
  # https://cmake.org/cmake/help/v3.13/command/add_library.html#alias-libraries
  foreach(DEVICE IN LISTS GTENSOR_BUILD_DEVICES)
    set_target_properties(gtensor::gtensor_${DEVICE}
                          PROPERTIES IMPORTED_GLOBAL TRUE)
  endforeach()
endif()
```
