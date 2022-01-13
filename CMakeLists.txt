cmake_minimum_required(VERSION 3.20)
project(wsproject1)
set(CMAKE_CXX_STANDARD 17)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/lib)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/include)

function(cloneLibRepos repoUrl)
  set (git_cmd "git")
  set (git_arg1 "clone")
  execute_process(COMMAND ${git_cmd} ${git_arg1} ${repoUrl}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  )
endfunction()

function(triggerMake dirPath)
  set (cmd1 "make")
  set (arg1 "-C")
  execute_process(COMMAND ${cmd1} ${arg1} ${dirPath}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  )
endfunction()

function(triggerMakeInstall argrepo)
  set (cmd2 "sudo")
  set (arg3 "-S")
  set (arg4 "make")
  set (arg5 "install")
  set (arg6 "-C")
  execute_process(COMMAND ${cmd2} ${arg3} ${arg4} ${arg5} ${arg6} ${argrepo}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  )
endfunction()

#clone github libs
cloneLibRepos("https://github.com/ebiggers/libdeflate.git")
cloneLibRepos("https://github.com/uNetworking/uSockets.git")
cloneLibRepos("https://github.com/uNetworking/uWebSockets.git")

file(COPY ${CMAKE_CURRENT_BINARY_DIR}/libdeflate/
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/libdeflate)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/uSockets/
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/uSockets)

#triggering make command
triggerMake(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/libdeflate)
triggerMake(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/uSockets)
triggerMake(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets)

#triggering make install command
triggerMakeInstall(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/libdeflate)
triggerMakeInstall(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/uSockets)
triggerMakeInstall(${CMAKE_CURRENT_BINARY_DIR}/uWebSockets)

#copying files and directories to 3rdparty libs location
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/uSockets/uSockets.a
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/lib)
file(RENAME ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/lib/uSockets.a ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/lib/libuSockets.a)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/src/
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/include)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/uWebSockets/uSockets/src/libusockets.h
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/include)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/include ${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/include/uSockets/)
link_directories(${CMAKE_CURRENT_BINARY_DIR}/nextgen/3rdparty/lib)
link_directories(/usr/local/ssl)

add_executable(webservice_sample1 pradeepkws1.cpp)
target_link_libraries(webservice_sample1 uSockets pthread z ssl)