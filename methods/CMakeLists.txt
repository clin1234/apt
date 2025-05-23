# Create the executable targets
include_directories($<$<BOOL:${SECCOMP_FOUND}>:${SECCOMP_INCLUDE_DIR}>)
link_libraries(apt-pkg $<$<BOOL:${SECCOMP_FOUND}>:${SECCOMP_LIBRARIES}>)

add_library(connectlib OBJECT connect.cc rfc2553emu.cc)

add_executable(file file.cc)
add_executable(copy copy.cc)
add_executable(store store.cc)
add_executable(gpgv gpgv.cc)
if (SQV_EXECUTABLE)
add_executable(sqv sqv.cc)
install(TARGETS sqv
        RUNTIME DESTINATION ${CMAKE_INSTALL_LIBEXECDIR}/apt/methods)
endif()
add_executable(cdrom cdrom.cc)
add_executable(http http.cc basehttp.cc $<TARGET_OBJECTS:connectlib>)
add_executable(mirror mirror.cc)
add_executable(rred rred.cc)

target_compile_definitions(connectlib PRIVATE ${OPENSSL_DEFINITIONS})
target_include_directories(connectlib PRIVATE ${OPENSSL_INCLUDE_DIR})
target_include_directories(http PRIVATE $<$<BOOL:${SYSTEMD_FOUND}>:${SYSTEMD_INCLUDE_DIRS}>)

# Additional libraries to link against for networked stuff
target_link_libraries(http OpenSSL::SSL $<$<BOOL:${SYSTEMD_FOUND}>:${SYSTEMD_LIBRARIES}>)

target_link_libraries(rred apt-private)

# Install the library
install(TARGETS file copy store cdrom gpgv http rred mirror
        RUNTIME DESTINATION ${CMAKE_INSTALL_LIBEXECDIR}/apt/methods)

add_links(${CMAKE_INSTALL_LIBEXECDIR}/apt/methods mirror mirror+http mirror+https mirror+file mirror+copy)


add_links(${CMAKE_INSTALL_LIBEXECDIR}/apt/methods http https)
