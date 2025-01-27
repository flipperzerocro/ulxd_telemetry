# Define the application name
set(TARGET ulxd_telemetry)

# Add the application source file
add_executable(${TARGET}
    ulxd_telemetry.c
)

# Link the application with required libraries
target_link_libraries(${TARGET}
    furi
    furi_hal
    gui
    input
    subghz
)

# Specify the application install path within the firmware
install(TARGETS ${TARGET} RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/applications_user/${TARGET})
