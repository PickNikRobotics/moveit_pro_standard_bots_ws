cmake_minimum_required(VERSION 3.22)
project(sbot_ro1_mock)

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY config description launch objectives waypoints
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
