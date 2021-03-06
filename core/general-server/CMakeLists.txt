include_directories(SYSTEM  ${CMAKE_CURRENT_LIST_DIR}/../../)
include(op/CMakeLists.txt)
include(proto/CMakeLists.txt)
add_executable(serving ${serving_srcs})
add_dependencies(serving pdcodegen fluid_cpu_engine pdserving paddle_fluid cube-api utils)
if (WITH_GPU)
    add_dependencies(serving fluid_gpu_engine)
endif()
target_include_directories(serving PUBLIC
        ${CMAKE_CURRENT_BINARY_DIR}/../../core/predictor
        )

if(WITH_GPU)
    target_link_libraries(serving -Wl,--whole-archive fluid_gpu_engine
            -Wl,--no-whole-archive)
endif()

target_link_libraries(serving -Wl,--whole-archive fluid_cpu_engine
        -Wl,--no-whole-archive)

target_link_libraries(serving paddle_fluid ${paddle_depend_libs})

target_link_libraries(serving pdserving)
target_link_libraries(serving cube-api)
target_link_libraries(serving utils)

if(WITH_GPU)
    target_link_libraries(serving ${CUDA_LIBRARIES})
endif()

if(WITH_MKL OR WITH_GPU)
    target_link_libraries(serving -liomp5 -lmklml_intel -lmkldnn -lpthread -lcrypto -lm -lrt -lssl -ldl -lz -lbz2)
else()
    target_link_libraries(serving openblas -lpthread -lcrypto -lm -lrt -lssl -ldl -lz -lbz2)
endif()

install(TARGETS serving
        RUNTIME DESTINATION
        ${PADDLE_SERVING_INSTALL_DIR}/demo/serving/bin)

FILE(GLOB inc ${CMAKE_CURRENT_BINARY_DIR}/*.pb.h)
install(FILES ${inc}
        DESTINATION ${PADDLE_SERVING_INSTALL_DIR}/include/serving)

if (${WITH_MKL})
install(FILES
        ${CMAKE_BINARY_DIR}/third_party/install/Paddle/third_party/install/mklml/lib/libmklml_intel.so
        ${CMAKE_BINARY_DIR}/third_party/install/Paddle/third_party/install/mklml/lib/libiomp5.so
        ${CMAKE_BINARY_DIR}/third_party/install/Paddle/third_party/install/mkldnn/lib/libmkldnn.so.0
        DESTINATION
        ${PADDLE_SERVING_INSTALL_DIR}/demo/serving/bin)
endif()
