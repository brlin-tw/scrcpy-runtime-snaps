# The unofficial scrcpy runtime snaps

Provide easier integration of the built scrcpy runtime into other snaps.

<https://gitlab.com/brlin/scrcpy-runtime-snaps>  
[![The GitLab CI pipeline status badge of the project's `main` branch](https://gitlab.com/brlin/scrcpy-runtime-snaps/badges/main/pipeline.svg?ignore_skipped=true "Click here to check out the comprehensive status of the GitLab CI pipelines")](https://gitlab.com/brlin/scrcpy-runtime-snaps/-/pipelines) [![GitHub Actions workflow status badge](https://github.com/brlin/scrcpy-runtime-snaps/actions/workflows/check-potential-problems.yml/badge.svg "GitHub Actions workflow status")](https://github.com/brlin/scrcpy-runtime-snaps/actions/workflows/check-potential-problems.yml) [![pre-commit enabled badge](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white "This project uses pre-commit to check potential problems")](https://pre-commit.com/) [![REUSE Specification compliance badge](https://api.reuse.software/badge/gitlab.com/brlin/scrcpy-runtime-snaps "This project complies to the REUSE specification to decrease software licensing costs")](https://api.reuse.software/info/gitlab.com/brlin/scrcpy-runtime-snaps)

\#scrcpy \#snap-packaging \#content-snap

## Prerequisites

The following are the prerequisites for using this runtime snap:

* Your snap must be built for the `core24` base, as this runtime snap is built for the `core24` base.  For snaps using different bases, please follow the corresponding documentation in the corresponding Git branches of this repository (e.g. `core22` branch for `core22`-based snaps, if available).

Currently only snap using the `gnome` extension is tested, for other extensions this content snap may or may not work.

## Usage

In your consumer snap's Snapcraft project file:

1. Add the following plug definition:

    ```yaml
    plugs:
      scrcpy-runtime-2404:
        interface: content
        target: scrcpy-runtime
        default-provider: scrcpy-runtime-2404
    ```

1. Ensure that you have a part that provides the Android Debug Bridge (ADB), this one should be suffice for most use cases:

    ```yaml
    parts:
      adb:
        plugin: nil
        stage-packages:
          - adb
    ```

1. In the app definition that needs to use the scrcpy runtime, _merge_ the following content:

    ```yaml
    apps:
      app-name:
        environment:
          LD_LIBRARY_PATH: ${SNAP}/scrcpy-runtime/usr/lib/${CRAFT_ARCH_TRIPLET_BUILD_FOR}:${SNAP}/usr/lib/${CRAFT_ARCH_TRIPLET_BUILD_FOR}/android:${SNAP}/gnome-platform/usr/lib/${CRAFT_ARCH_TRIPLET_BUILD_FOR}:${LD_LIBRARY_PATH}
          PATH: ${SNAP}/scrcpy-runtime/usr/local/bin:${PATH}
        plugs:
          # For ADB and ADB daemon communication
          - adb-support
          - network-bind
          - raw-usb
    ```

1. Include the following auxillary app definition to let the user to kill the snap-provided ADB daemon easily and for troubleshooting purposes:

    ```yaml
    apps:
      adb:
        command: usr/bin/adb
        environment:
          # NOTE: The gnome-platform content snap entry provides libprotobuf
          # required by the adb binary.
          LD_LIBRARY_PATH: ${SNAP}/usr/lib/${CRAFT_ARCH_TRIPLET_BUILD_FOR}/android:${SNAP}/gnome-platform/usr/lib/${CRAFT_ARCH_TRIPLET_BUILD_FOR}:${LD_LIBRARY_PATH}
        plugs:
          - network-bind
          - raw-usb
    ```

## References

The following resources are referenced during the development of this product:

* [adb-support interface - Snap documentation](https://snapcraft.io/docs/reference/interfaces/adb-support-interface/index.html)  
  [snapd/interfaces/builtin/adb_support.go at 2576363 · canonical/snapd](https://github.com/canonical/snapd/blob/2576363/interfaces/builtin/adb_support.go)  
  For the actual implementation of the adb-support interface, which is used to allow the snap-provided ADB daemon to access the Android devices.
* [snapcrafters/ffmpeg-2404-sdk: Content snap for ffmpeg](https://github.com/snapcrafters/ffmpeg-2404-sdk)  
  For reference on how to create a content snap and how to set up the environment variables for the consumer snaps.

## Licensing

Unless otherwise noted([comment headers](https://reuse.software/spec-3.3/#comment-headers)/[REUSE.toml](https://reuse.software/spec-3.3/#reusetoml)), this product is licensed under [the 2.0 version of the Apache License](https://www.apache.org/licenses/LICENSE-2.0), or any of its more recent versions of your preference.

This work complies to [the REUSE Specification](https://reuse.software/spec/), refer to the [REUSE - Make licensing easy for everyone](https://reuse.software/) website for info regarding the licensing of this product.
