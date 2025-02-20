FRIGATE-CONFIG-08-JAN-2025

```yml
######FRIGATE-CONFIG-22APRIL2024######
mqtt:
  enabled: true
  host: 192.168.1.9
  port: 1883
  topic_prefix: frigate
  client_id: frigate
  user: mqtt-user
  password: mqtt


######DETECTORS&NVR######
detectors:
  ov:
    type: openvino
    device: AUTO
    model:
      path: /openvino-model/ssdlite_mobilenet_v2.xml

model:
  width: 300
  height: 300
  input_tensor: nhwc
  input_pixel_format: bgr
  labelmap_path: /openvino-model/coco_91cl_bkgr.txt

ffmpeg:
  hwaccel_args: preset-vaapi
  output_args:
    record: preset-record-generic-audio-aac

motion:
  threshold: 30
  lightning_threshold: 0.8
  contour_area: 10
  frame_alpha: 0.01
  frame_height: 100
    #  Optional: motion mask
    #  NOTE: see docs for more detailed info on creating masks
  mask: 207,327,210,270,193,222,157,219,127,262,121,320
  improve_contrast: true
    #  Optional: Delay when updating camera motion through MQTT from ON -> OFF (default: shown below).
  mqtt_off_delay: 30


record:
  enabled: true
  retain:
    days: 7
    mode: motion
  events:
    pre_capture: 2
    post_capture: 3
    retain:
      default: 7
      mode: active_objects
    objects:
    - person
    - car

detect:
  fps: 25
  width: 704
  height: 576
  enabled: true
  stationary:
    interval: 50
    threshold: 50
    max_frames:
      # Optional: Default for all object types (default: not set, track forever)
      default: 3000
      # Optional: Object specific values
      objects:
        person: 1000
  annotation_offset: 0

objects:
  track:
  - person
  - car

snapshots:
  enabled: true
  timestamp: true
  bounding_box: true
  retain:
    default: 10



######TELEMETRY######
telemetry:
  # Optional: Enabled network interfaces for bandwidth stats monitoring (default: empty list, let nethogs search all)
  network_interfaces:
  - eth
  - enp
  - eno
  - ens
  - wl
  - lo

  stats:
    intel_gpu_stats: true
    # Enable network bandwidth stats monitoring for camera ffmpeg processes, go2rtc, and object detectors. (default: shown below)
    # NOTE: The container must either be privileged or have cap_net_admin, cap_net_raw capabilities enabled.
    network_bandwidth: true

  version_check: true


#birdseye:
  # Optional: Enable birdseye view (default: shown below)
#  enabled: True
  # Optional: Restream birdseye via RTSP (default: shown below)
  # NOTE: Enabling this will set birdseye to run 24/7 which may increase CPU usage somewhat.
#  restream: False
  # Optional: Width of the output resolution (default: shown below)
#  width: 1280
  # Optional: Height of the output resolution (default: shown below)
#  height: 720
  # Optional: Encoding quality of the mpeg1 feed (default: shown below)
  # 1 is the highest quality, and 31 is the lowest. Lower quality feeds utilize less CPU resources.
#  quality: 8
  # Optional: Mode of the view. Available options are: objects, motion, and continuous
  #   objects - cameras are included if they have had a tracked object within the last 30 seconds
  #   motion - cameras are included if motion was detected in the last 30 seconds
  #   continuous - all cameras are included always
#  mode: objects
  # Optional: Threshold for camera activity to stop showing camera (default: shown below)
#  inactivity_threshold: 30
  # Optional: Configure the birdseye layout
#  layout:
    # Optional: Scaling factor for the layout calculator, range 1.0-5.0 (default: shown below)
#    scaling_factor: 2.0
    # Optional: Maximum number of cameras to show at one time, showing the most recent (default: show all cameras)
#    max_cameras: 1



######go2rtc######
go2rtc:
  streams:
    Carport_Depan:
    - rtsp://admin:brabus18@192.168.1.200:554/cam/realmonitor?channel=1subtype=0   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg
    Carport_Depan_sub:
    - rtsp://admin:brabus18@192.168.1.200:554/cam/realmonitor?channel=1subtype=1   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg
    Carport_Samping:
    - rtsp://admin:brabus18@192.168.1.205:554/cam/realmonitor?channel=1subtype=0   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg
    Carport_Samping_sub:
    - rtsp://admin:brabus18@192.168.1.205:554/cam/realmonitor?channel=1subtype=1   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg 
    Taman_Belakang:
    - rtsp://admin:brabus18@192.168.1.210:554/cam/realmonitor?channel=1subtype=0   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg
    Taman_Belakang_sub:
    - rtsp://admin:brabus18@192.168.1.210:554/cam/realmonitor?channel=1subtype=1   # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg 
    Office:
    - ffmpeg:rtsp://192.168.1.215:554/user=admin_password=MH3DeRW6_channel=1_stream=0&amp;onvif=0.sdp?real_st#video=copy#audio=copy#audio=aac    # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg
    Office_sub:
    - ffmpeg:rtsp://192.168.1.215:554/user=admin_password=MH3DeRW6_channel=1_stream=1&amp;onvif=0.sdp?real_st#video=copy#audio=copy#audio=aac    # <- stream which supports video & aac audio. This is only supported for rtsp streams, http must use ffmpeg

  webrtc:
    candidates:
    - 192.168.1.5:8555
    - stun:8555


######CAMERAS######
cameras:
  Carport_Depan:
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
      - path: rtsp://192.168.1.5:8554/Carport_Depan   # <--- the name here must match the name of the camera in restream
        input_args: preset-rtsp-restream
        roles:
        - record
      - path: rtsp://192.168.1.5:8554/Carport_Depan_sub   # <--- the name here must match the name of the camera_sub in restream
        input_args: preset-rtsp-restream
        roles:
        - detect
    webui_url: 192.168.1.200

  Carport_Samping:
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
      - path: rtsp://192.168.1.5:8554/Carport_Samping   # <--- the name here must match the name of the camera in restream
        input_args: preset-rtsp-restream
        roles:
        - record
      - path: rtsp://192.168.1.5:8554/Carport_Samping_sub   # <--- the name here must match the name of the camera_sub in restream
        input_args: preset-rtsp-restream
        roles:
        - detect
    webui_url: 192.168.1.205
    motion:
      mask:
      - 242,328,357,325,384,188,388,67,322,83
      - 300,576,320,462,319,355,186,318,93,362,29,461,0,576

  Taman_Belakang:
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
      - path: rtsp://192.168.1.5:8554/Taman_Belakang   # <--- the name here must match the name of the camera in restream
        input_args: preset-rtsp-restream
        roles:
        - record
      - path: rtsp://192.168.1.5:8554/Taman_Belakang_sub   # <--- the name here must match the name of the camera_sub in restream
        input_args: preset-rtsp-restream
        roles:
        - detect
    webui_url: 192.168.1.210

  Office:
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
      - path: rtsp://192.168.1.5:8554/Office   # <--- the name here must match the name of the camera in restream
        input_args: preset-rtsp-restream
        roles:
        - record
      - path: rtsp://192.168.1.5:8554/Office_sub   # <--- the name here must match the name of the camera_sub in restream
        input_args: preset-rtsp-restream
        roles:
        - detect
    webui_url: 192.168.1.215



```