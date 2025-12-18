# ########################################
# ##         STAGE 1: BASE              ##
# ########################################
FROM ros:humble-ros-base-jammy AS base

# Setup shell and non-interactive mode
SHELL ["/bin/bash", "-o", "pipefail", "-o", "errexit", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

# Setup timezone
ENV TZ=Etc/UTC
RUN echo $TZ > /etc/timezone && \
    ln -fs /usr/share/zoneinfo/$TZ /etc/localtime

    # This stage now contains all the 'atop', 'cmake', 'libeigen', etc.
RUN --mount=type=bind,source=./scripts/install_deps.sh,target=/root/scripts/install_deps.sh \
    /root/scripts/install_deps.sh

RUN apt-get update && apt-get install -y locales && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG=en_US.UTF-8
# ########################################
# ##    STAGE 2: ROS 2 DEPENDENCIES     ##
# ########################################
FROM base AS dep-px4-ros2

ARG MICRO_XRCE_DDS_AGENT_VERSION=v2.4.2
ARG PX4_MSGS_VERSION=release/1.16
ARG PX4_ROS2_INTERFACE_LIB_VERSION=release/1.16
ARG PX4_ROS_COM_VERSION=release/1.16

RUN --mount=type=bind,source=./scripts/build_ros_deps.sh,target=/root/scripts/build_ros_deps.sh \
    /root/scripts/build_ros_deps.sh \
    ${MICRO_XRCE_DDS_AGENT_VERSION} \
    ${PX4_MSGS_VERSION} \
    ${PX4_ROS2_INTERFACE_LIB_VERSION} \
    ${PX4_ROS_COM_VERSION}

# ########################################
# ##         STAGE 3: BUILD PX4         ##
# ########################################
FROM dep-px4-ros2 AS build-px4

ARG PX4_VERSION=v1.16.0
ENV USER=px4 

RUN --mount=type=bind,source=./scripts/build_px4.sh,target=/home/${USER}/scripts/build_px4.sh \
    /home/${USER}/scripts/build_px4.sh ${PX4_VERSION}

########################################
##            INSTALL QGC             ##
########################################
FROM base AS qgc
ENV USER=px4 
ARG TARGETARCH
RUN echo "TARGETARCH=${TARGETARCH}"
ARG QGC_VERSION=v5.0.8
RUN --mount=type=bind,source=./scripts/install_qgc.sh,target=/home/${USER}/scripts/install_qgc.sh /home/${USER}/scripts/install_qgc.sh ${QGC_VERSION}
########################################
##           STAGE 4: FINAL           ##
########################################
FROM build-px4 AS final
# retrieve PX4-gazebo-models
ENV USER=px4 
COPY --from=build-px4 --chown=${USER}:${USER} /home/${USER}/PX4-Autopilot/Tools/simulation/gz /home/${USER}/PX4-gazebo-models
COPY --from=qgc --chown=${USER}:${USER} /home/${USER}/QGroundControl /home/${USER}/QGroundControl