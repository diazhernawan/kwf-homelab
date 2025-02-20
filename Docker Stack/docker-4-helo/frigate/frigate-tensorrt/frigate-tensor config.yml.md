
FRIGATE-CONFIG-08-JAN-2025

```yml
######FRIGATE-CONFIG-08JAN2025######
tls:
  enabled: false
mqtt:
  enabled: true
  host: 10.0.3.9
  port: 1883
  topic_prefix: frigate
  client_id: frigate
  user: mqtt-user
  password: mqtt

######DETECTORS######
detectors:
  tensorrt:
    type: tensorrt
    device: 0
model:
  path: /config/model_cache/tensorrt/yolov7-tiny-416.trt
  input_tensor: nchw
  input_pixel_format: rgb
  width: 416
  height: 416

######NVR######
ffmpeg:
  hwaccel_args: preset-nvidia-h264-decode
  output_args:
    record: preset-record-generic-audio-aac
record:
  enabled: true
  retain:
    days: 7
    mode: motion
detect:
  fps: 5
  width: 704
  height: 576
  enabled: true
  stationary:
    interval: 50
    threshold: 50
    max_frames:
      default: 3000
      objects:
        person: 1000
  annotation_offset: 0
objects:
  track:
    - person
    - cat
    - dog
    - car
    - motorcycle
snapshots:
  enabled: true
  timestamp: true
  bounding_box: true
  retain:
    default: 10

######TELEMETRY######
telemetry:
  network_interfaces:
    - eth
    - enp
    - eno
    - ens
    - wl
    - lo

  stats:
    intel_gpu_stats: true
    network_bandwidth: true
  version_check: true

######GO2RTC######
go2rtc:
  streams:
    Carport_Depan:
      - rtsp://admin:brabus18@10.0.3.200:554/cam/realmonitor?channel=1subtype=0
    Carport_Depan_sub:
      - rtsp://admin:brabus18@10.0.3.200:554/cam/realmonitor?channel=1subtype=1
    Carport_Samping:
      - rtsp://admin:brabus18@10.0.3.205:554/cam/realmonitor?channel=1subtype=0
    Carport_Samping_sub:
      - rtsp://admin:brabus18@10.0.3.205:554/cam/realmonitor?channel=1subtype=1
    Taman_Belakang:
      - rtsp://admin:brabus18@10.0.3.210:554/cam/realmonitor?channel=1subtype=0
    Taman_Belakang_sub:
      - rtsp://admin:brabus18@10.0.3.210:554/cam/realmonitor?channel=1subtype=1
#    Office:
#      - ffmpeg:rtsp://10.0.3.215:554/user=admin_password=MH3DeRW6_channel=1_stream=0&amp;onvif=0.sdp?real_st#video=copy#audio=copy#audio=aac
#    Office_sub:
#      - ffmpeg:rtsp://10.0.3.215:554/user=admin_password=MH3DeRW6_channel=1_stream=1&amp;onvif=0.sdp?real_st#video=copy#audio=copy#audio=aac
  webrtc:
    candidates:
      - 10.0.1.54:8555
      - stun:8555


######CAMERAS######
cameras:
  Carport_Depan:
    ffmpeg:
      hwaccel_args: preset-nvidia-h264
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://10.0.1.54:8554/Carport_Depan
          input_args: preset-rtsp-restream
          roles:
            - record
        - path: rtsp://10.0.1.54:8554/Carport_Depan_sub
          input_args: preset-rtsp-restream
          roles:
            - detect
    webui_url: 10.0.3.200
  Carport_Samping:
    ffmpeg:
      hwaccel_args: preset-nvidia-h264
      output_args:
        record: preset-record-generic-audio-aac
      inputs:
        - path: rtsp://10.0.1.54:8554/Carport_Samping
          input_args: preset-rtsp-restream
          roles:
            - record
        - path: rtsp://10.0.1.54:8554/Carport_Samping_sub
          input_args: preset-rtsp-restream
          roles:
            - detect
    webui_url: 10.0.3.205
    motion:
      mask:
        - 0.362,0.411,0.425,0.187,0.513,0.191,0.51,0.425
        - 0.23,0.546,0.475,0.581,0.454,1,0,1,0,0.902,0.103,0.646
  Taman_Belakang:
    ffmpeg:
      hwaccel_args: preset-nvidia-h264
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://10.0.1.54:8554/Taman_Belakang
          input_args: preset-rtsp-restream
          roles:
            - record
        - path: rtsp://10.0.1.54:8554/Taman_Belakang_sub
          input_args: preset-rtsp-restream
          roles:
            - detect
    webui_url: 10.0.3.210
#  Office:
#    ffmpeg:
#      hwaccel_args: preset-nvidia-h264
#      output_args:
#        record: preset-record-generic-audio-copy
#      inputs:
#        - path: rtsp://10.0.1.54:8554/Office
#          input_args: preset-rtsp-restream
#          roles:
#            - record
#        - path: rtsp://10.10.50.84:8554/Office_sub
#          input_args: preset-rtsp-restream
#          roles:
#            - detect
#    webui_url: 10.0.3.215
version: 0.14
camera_groups:
  Public:
    order: 1
    icon: LuActivity
    cameras:
      - Carport_Depan
      - Carport_Samping
      - Taman_Belakang
```