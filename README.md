# Duckietown robotics coursework and experiments

Python experiments collected during robotics coursework, using a Duckietown ROS template. The repository contains a camera-based line follower, object-detection integration code, landmark simulation helpers and a CNN training script.

**Status:** historical coursework under documentation and reproducibility review. This is not a complete, validated autonomous-driving stack.

## Repository contents

| File | What is present | Current limitation |
| --- | --- | --- |
| `packages/LineFollower.py` | ROS 1 image subscriber, OpenCV HSV masking, image-moment centroid and proportional steering published as `Twist` | Adapted/included upstream example; hardware and simulator execution have not been revalidated |
| `packages/featur_det.py` | ROS wrapper using `dodo_detector`, with TensorFlow/keypoint detector options and optional point-cloud input | External detector packages, models and configuration are required; integration is not verified |
| `packages/SLAM.py` | Landmark-world visualization and simulated measurement/motion generation | Imports `robot_class`, which is absent; this file is not a complete SLAM estimator |
| `packages/model.py` | Keras binary CNN training script with image augmentation | Referenced dataset is absent; no verified training result or robotics integration is supplied |
| `Dockerfile`, `launchers/` | Original Duckietown template infrastructure | Placeholder metadata, empty dependency declarations and placeholder launcher remain |

## Line-following pipeline

A ROS image is converted to BGR with `cv_bridge`, converted to HSV and thresholded. A horizontal strip near the lower portion of the image is retained. Its mask centroid determines horizontal error relative to the image center, and a proportional controller publishes forward and angular velocity.

Current interfaces:
- Camera input: `/mybot/camera1/image_raw` (`sensor_msgs/Image`).
- Velocity output: `/cmd_vel` (`geometry_msgs/Twist`).
- ROS API: `rospy` (ROS 1, not ROS 2).
- Debug display: OpenCV GUI windows.

These topic names follow the upstream simulated robot example; they are not a verified Duckiebot hardware interface.

## Dependencies and execution status

The line follower requires a compatible ROS 1 environment with `rospy`, `sensor_msgs`, `geometry_msgs`, `cv_bridge`, NumPy and OpenCV. Python, ROS and cv_bridge versions must be compatible. The original working environment has not been recovered.

Once ROS is configured and a compatible simulated robot supplies the camera and velocity interfaces, the script entry point is:

```bash
python3 packages/LineFollower.py
```

This is an entry point, not a verified end-to-end installation recipe. The supplied Docker template does not currently launch it.

For local image-processing checks without ROS:

```bash
python3 -m pip install numpy opencv-python-headless
python3 -m unittest discover -s tests -v
```

The tests use real NumPy/OpenCV on synthetic images and stub ROS transport and GUI calls. They check centroid-based steering and cropping; they do not validate ROS communication, simulation, hardware or real-world perception.

## Limitations

- The inherited HSV thresholds are broad and need calibration; the code does not establish robust yellow-lane detection.
- If no mask is found, the current controller publishes no new command. It has no explicit stop-on-loss or stale-image watchdog; do not deploy it on a moving robot without addressing and testing those behaviors.
- The repository does not establish obstacle avoidance, integrated mapping, a completed SLAM pipeline or measured navigation performance.
- Dataset identities, trained model results and historical personal modifications require additional evidence.

## Attribution and project context

Repository maintained by Badr Essefiany as a record of robotics coursework.

The line follower is derived from [Arjun S Kumar's Line-Follower--ROS](https://github.com/arjunskumar/Line-Follower--ROS). Its image-processing and steering logic closely match that example. Upstream authorship is retained; inclusion here does not imply original authorship of that algorithm.

The repository infrastructure comes from [Duckietown's ROS template](https://github.com/duckietown/template-ros). The detector wrapper imports `dodo_detector` and `dodo_detector_ros`; the provenance and extent of modifications to the remaining experiments still need review. Existing license material is retained.

The current maintenance pass fixes Python syntax and slice indexing and adds documentation and focused checks. It does not establish which historical components were independently implemented by the repository owner.

