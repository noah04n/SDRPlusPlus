# Building SDR++ on macOS

This guide will walk you through building SDR++ from source with the Audio Source enabled.

## 1. Install Dependencies

Install the core dependencies and libraries. In my case, I only have an **RTL-SDR**, so I opted to ignore all other hardware dependencies.

```sh
brew install cmake fmt pkg-config libusb fftw glfw portaudio rtl-sdr rtaudio codec2 zstd
pip3 install mako --break-system-packages
```
Optionally, if you are using a different hardware source, you can install the following to enable other hardware sources:

```sh
brew install airspy airspyhf hackrf libbladerf
```

## 2. Install Volk

Next, the vector optimization library has to be built from source.

```sh
git clone --recursive https://github.com/gnuradio/volk
cd volk
mkdir build && cd build
cmake -DCMAKE_OSX_DEPLOYMENT_TARGET=10.15 -DCMAKE_BUILD_TYPE=Release ..
make -j4
sudo make install
cd ../..
```

## 3. Configure and Build SDR++

### Option 1: Minimal / RTL-SDR Only

```sh
# Create build directory
mkdir build
cd build

# Run CMake with specific hardware simplifications
cmake .. \
    -DOPT_BUILD_RTL_SDR_SOURCE=ON \
    -DOPT_BUILD_AUDIO_SOURCE=ON \
    -DOPT_BUILD_PORTAUDIO_SINK=ON \
    -DOPT_BUILD_NEW_PORTAUDIO_SINK=ON \
    -DOPT_BUILD_RTL_TCP_SOURCE=ON \
    -DOPT_BUILD_SOAPY_SOURCE=OFF \
    -DOPT_BUILD_AIRSPY_SOURCE=OFF \
    -DOPT_BUILD_AIRSPYHF_SOURCE=OFF \
    -DOPT_BUILD_BLADERF_SOURCE=OFF \
    -DOPT_BUILD_HACKRF_SOURCE=OFF \
    -DOPT_BUILD_LIMESDR_SOURCE=OFF \
    -DOPT_BUILD_PLUTOSDR_SOURCE=OFF \
    -DOPT_BUILD_SDRPLAY_SOURCE=OFF \
    -DOPT_BUILD_SPYSERVER_SOURCE=OFF \
    -DOPT_BUILD_HERMES_SOURCE=OFF \
    -DOPT_BUILD_RFSPACE_SOURCE=OFF \
    -DOPT_BUILD_AUDIO_SINK=OFF \
    -DOPT_BUILD_M17_DECODER=OFF \
    -DUSE_BUNDLE_DEFAULTS=ON \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_POLICY_VERSION_MINIMUM=3.5 \
    -DCMAKE_OSX_DEPLOYMENT_TARGET=10.15

# Compile
make -j4
```

### Option 2: All Hardware Enabled

If you want to enable all hardware sources, use the following command.
**Note:** Ensure you have installed all necessary dependencies (see Step 1) or specific libraries before running this. Also, lines 166 to 211 in `core.cpp` need to be modified to enable all hardware sources. 

```sh
# Create build directory
mkdir build
cd build

# Run CMake enabling all hardware
cmake .. \
    -DOPT_BUILD_RTL_SDR_SOURCE=ON \
    -DOPT_BUILD_RTL_TCP_SOURCE=ON \
    -DOPT_BUILD_AIRSPY_SOURCE=ON \
    -DOPT_BUILD_AIRSPYHF_SOURCE=ON \
    -DOPT_BUILD_BLADERF_SOURCE=ON \
    -DOPT_BUILD_HACKRF_SOURCE=ON \
    -DOPT_BUILD_LIMESDR_SOURCE=ON \
    -DOPT_BUILD_PLUTOSDR_SOURCE=ON \
    -DOPT_BUILD_SDRPLAY_SOURCE=ON \
    -DOPT_BUILD_SOAPY_SOURCE=ON \
    -DOPT_BUILD_SPYSERVER_SOURCE=ON \
    -DOPT_BUILD_HERMES_SOURCE=ON \
    -DOPT_BUILD_RFSPACE_SOURCE=ON \
    -DOPT_BUILD_USRP_SOURCE=ON \
    -DOPT_BUILD_AUDIO_SOURCE=ON \
    -DOPT_BUILD_PORTAUDIO_SINK=ON \
    -DOPT_BUILD_NEW_PORTAUDIO_SINK=ON \
    -DOPT_BUILD_M17_DECODER=ON \
    -DUSE_BUNDLE_DEFAULTS=ON \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_POLICY_VERSION_MINIMUM=3.5 \
    -DCMAKE_OSX_DEPLOYMENT_TARGET=10.15

# Compile
make -j4
```

## 4. Create App Bundle

Package the application. This is where you can define the name of the resulting app bundle. Here I have named it `SDR++Local.app` so as to not conflict with the official SDR++ app.

```sh
cd ..
sh make_macos_bundle.sh ./build ./SDR++Local.app
```

## 5. Enable Microphone Access

This step is required for the Audio Source to work.

```sh
plutil -insert NSMicrophoneUsageDescription -string "SDR++ needs microphone access for the Audio Source module." SDR++Local.app/Contents/Info.plist
```

## 6. Done

Drag `SDR++Local.app` to your Applications folder. You can also run it from the terminal with `./SDR++Local.app/Contents/MacOS/sdrpp`.
