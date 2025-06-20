cmake_minimum_required(VERSION 3.8)
project(neural_network_detector)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(OpenCV REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(neural_network_msgs REQUIRED)
find_package(ros2_utils REQUIRED)


include_directories(include)
include_directories(third_party/YOLOs-CPP/include)  # Add YOLO include path
include_directories(third_party/YOLOs-CPP/onnxruntime-linux-x64-1.20.1/include)  # Add ONNX include path

# Define ONNX Runtime paths
set(ONNXRUNTIME_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/third_party/YOLOs-CPP/onnxruntime-linux-x64-1.20.1")
set(ONNXRUNTIME_LIB "${ONNXRUNTIME_ROOT}/lib/libonnxruntime.so.1.20.1")

# Verify the library exists
if(NOT EXISTS "${ONNXRUNTIME_LIB}")
    message(FATAL_ERROR "ONNX Runtime library not found at: ${ONNXRUNTIME_LIB}")
endif()

add_executable(yolo12_detector_node 
src/yolo12_detector_node.cpp
)

ament_target_dependencies(yolo12_detector_node
  rclcpp
  std_msgs
  sensor_msgs
  geometry_msgs
  cv_bridge
  image_transport
  neural_network_msgs
  OpenCV
  rclcpp_components
  ros2_utils
)

ament_export_include_directories(include)
ament_export_libraries(yolo12_detector_node_component)
ament_export_dependencies(
  rclcpp
  rclcpp_components
  geometry_msgs
  sensor_msgs
  std_msgs
  neural_network_msgs
  tf2_ros
  tf2
  tf2_geometry_msgs
  cv_bridge
  nav_msgs
  OpenCV
  ros2_utils
)

# Link ONNX Runtime libraries to your executable
target_link_libraries(yolo12_detector_node 
    "${ONNXRUNTIME_LIB}"
)

# Set RPATH so the executable can find the library at runtime
set_target_properties(yolo12_detector_node PROPERTIES
    INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib/${PROJECT_NAME}/lib"
    BUILD_WITH_INSTALL_RPATH FALSE
    INSTALL_RPATH_USE_LINK_PATH TRUE
)

target_link_libraries(yolo12_detector_node "${cpp_typesupport_target}")


if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

# Install the executable
install(TARGETS
  yolo12_detector_node
  DESTINATION lib/${PROJECT_NAME}
)

# Install ONNX Runtime library
install(FILES
  "${ONNXRUNTIME_LIB}"
  DESTINATION lib/${PROJECT_NAME}/lib
  RENAME libonnxruntime.so.1
)

# Install YOLO models and labels
install(DIRECTORY
  third_party/
  DESTINATION share/${PROJECT_NAME}/
  FILES_MATCHING 
  PATTERN "*.onnx"
  PATTERN "*.names"
  PATTERN "*.txt"
)

install(DIRECTORY include/
  DESTINATION include
)

ament_package()