FROM ubuntu:focal
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    expect \
    locales \
    nano \
    # Sticking to jdk8 for the moment
    # Problems with Kapt on JDK>8, https://youtrack.jetbrains.com/issue/KT-28587
    # Also 8 is whats included with Android Studio
    openjdk-8-jdk \
    unzip \
    curl \
  && rm -rf /var/lib/apt/lists/*

# Seems somethings build better with utf8 locale specified
# http://jaredmarkell.com/docker-and-locales/
# https://github.com/square/moshi/issues/804#issuecomment-466926878
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

##<node>##
# hadolint ignore=DL3008
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    apt-transport-https \
    gnupg \
    lsb-release \
  # For nodejs we use nodesource, its nice and easy and gets us the correct version
  # Find latest link https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions
  && curl -sSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && echo "deb https://deb.nodesource.com/node_12.x $(lsb_release -s -c) main" | tee /etc/apt/sources.list.d/nodesource.list \
  && echo "deb-src https://deb.nodesource.com/node_12.x $(lsb_release -s -c) main" | tee -a /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    nodejs \
  && rm -rf /var/lib/apt/lists/*

# hadolint ignore=DL3016
RUN npm -g install xcode-build-tools
##</node>##

# Install the SDK
# https://developer.android.com/studio#downloads
ENV ANDROID_CMDLINE_TOOLS https://dl.google.com/android/repository/commandlinetools-linux-6609375_latest.zip
# hadolint ignore=DL3003
RUN ( \
    cd /opt \
    && mkdir android-sdk-linux \
    && curl -sSL -o cmdline-tools.zip "$ANDROID_CMDLINE_TOOLS" \
    && unzip cmdline-tools.zip -d android-sdk-linux/cmdline-tools \
    && rm -f cmdline-tools.zip \
    && chown -R root:root android-sdk-linux \
  )

ENV ANDROID_SDK_ROOT /opt/android-sdk-linux
ENV ANDROID_HOME $ANDROID_SDK_ROOT
ENV PATH $ANDROID_HOME/cmdline-tools/tools/bin:$ANDROID_SDK_ROOT/tools/bin:$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/platform-tools:$PATH

# Libs required for sdkmanager to run using openjdk-12
# https://stackoverflow.com/a/55982976/859027
# Removed as currently back on jdk 8
#RUN mkdir /opt/jaxb_lib \
#  && wget http://central.maven.org/maven2/javax/activation/activation/1.1.1/activation-1.1.1.jar -O /opt/jaxb_lib/activation.jar \
#  && wget http://central.maven.org/maven2/javax/xml/jaxb-impl/2.1/jaxb-impl-2.1.jar -O /opt/jaxb_lib/jaxb-impl.jar \
#  && wget http://central.maven.org/maven2/org/glassfish/jaxb/jaxb-xjc/2.3.2/jaxb-xjc-2.3.2.jar -O /opt/jaxb_lib/jaxb-xjc.jar \
#  && wget http://central.maven.org/maven2/org/glassfish/jaxb/jaxb-core/2.3.0.1/jaxb-core-2.3.0.1.jar -O /opt/jaxb_lib/jaxb-core.jar \
#  && wget http://central.maven.org/maven2/org/glassfish/jaxb/jaxb-jxc/2.3.2/jaxb-jxc-2.3.2.jar -O /opt/jaxb_lib/jaxb-jxc.jar \
#  && wget http://central.maven.org/maven2/javax/xml/bind/jaxb-api/2.3.1/jaxb-api-2.3.1.jar -O /opt/jaxb_lib/jaxb-api.jar \
#  && sed -i '/^CLASSPATH=/a CLASSPATH=/opt/jaxb_lib/activation.jar:/opt/jaxb_lib/jaxb-impl.jar:/opt/jaxb_lib/jaxb-xjc.jar:/opt/jaxb_lib/jaxb-core.jar:/opt/jaxb_lib/jaxb-jxc.jar:/opt/jaxb_lib/jaxb-api.jar:$CLASSPATH' $ANDROID_SDK_ROOT/tools/bin/sdkmanager \
#  && sed -i '/^CLASSPATH=/a CLASSPATH=/opt/jaxb_lib/activation.jar:/opt/jaxb_lib/jaxb-impl.jar:/opt/jaxb_lib/jaxb-xjc.jar:/opt/jaxb_lib/jaxb-core.jar:/opt/jaxb_lib/jaxb-jxc.jar:/opt/jaxb_lib/jaxb-api.jar:$CLASSPATH' $ANDROID_SDK_ROOT/tools/bin/avdmanager

# Install custom tools
COPY tools/license_accepter /opt/tools/
COPY tools/adb-all /opt/tools
ENV PATH /opt/tools:$PATH
RUN license_accepter

# Install Android platform and things
ENV ANDROID_PLATFORM_VERSION 29
ENV ANDROID_BUILD_TOOLS_VERSION 29.0.3
ENV PATH $ANDROID_SDK_ROOT/build-tools/$ANDROID_BUILD_TOOLS_VERSION:$PATH
ENV ANDROID_EXTRA_PACKAGES "build-tools;29.0.0" "build-tools;29.0.1" "build-tools;29.0.2"
ENV ANDROID_REPOSITORIES "extras;android;m2repository" "extras;google;m2repository"
ENV ANDROID_CONSTRAINT_PACKAGES "extras;m2repository;com;android;support;constraint;constraint-layout;1.0.2" "extras;m2repository;com;android;support;constraint;constraint-layout;1.0.1" "extras;m2repository;com;android;support;constraint;constraint-layout;1.0.0"
RUN sdkmanager --verbose "platform-tools" "platforms;android-$ANDROID_PLATFORM_VERSION" "build-tools;$ANDROID_BUILD_TOOLS_VERSION" $ANDROID_EXTRA_PACKAGES $ANDROID_REPOSITORIES $ANDROID_CONSTRAINT_PACKAGES


##<emulator>##
# hadolint ignore=DL3008
RUN apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    file \
    qt5-default \
    libpulse0 \
  && rm -rf /var/lib/apt/lists/*

ENV ANDROID_EMULATOR_PACKAGE "system-images;android-$ANDROID_PLATFORM_VERSION;google_apis_playstore;x86_64"
RUN sdkmanager --verbose "emulator" $ANDROID_EMULATOR_PACKAGE

# Fix for emulator detect 64bit
ENV SHELL /bin/bash
# https://www.bram.us/2017/05/12/launching-the-android-emulator-from-the-command-line/
ENV PATH $ANDROID_SDK_ROOT/emulator:$PATH

COPY tools-emulator/android-avdmanager-create /opt/tools/
COPY tools-emulator/android-start-emulator /opt/tools/
COPY tools-emulator/android-wait-for-emulator /opt/tools/
##</emulator>##

