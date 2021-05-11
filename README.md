# MediaPipe Python on aarch64 Ubuntu 20.04

*[Experimental] Only tested on Raspberry Pi 4 and Jeton Xavier NX.*

## Step to build MediaPipe Python on aarch64 Ubuntu 20.04

1.  Clone [the MediaPipe repo](https://github.com/google/mediapipe) and follow
    the instruction to setup Bazel.

2.  Install build dependencies.

    ```bash
    $ sudo apt install -y python3-dev
    $ sudo apt install -y cmake
    ```

3.  Install proto compiler.

    ```
    $ sudo apt install -y protobuf-compiler
    ```

    If you see a missing any.proto error later, which means the protoc might be
    too old, you can download the latest protoc-3.x.x-linux-aarch_64.zip from
    [GitHub](https://github.com/protocolbuffers/protobuf/releases) and copy the
    "bin" and "include/google" directories to the system libraries. Then, modify
    `mediapipe/setup.py` like the following:

    ```python
    diff --git a/setup.py b/setup.py
    index 61848de..462d91d 100644
    --- a/setup.py
    +++ b/setup.py
    @@ -208,7 +208,7 @@ class GeneratePyProtos(setuptools.Command):
             sys.stderr.write('cannot find required file: %s\n' % source)
             sys.exit(-1)

    -      protoc_command = [self._protoc, '-I.', '--python_out=.', source]
    +      protoc_command = [self._protoc, '-I.', '-I/usr/local/include', '--python_out=.', source]
           if subprocess.call(protoc_command) != 0:
             sys.exit(-1)

    ```

4.  Go to the MediaPipe directory.

    ```bash
    ~$ cd mediapipe
    ```

5.  Remove unnecessary OpenCV modules and linker flags.

    ```bash
    sed -i -e "/\"imgcodecs\"/d;/\"calib3d\"/d;/\"features2d\"/d;/\"highgui\"/d;/\"video\"/d;/\"videoio\"/d" third_party/BUILD
    sed -i -e "/-ljpeg/d;/-lpng/d;/-ltiff/d;/-lImath/d;/-lIlmImf/d;/-lHalf/d;/-lIex/d;/-lIlmThread/d;/-lrt/d;/-ldc1394/d;/-lavcodec/d;/-lavformat/d;/-lavutil/d;/-lswscale/d;/-lavresample/d" third_party/BUILD
    ```

6.  Disable carotene_o4t in `third_party/BUILD`.

    ```
    diff --git a/third_party/BUILD b/third_party/BUILD
    index ef408e4..51e1104 100644
    --- a/third_party/BUILD
    +++ b/third_party/BUILD
    @@ -110,6 +104,8 @@ cmake_external(
       "WITH_ITT": "OFF",
       "WITH_JASPER": "OFF",
       "WITH_WEBP": "OFF",
    +   "ENABLE_NEON": "OFF",
    +   "WITH_TENGINE": "OFF",
    ```

7.  Build the package.

    ```bash
    ~/mediapipe$ python3 setup.py gen_protos && python3 setup.py bdist_wheel
    ```

## Steps to run MediaPipe Python on aarch64 Ubuntu 20.04

1.  Install MediaPipe package.

    ```bash
    ~$ python3 -m pip install cython
    ~$ python3 -m pip install numpy
    ~$ python3 -m pip install pillow
    ~$ python3 -m pip install mediapipe/dist/mediapipe-0.8-cp38-cp38-linux_aarch64.whl
    or 
    ~$ python3 -m pip install mediapipe-python-aarch64/mediapipe-0.8.4-cp38-cp38-linux_aarch64.whl
    ```

    Append `--no-deps` flag if any dependency Python packages cannot be installed.

2.  Run the example code.

    ```python
    import mediapipe as mp
    import numpy as np
    import PIL.Image as Image
    mp_holistic = mp.solutions.holistic

    holistic = mp_holistic.Holistic(static_image_mode=True)
    for idx, file in enumerate(['/path/to/pic.jpg', '/path/to/pic2.jpg']):
      pic = Image.open(file)
      image_data = np.frombuffer(pic.tobytes(), dtype=np.uint8)
      image = np.copy(image_data.reshape((image_height, image_width, 3))[:,:,::-1])
      image_height, image_width, _ = image.shape
      results = holistic.process(image)
      if results.pose_landmarks:
        print(
            f'Nose coordinates: ('
            f'{results.pose_landmarks.landmark[mp_holistic.PoseLandmark.NOSE]\
     .x * image_width}, '
            f'{results.pose_landmarks.landmark[mp_holistic.PoseLandmark.NOSE]\
     .y * image_height})'
        )
    holistic.close()
    ```
