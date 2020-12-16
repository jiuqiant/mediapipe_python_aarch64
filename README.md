# MediaPipe Python on aarch64 Ubuntu

## Step to build MediaPipe Python on aarch64 Ubuntu (raspberry pi 4).

Step 1. Clone [the mediapipe repo](https://github.com/google/mediapipe) and follow the instruction to setup bazel.

Step 2. Install the dependencies.

     ```bash
     $ sudo apt install -y python3-dev
     $ sudo apt install -y protobuf-compiler
     $ sudo apt install -y cmake
     ```

Step 3. Go to the meidapipe dir.

Step 4. Remove unnecessary opencv modules and linker flags.

     ```bash
     sed -i -e "/\"imgcodecs\"/d;/\"calib3d\"/d;/\"features2d\"/d;/\"highgui\"/d;/\"video\"/d;/\"videoio\"/d" third_party/BUILD

     sed -i -e "/-ljpeg/d;/-lpng/d;/-ltiff/d;/-lImath/d;/-lIlmImf/d;/-lHalf/d;/-lIex/d;/-lIlmThread/d;/-lrt/d;/-ldc1394/d;/-lavcodec/d;/-lavformat/d;/-lavutil/d;/-lswscale/d;/-lavresample/d" third_party/BUILD
     ```

Step 5. Remove opencv-python from requirements.txt

    ```
    diff --git a/requirements.txt b/requirements.txt
    index cee4e45..84617cb 100644
    --- a/requirements.txt
    +++ b/requirements.txt
    @@ -1,7 +1,6 @@
     absl-py
     dataclasses
     numpy == 1.19.3
    -opencv-python
     protobuf>=3.11.4
     six
     wheel
    ```

Step 6. Remove drawing_utils from `__init__.py` since opencv-python is not available.

     ```
     --- a/mediapipe/python/solutions/__init__.py
     +++ b/mediapipe/python/solutions/__init__.py
     @@ -14,7 +14,8 @@

      """MediaPipe Solutions Python API."""

      -import mediapipe.python.solutions.drawing_utils
      +# Disable when opencv-python is not available.
      +# import mediapipe.python.solutions.drawing_utils
      import mediapipe.python.solutions.face_mesh
      import mediapipe.python.solutions.hands
      import mediapipe.python.solutions.holistic
      ```

Step 7. Disable  carotene_o4t in `third_party/BUILD`.

     ```
     diff --git a/third_party/BUILD b/third_party/BUILD
     index ef408e4..51e1104 100644
     --- a/third_party/BUILD
     +++ b/third_party/BUILD
     @@ -110,6 +104,8 @@ cmake_external(
         "WITH_ITT": "OFF",
         "WITH_JASPER": "OFF",
         "WITH_WEBP": "OFF",
+        "ENABLE_NEON": "OFF",
+        "WITH_TENGINE": "OFF",
     ```

Step8. Build the package.

     ```bash
     ~/mediapipe$ python3 setup.py gen_protos && python3 setup.py bdist_wheel
     ```

## Steps to run MediaPipe Python

Step 1. Install mediapipe mediapipe/dist/mediapipe-0.8.1-cp38-cp38-linux_aarch64.whl

    ```bash
    ~$ python3 -m pip install mediapipe/dist/mediapipe-0.8.1-cp38-cp38-linux_aarch64.whl
    ```

Step 2. Run the following example code:

    ```python
    import mediapipe as mp
    import numpy as np
    import PIL.Image as Image
    mp_holistic = mp.solutions.holistic

    holistic = mp_holistic.Holistic(static_image_mode=True)
    for idx, file in enumerate(['/path/to/pic.jpg', ''/path/to/pic2.jpg'']):
      pic = Image.open(file)
      image_data = np.frombuffer(pic.tobytes(), dtype=np.uint8)
      image = np.copy(image_data.reshape((image_width, image_hight, 3))[:,:,::-1])
      image_hight, image_width, _ = image.shape
      results = holistic.process(image)
      if results.pose_landmarks:
        print(
            f'Nose coordinates: ('
            f'{results.pose_landmarks.landmark[mp_holistic.PoseLandmark.NOSE]\
    .x * image_width}, '
            f'{results.pose_landmarks.landmark[mp_holistic.PoseLandmark.NOSE]\
    .y * image_hight})'
        )
    holistic.close()
    ```
