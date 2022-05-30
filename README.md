# sh_android_dev_env

## Set up the Android development environment

```
sudo apt install default-jdk gradle curl python3-colcon-common-extensions python3-pip python3-vcstool
python3 -m pip install -U git+https://github.com/colcon/colcon-gradle
python3 -m pip install --no-deps git+https://github.com/colcon/colcon-ros-gradle
download the Android SDK:
- https://developer.android.com/studio/#downloads
- set the env variable ANDROID_HOME to the path where it's extracted
download the Android NDK:
- https://github.com/android/ndk/wiki/Unsupported-Downloads, version android-ndk-r18b-linux-x86_64.zip (see https://github.com/android/ndk/issues/1363)
- Add the following line after line 400 of android-ndk-r18b/build/cmake/android.toolchain.cmake
    list(APPEND ANDROID_STL_LDLIBS dl)
- set the env variable ANDROID_NDK to the path where it's extracted
```

## Build instructions for our select ROS2 repos

```
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

sudo apt update && sudo apt install -y curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt upgrade
sudo apt install ros-galactic-ros-base

mkdir -p /path/to/sh_android_dev_ws/src
cd /path/to/sh_android_dev_ws
rosinstall src /path/to/sh_android_dev_env/sh_android_dev.rosinstall
source /opt/ros/galactic/setup.bash
rosdep install --from-paths src -y -i --skip-keys "ament_tools"

export ANDROID_HOME="$HOME/android-studio"
export ANDROID_NDK="$HOME/android-ndk-r18b"
export PYTHON3_EXEC="$(which python3)"
export PYTHON3_LIBRARY="$(${PYTHON3_EXEC} -c 'import os.path; from distutils import sysconfig; print(os.path.realpath(os.path.join(sysconfig.get_config_var("LIBPL"), sysconfig.get_config_var("LDLIBRARY"))))')"
export PYTHON3_INCLUDE_DIR="$(${PYTHON3_EXEC} -c 'from distutils import sysconfig; print(sysconfig.get_config_var("INCLUDEPY"))')"
export ANDROID_ABI=armeabi-v7a
export ANDROID_NATIVE_API_LEVEL=android-21
export ANDROID_TOOLCHAIN_NAME=arm-linux-androideabi-clang

colcon build \
  --symlink-install \
  --packages-ignore cyclonedds rcl_logging_log4cxx rosidl_generator_py \
  --packages-up-to rcljava \
  --cmake-args \
  -DPYTHON_EXECUTABLE=${PYTHON3_EXEC} \
  -DPYTHON_LIBRARY=${PYTHON3_LIBRARY} \
  -DPYTHON_INCLUDE_DIR=${PYTHON3_INCLUDE_DIR} \
  -DCMAKE_TOOLCHAIN_FILE=${ANDROID_NDK}/build/cmake/android.toolchain.cmake \
  -DANDROID_FUNCTION_LEVEL_LINKING=OFF \
  -DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL} \
  -DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME} \
  -DANDROID_STL=c++_shared \
  -DANDROID_ABI=${ANDROID_ABI} \
  -DANDROID_NDK=${ANDROID_NDK} \
  -DTHIRDPARTY=ON \
  -DCOMPILE_EXAMPLES=OFF \
  -DCMAKE_FIND_ROOT_PATH="${PWD}/install"
```
