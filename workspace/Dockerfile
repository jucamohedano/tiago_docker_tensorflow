# Preamble from extension [nvidia]
FROM nvidia/opengl:1.0-glvnd-devel-ubuntu18.04 as glvnd
FROM tensorflow/tensorflow:2.2.0-gpu as tensorflow
FROM osrf/ros:melodic-desktop-full

USER root

ARG TENSORFLOW_VERSION=v2.2.0
ARG CUDA_VER=10.1

RUN apt-get update && apt-get install -y --no-install-recommends \
    libglvnd0 \
    libgl1 \
    libglx0 \
    libegl1 \
    libgles2 \
    && rm -rf /var/lib/apt/lists/*
COPY --from=glvnd /usr/share/glvnd/egl_vendor.d/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics
# cuda and nvidia paths
ENV PATH=$PATH:/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/cuda-${CUDA_VER}/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-${CUDA_VER}/lib64

COPY --from=tensorflow /usr/local/cuda /usr/local/cuda/
COPY --from=tensorflow /usr/local/cuda-${CUDA_VER}/ /usr/local/cuda-${CUDA_VER}/
COPY --from=tensorflow /usr/local/lib/python3.6/dist-packages/tensorflow/ /usr/local/lib/python3.6/dist-packages/tensorflow/
COPY --from=tensorflow /usr/lib/x86_64-linux-gnu/ /usr/lib/x86_64-linux-gnu/


RUN apt-get update && apt-get install -y alsa-base pulseaudio

ARG TIAGO_USER
SHELL ["/bin/bash", "-c"]
RUN echo "${TIAGO_USER} ALL=(ALL) NOPASSWD:ALL" | EDITOR="tee -a" visudo

RUN apt-get update
RUN apt-get install wget vim openssh-server apt-utils -y

RUN sed -i "s/Port 22/Port 2222/" /etc/ssh/sshd_config
COPY entrypoint.sh /entrypoint.sh

COPY pythonr /usr/bin/pythonr
RUN chown root.root /usr/bin/pythonr
RUN chmod 777 /usr/bin/pythonr

RUN mkdir -p /opt/tiago_public_ws
WORKDIR /opt/tiago_public_ws
RUN wget https://raw.githubusercontent.com/pal-robotics/tiago_tutorials/kinetic-devel/tiago_public-melodic.rosinstall
RUN rosinstall src /opt/ros/melodic tiago_public-melodic.rosinstall

RUN rm -f /etc/ros/rosdep/sources.list.d/20-default.list
RUN rosdep init
RUN rosdep update
# ARG ROSDEP_IGNORE="opencv2 opencv2-nonfree pal_laser_filters speed_limit_node sensor_to_cloud hokuyo_node libdw-dev python-graphitesend-pip python-statsd pal_filters pal_vo_server pal_usb_utils pal_pcl pal_pcl_points_throttle_and_filter pal_karto pal_local_joint_control camera_calibration_files pal_startup_msgs pal-orbbec-openni2 dummy_actuators_manager pal_local_planner gravity_compensation_controller current_limit_controller dynamic_footprint dynamixel_cpp tf_lookup slam_toolbox joint_impedance_trajectory_controller cartesian_impedance_controller omni_base_description omni_drive_controller mynteye_description stockbot_description" -y
ARG ROSDEP_IGNORE="opencv2 opencv2-nonfree pal_laser_filters speed_limit_node sensor_to_cloud hokuyo_node libdw-dev python-graphitesend-pip python-statsd pal_filters pal_vo_server pal_usb_utils pal_pcl pal_pcl_points_throttle_and_filter pal_karto pal_local_joint_control camera_calibration_files pal_startup_msgs pal-orbbec-openni2 dummy_actuators_manager pal_local_planner gravity_compensation_controller current_limit_controller dynamic_footprint dynamixel_cpp tf_lookup slam_toolbox joint_impedance_trajectory_controller cartesian_impedance_controller omni_base_description omni_drive_controller mynteye_description stockbot_description" -y
RUN apt-get update && rosdep install --from-paths src --ignore-src -y --rosdistro melodic --skip-keys="${ROSDEP_IGNORE}"


# install python3.7
RUN apt-get install -y zip
ADD contact_graspnet_requirements.txt /
WORKDIR /
RUN apt-get install -y software-properties-common
RUN apt-get update
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update
RUN apt-get install -y python3.7
RUN apt-get install -y python3-pip python3-yaml
RUN pip3 install --upgrade pip
RUN pip3 install rospkg catkin_pkg
RUN pip3 install -r contact_graspnet_requirements.txt

WORKDIR /opt/tiago_public_ws
RUN apt-get update
RUN apt-get install ros-melodic-catkin python-catkin-tools python-pip -y
RUN source /opt/ros/melodic/setup.bash; catkin build -DCATKIN_ENABLE_TESTING=0
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64:/usr/local/cuda/lib64:/usr/local/nvidia/lib:/usr/local/nvidia/lib64

