# syntax = docker/dockerfile:1.4
# Copyright (c) Contributors to the aswf-docker Project. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# "Global" ARGs
ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_CLANG_MAJOR_VERSION
ARG ASWF_VFXPLATFORM_VERSION

ARG ASWF_BOOST_VERSION
ARG ASWF_CMAKE_VERSION
ARG ASWF_CPPUNIT_VERSION
ARG ASWF_GLEW_VERSION
ARG ASWF_GLFW_VERSION
ARG ASWF_PYTHON_VERSION
ARG ASWF_PYTHON_MAJOR_MINOR_VERSION
ARG ASWF_TBB_VERSION
ARG ASWF_PYBIND11_VERSION

ARG ASWF_ALEMBIC_VERSION
ARG ASWF_BLOSC_VERSION
ARG ASWF_HDF5_VERSION
ARG ASWF_OCIO_CONFIGS_VERSION
ARG ASWF_OCIO_VERSION
ARG ASWF_OIIO_VERSION
ARG ASWF_OPENEXR_VERSION

# Required base packages built in previous stages
FROM ${ASWF_ORG}/ci-package-pybind11:${ASWF_VFXPLATFORM_VERSION}-${ASWF_PYBIND11_VERSION} as ci-package-pybind11-external


#################### ci-base-builder ####################
FROM ${ASWF_ORG}/ci-common:${CI_COMMON_VERSION}-clang${ASWF_CLANG_MAJOR_VERSION} AS ci-base-builder

ARG ASWF_CMAKE_VERSION
ARG ASWF_PYTHON_VERSION
ARG ASWF_NUMPY_VERSION
ARG ASWF_BOOST_VERSION
ARG ASWF_TBB_VERSION
ARG ASWF_PKG_ORG
ARG ASWF_CONAN_CHANNEL
ARG ASWF_VFXPLATFORM_VERSION

ENV ASWF_CMAKE_VERSION=${ASWF_CMAKE_VERSION}
ENV ASWF_PYTHON_VERSION=${ASWF_PYTHON_VERSION}
ENV ASWF_NUMPY_VERSION=${ASWF_NUMPY_VERSION}
ENV ASWF_BOOST_VERSION=${ASWF_BOOST_VERSION}
ENV ASWF_TBB_VERSION=${ASWF_TBB_VERSION}
ENV ASWF_VFXPLATFORM_VERSION=${ASWF_VFXPLATFORM_VERSION}

COPY scripts/common/install_conanpackages.sh /tmp

# Use conan to install some packages
COPY <<EOF /usr/local/conanfile.txt
[generators]
[imports]
., * -> .
[requires]
cmake/${ASWF_CMAKE_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
python/${ASWF_PYTHON_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
boost/${ASWF_BOOST_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
tbb/${ASWF_TBB_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
EOF

RUN --mount=type=cache,target=/opt/conan_home/d \
    --mount=type=bind,rw,target=/opt/conan_home/.conan,source=packages/conan/settings \
    /tmp/install_conanpackages.sh /usr/local vfx${ASWF_VFXPLATFORM_VERSION}

COPY ../scripts/common/before_build.sh \
     ../scripts/common/copy_new_files.sh \
     /tmp/

ENV DOWNLOADS_DIR=/tmp/downloads \
    CCACHE_DIR=/tmp/ccache \
    ASWF_INSTALL_PREFIX=/usr/local \
    PYTHONPATH=${ASWF_INSTALL_PREFIX}/lib/python${ASWF_PYTHON_MAJOR_MINOR_VERSION}/site-packages:${PYTHONPATH}


#################### ci-imath-builder ####################
FROM ci-base-builder AS ci-imath-builder

ARG ASWF_IMATH_VERSION
ENV ASWF_IMATH_VERSION=${ASWF_IMATH_VERSION}

COPY ../scripts/vfx/build_imath.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_imath.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-imath ####################
FROM scratch AS ci-package-imath

ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_IMATH_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-imath"
LABEL org.opencontainers.image.title="Imath package built for ASWF Docker images"
LABEL org.opencontainers.image.description="Imath headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/AcademySoftwareFoundation/imath"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_IMATH_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.imath="${ASWF_IMATH_VERSION}"

COPY --from=ci-imath-builder /package/. /


#################### ci-openexr-builder ####################
FROM ci-imath-builder AS ci-openexr-builder

ARG ASWF_OPENEXR_VERSION
ENV ASWF_OPENEXR_VERSION=${ASWF_OPENEXR_VERSION}

COPY ../scripts/vfx/build_openexr.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_openexr.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-openexr ####################
FROM scratch AS ci-package-openexr

ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_OPENEXR_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-openexr"
LABEL org.opencontainers.image.title="OpenEXR package built for ASWF Docker images"
LABEL org.opencontainers.image.description="OpenEXR headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/AcademySoftwareFoundation/openexr"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_OPENEXR_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.openexr="${ASWF_OPENEXR_VERSION}"

COPY --from=ci-openexr-builder /package/. /



#################### ci-alembic-builder ####################
FROM ci-openexr-builder AS ci-alembic-builder

ARG ASWF_ALEMBIC_VERSION
ENV ASWF_ALEMBIC_VERSION=${ASWF_ALEMBIC_VERSION}
ARG ASWF_HDF5_VERSION
ENV ASWF_HDF5_VERSION=${ASWF_HDF5_VERSION}

COPY ../scripts/vfx/build_alembic.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_alembic.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-alembic ####################
FROM scratch AS ci-package-alembic

ARG ASWF_ALEMBIC_VERSION
ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_HDF5_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-alembic"
LABEL org.opencontainers.image.title="Alembic package built for ASWF Docker images"
LABEL org.opencontainers.image.description="Alembic headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/alembic/alembic"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_ALEMBIC_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.alembic="${ASWF_ALEMBIC_VERSION}"
LABEL io.aswf.docker.versions.hdf5="${ASWF_HDF5_VERSION}"

COPY --from=ci-alembic-builder /package/. /


#################### ci-blosc-builder ####################
FROM ci-base-builder AS ci-blosc-builder

ARG ASWF_BLOSC_VERSION
ENV ASWF_BLOSC_VERSION=${ASWF_BLOSC_VERSION}

COPY ../scripts/vfx/build_blosc.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_blosc.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-blosc ####################
FROM scratch AS ci-package-blosc

ARG ASWF_ORG
ARG ASWF_BLOSC_VERSION
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-blosc"
LABEL org.opencontainers.image.title="Blosc package built for ASWF Docker images"
LABEL org.opencontainers.image.description="Blosc headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/Blosc/c-blosc"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_BLOSC_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.blosc="${ASWF_BLOSC_VERSION}"

COPY --from=ci-blosc-builder /package/. /


#################### ci-openvdb-builder ####################
FROM ci-openexr-builder AS ci-openvdb-builder

ARG ASWF_OPENVDB_VERSION
ARG ASWF_VFXPLATFORM_VERSION
ARG ASWF_CPPUNIT_VERSION
ARG ASWF_GLEW_VERSION
ARG ASWF_GLFW_VERSION

ENV ASWF_OPENVDB_VERSION=${ASWF_OPENVDB_VERSION}
ENV ASWF_VFXPLATFORM_VERSION=${ASWF_VFXPLATFORM_VERSION}
ENV ASWF_CPPUNIT_VERSION=${ASWF_CPPUNIT_VERSION}
ENV ASWF_GLEW_VERSION=${ASWF_GLEW_VERSION}
ENV ASWF_GLFW_VERSION=${ASWF_GLFW_VERSION}

COPY ../scripts/vfx/build_openvdb.sh \
     ../scripts/vfx/openvdb-imath.patch \
     /tmp/

COPY --from=ci-package-blosc /. /usr/local/

COPY scripts/common/install_conanpackages.sh /tmp

# Use conan to install some packages
COPY <<EOF /usr/local/conanfile.txt
[generators]
[imports]
., * -> .
[requires]
cppunit/${ASWF_CPPUNIT_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
glew/${ASWF_GLEW_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
glfw/${ASWF_GLFW_VERSION}@${ASWF_PKG_ORG}/${ASWF_CONAN_CHANNEL}
EOF

RUN --mount=type=cache,target=/opt/conan_home/d \
    --mount=type=bind,rw,target=/opt/conan_home/.conan,source=packages/conan/settings \
    /tmp/install_conanpackages.sh /usr/local vfx${ASWF_VFXPLATFORM_VERSION}



RUN --mount=type=cache,target=/tmp/ccache \
     --mount=type=cache,sharing=private,target=/tmp/downloads \
     /tmp/before_build.sh && \
     /tmp/build_openvdb.sh && \
     /tmp/copy_new_files.sh && \
     ccache --show-stats


#################### ci-package-openvdb ####################
FROM scratch AS ci-package-openvdb

ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_OPENVDB_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-openvdb"
LABEL org.opencontainers.image.title="OpenVDB package built for ASWF Docker images"
LABEL org.opencontainers.image.description="OpenVDB headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/AcademySoftwareFoundation/openvdb"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_OPENVDB_VERSION}"
LABEL org.opencontainers.image.licenses="MPL-2.0"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.openvdb="${ASWF_OPENVDB_VERSION}"

COPY --from=ci-openvdb-builder /package/. /


#################### ci-oiio-builder ####################
FROM ci-openvdb-builder AS ci-oiio-builder

ARG ASWF_OIIO_VERSION
ENV ASWF_OIIO_VERSION=${ASWF_OIIO_VERSION}

COPY --from=ci-package-pybind11-external /. /usr/local/

COPY ../scripts/vfx/build_oiio.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_oiio.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-oiio ####################
FROM scratch AS ci-package-oiio

ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_OIIO_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-oiio"
LABEL org.opencontainers.image.title="OpenImageIO package built for ASWF Docker images"
LABEL org.opencontainers.image.description="OpenImageIO headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/OpenImageIO/oiio"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_OIIO_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.oiio="${ASWF_OIIO_VERSION}"

COPY --from=ci-oiio-builder /package/. /


#################### ci-ocio-builder ####################
FROM ci-oiio-builder AS ci-ocio-builder

ARG ASWF_OCIO_VERSION
ENV ASWF_OCIO_VERSION=${ASWF_OCIO_VERSION}
ARG ASWF_OCIO_CONFIGS_VERSION
ENV ASWF_OCIO_CONFIGS_VERSION=${ASWF_OCIO_CONFIGS_VERSION}

COPY ../scripts/vfx/build_ocio.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_ocio.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats


#################### ci-package-ocio ####################
FROM scratch AS ci-package-ocio

ARG ASWF_ORG
ARG CI_COMMON_VERSION
ARG ASWF_DTS_VERSION
ARG ASWF_DTS_PREFIX
ARG ASWF_OCIO_CONFIGS_VERSION
ARG ASWF_OCIO_VERSION
ARG ASWF_VFXPLATFORM_VERSION

LABEL org.opencontainers.image.name="$ASWF_ORG/ci-package-ocio"
LABEL org.opencontainers.image.title="OpenColorIO package built for ASWF Docker images"
LABEL org.opencontainers.image.description="OpenColorIO headers and binaries to be installed in ASWF Docker images"
LABEL org.opencontainers.image.authors="Built by aswf.io CI Working Group"
LABEL org.opencontainers.image.vendor="AcademySoftwareFoundation"
LABEL org.opencontainers.image.url="https://github.com/AcademySoftwareFoundation/OpenColorIO"
LABEL org.opencontainers.image.source="https://github.com/AcademySoftwareFoundation/aswf-docker/blob/master/packages/Dockerfile"
LABEL org.opencontainers.image.version="${ASWF_OCIO_VERSION}"
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL io.aswf.docker.versions.ci-common="${CI_COMMON_VERSION}"
LABEL io.aswf.docker.versions.vfx-platform="${ASWF_VFXPLATFORM_VERSION}"
LABEL io.aswf.docker.versions.dts="${ASWF_DTS_VERSION}"
LABEL io.aswf.docker.versions.ocio="${ASWF_OCIO_VERSION}"
LABEL io.aswf.docker.versions.ocioconfigs="${ASWF_OCIO_CONFIGS_VERSION}"

COPY --from=ci-ocio-builder /package/. /


#################### ci-osl-builder ####################
FROM ci-oiio-builder AS ci-osl-builder

ARG ASWF_OSL_VERSION
ENV ASWF_OSL_VERSION=${ASWF_OSL_VERSION}

COPY --from=ci-package-clang-external /. /usr/local/
COPY --from=ci-package-qt-external /. /usr/local/
COPY --from=ci-package-partio /. /usr/local/

COPY ../scripts/vfx/build_osl.sh \
     /tmp/

RUN --mount=type=cache,target=/tmp/ccache \
    --mount=type=cache,sharing=private,target=/tmp/downloads \
    /tmp/before_build.sh && \
    /tmp/build_osl.sh && \
    /tmp/copy_new_files.sh && \
    ccache --show-stats

