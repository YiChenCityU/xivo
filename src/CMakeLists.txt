cmake_minimum_required(VERSION 3.5)

project(estimator)

# # overwrite binary output directory
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

# if set, use inverse-depth parametrization
# add_definitions(-DUSE_INVDEPTH)

# if set, perform online temporal calibration
add_definitions(-DUSE_ONLINE_IMU_CALIB)
add_definitions(-DUSE_ONLINE_TEMPORAL_CALIB)
add_definitions(-DUSE_ONLINE_CAMERA_CALIB)
# if set, approximate the initial correlation between feature state and
# group state with gradient information from depth refinement optimization
# add_definitions(-APPROXIMATE_INIT_CORRELATION)

include_directories(
  ${PROJECT_SOURCE_DIR}
  ${CMAKE_SOURCE_DIR}/common)

link_directories(
  ${CMAKE_SOURCE_DIR}/lib)

list(APPEND deps
        opencv_highgui
        opencv_features2d
        opencv_core
        opencv_video
        opencv_imgproc
        opencv_imgcodecs
        opencv_xfeatures2d
        glog
        pangolin
        pthread
        jsoncpp
        GL
        GLEW
        common
        )

add_library(xapp SHARED
        estimator_process.cpp
        loader.cpp
        geometry.cpp
        metrics.cpp
        publisher.cpp
        viewer.cpp)
target_link_libraries(xapp ${deps})

add_library(xest SHARED
        estimator.cpp
        princedormand.cpp
        rk4.cpp
        visualize.cpp
        tracker.cpp
        manager.cpp
        update.cpp
        graph.cpp
        feature.cpp
        oos.cpp
        group.cpp
        helpers.cpp
        options.cpp
        param.cpp
        mm.cpp
        camera_manager.cpp
        imu.cpp)
target_link_libraries(xest ${deps}) # profiler)

set(libxivo xest xapp)

if (BUILD_G2O)
  add_library(xopt
    optimizer.cpp
  )
  target_link_libraries(xopt xest ${deps})
  list(APPEND libxivo xopt)
endif(BUILD_G2O)

add_executable(vio app/vio.cpp)
target_link_libraries(vio ${libxivo} gflags::gflags)

# add_executable(double_vio app/double_vio.cpp)
# target_link_libraries(double_vio ${libxivo} gflags::gflags)

################################################################################
# TOOLING
################################################################################
# add_executable(eval app/evaluate.cpp)
# target_link_libraries(eval estimator application gflags::gflags)

################################################################################
# TESTS
################################################################################
if (BUILD_G2O)
  message(INFO ${libxivo})
  add_executable(test_optimizer test/test_optimizer.cpp)
  target_link_libraries(test_optimizer ${libxivo})
endif(BUILD_G2O)

# add_executable(test_givens test/test_givens.cpp )
# target_link_libraries(test_givens estimator)
#
# add_executable(test_qr test/test_qr.cpp )
# target_link_libraries(test_qr estimator)

# add_executable(test_estimator test/test_estimator.cpp )
# target_link_libraries(test_estimator estimator gtest gtest_main)
#
# add_executable(test_simulator test/test_simulator.cpp )
# target_link_libraries(test_simulator estimator gtest gtest_main)

# add_executable(test_tumvi test/test_tumvi.cpp )
# target_link_libraries(test_tumvi estimator gtest gtest_main)

# add_executable(test_viewer test/test_viewer.cpp)
# target_link_libraries(test_viewer estimator gtest gtest_main)

# add_executable(test_tracker test/test_tracker.cpp )
# target_link_libraries(test_tracker estimator gtest gtest_main)
#
# add_executable(test_matcher test/test_matcher.cpp )
# target_link_libraries(test_matcher estimator gtest gtest_main)
